#summary Tools for monitoring memcached

  * Cacti templates:
    * [Better Cacti Templates Memcached Templates](http://code.google.com/p/mysql-cacti-templates/wiki/MemcachedTemplates)
    * [Dealnews Cacti template](http://dealnews.com/developers/cacti/memcached.html)
    * http://forums.cacti.net/about14605.html
    * http://www.faemalia.net/mysqlUtils/
    * [Multiple port per server Cacti Templates](http://tag1consulting.com/node/58)

  * [PhpMemcacheAdmin](http://code.google.com/p/phpmemcacheadmin/)
  * [Hyperic plugin](http://www.hyperic.com/products/managed/memcached-management.htm)
  * [Nagios plugin](http://search.cpan.org/~zigorou/Nagios-Plugins-Memcached-0.02/lib/Nagios/Plugins/Memcached.pm)
  * Ganglia
    * http://ben.hartshorne.net/ganglia/ (2006)
    * http://www.hitflip.de/opensource.html
  * [PHP-based control panel](http://livebookmark.net/journal/2008/05/21/memcachephp-stats-like-apcphp/)
  * [Munin plugin](http://munin.projects.linpro.no/wiki/plugin-memcache)
  * Ruby gem: memcache-client-stats
  * Python: [django examples](http://effbot.org/zone/django-memcached-view.htm)
  * During development:
    * http://github.com/andrewfromgeni/mcinsight - GUI to examine memcached server
  * http://code.google.com/p/memcached-manager
  * Windows
    * MemCacheD Manager, by Nick Pirocanac
      * http://allegiance.chi-town.com/MemCacheDManager.aspx

Or, just do a simple connect to the port where memcached is listening. You can run a simple command like 'version' or 'stats' to see if memcached is listening.