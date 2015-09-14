_ed note: this is an overview of basic memcached use case, and how memcached clients work_

Two plucky adventurers, Programmer and Sysadmin, set out on a journey. Together they make websites. Websites with webservers and databases. Users from all over the Internet talk to the webservers and ask them to make pages for them. The webservers ask the databases for junk they need to make the pages. Programmer codes, Sysadmin adds webservers and database servers.

One day the Sysadmin realizes that their database is sick! It's spewing bile and red stuff all over! Sysadmin declares it has a fever, a load average of 20! Programmer asks Sysadmin, "well, what can we do?" Sysadmin says, "I heard about this great thing called memcached. It really helped livejournal!" "Okay, let's try it!" says the Programmer.

Our plucky Sysadmin eyes his webservers, of which he has six. He decides to use three of them to run the 'memcached' server. Sysadmin adds a gigabyte of ram to each webserver, and starts up memcached with a limit of 1 gigabyte each. So he has three memcached instances, each can hold up to 1 gigabyte of data. So the Programmer and the Sysadmin step back and behold their glorious memcached!

"So now what?" they say, "it's not DOING anything!" The memcacheds aren't talking to anything and they certainly don't have any data. And NOW their database has a load of 25!

Our adventurous Programmer grabs the pecl/memcache client library manual, which the plucky Sysadmin has helpfully installed on all SIX webservers. "Never fear!" he says. "I've got an idea!" He takes the IP addresses and port numbers of the THREE memcacheds and adds them to an array in php.

```
$MEMCACHE_SERVERS = array(
    "10.1.1.1", //web1
    "10.1.1.2", //web2
    "10.1.1.3", //web3
);
```

Then he makes an object, which he cleverly calls '$memcache'.

```
$memcache = new Memcache();
foreach($MEMCACHE_SERVERS as $server){
    $memcache->addServer ( $server );
}
```

Now Programmer thinks. He thinks and thinks and thinks. "I know!" he says. "There's this thing on the front page that runs `SELECT * FROM hugetable WHERE timestamp > lastweek ORDER BY timestamp ASC LIMIT 50000;` and it takes five seconds!" "Let's put it in memcached," he says. So he wraps his code for the SELECT and uses his $memcache object. His code asks:

Are the results of this select in memcache?
If not, run the query, take the results, and PUT it in memcache!
Like so:

```
$huge_data_for_front_page = $memcache->get("huge_data_for_front_page");
if($huge_data_for_front_page === false){
    $huge_data_for_front_page = array();
    $sql = "SELECT * FROM hugetable WHERE timestamp > lastweek ORDER BY timestamp ASC LIMIT 50000";
    $res = mysql_query($sql, $mysql_connection);
    while($rec = mysql_fetch_assoc($res)){
        $huge_data_for_front_page[] = $rec;
    }
    // cache for 10 minutes
    $memcache->set("huge_data_for_front_page", $huge_data_for_front_page, 0, 600);
}

// use $huge_data_for_front_page how you please
```

Programmer pushes code. Sysadmin sweats. BAM! DB load is down to 10! The website is pretty fast now. So now, the Sysadmin puzzles, "What the HELL just happened!?" "I put graphs on my memcacheds! I used cacti, and this is what I see! I see traffic to one memcached, but I made three :(." So, the Sysadmin quickly learns the ascii protocol and telnets to port 11211 on each memcached and asks it:

Hey, 'get huge\_data\_for\_front\_page' are you there?

The first memcached does not answer...

The second memcached does not answer...

The third memcached, however, spits back a huge glob of crap into his telnet session! There's the data! Only once memcached has the key that the Programmer cached!

Puzzled, he asks on the mailing list. They all respond in unison, "It's a distributed cache! That's what it does!" But what does that mean? Still confused, and a little scared for his life, the Sysadmin asks the Programmer to cache a few more things. "Let's see what happens. We're curious folk. We can figure this one out," says the Sysadmin.

"Well, there is another query that is not slow, but is run 100 times per second. Maybe that would help," says the Programmer. So he wraps that up like he did before. Sure enough, the server loads drops to 8!

So the Programmer codes more and more things get cached. He uses new techniques. "I found them on the list and the faq! What nice blokes," he says. The DB load drops; 7, 5, 3, 2, 1!

"Okay," says the Sysadmin, "let's try again." Now he looks at the graphs. ALL of the memcacheds are running! All of them are getting requests! This is great! They're all used!

So again, he takes keys that the Programmer uses and looks for them on his memcached servers. 'get this\_key' 'get that\_key' But each time he does this, he only finds each key on one memcached! Now WHY would you do this, he thinks? And he puzzles all night. That's silly! Don't you want the keys to be on all memcacheds?

"But wait", he thinks "I gave each memcached 1 gigabyte of memory, and that means, in total, I can cache three gigabytes of my database, instead of just ONE! Oh man, this is great," he thinks. "This'll save me a ton of cash. Brad Fitzpatrick, I love your ass!"

"But hmm, the next problem, and this one's a puzzler, this webserver right here, this one runing memcached it's old, it's sick and needs to be upgraded. But in order to do that I have to take it offline! What will happen to my poor memcache cluster? Eh, let's find out," he says, and he shuts down the box. Now he looks at his graphs. "Oh noes, the DB load, it's gone up in stride! The load isn't one, it's now two. Hmm, but still tolerable. All of the other memcacheds are still getting traffic. This ain't so bad. Just a few cache misses, and I'm almost done with my work. So he turns the machine back on, and puts memcached back to work. After a few minutes, the DB load drops again back down to 1, where it should always be.

"The cache restored itself! I get it now. If it's not available it just means a few of my requests get missed. But it's not enough to kill me. That's pretty sweet."

So, the Programmer and Sysadmin continue to build websites. They continue to cache. When they have questions, they ask the mailing list or read the faq again. They watch their graphs. And all live happily ever after.

Author: Dormando via IRC. Edited by Brian Moon for fun. Further fun editing by Emufarmers.

This story has been illustrated by the online comic [TOBlender.com](http://toblender.com/tag/memcached/).

Chinese [translation](http://code.google.com/p/memcached/wiki/TutorialCachingStory) by Wei Liu.