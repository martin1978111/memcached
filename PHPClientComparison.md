# PHP Client Comparison #

There are primarily two clients used with PHP.  One is the older, more widespread [pecl/memcache](http://pecl.php.net/package/memcache) and the other is the newer, less used, more feature rich [pecl/memcached](http://pecl.php.net/package/memcached).

Both support the basics such as multiple servers, setting vaules, getting values, increment, decrement and getting stats.

Here are some more advanced features and information.

<table cellpadding='5' border='1'>
<blockquote><tr>
<blockquote><th> </th>
<th><a href='http://pecl.php.net/package/memcache'>pecl/memcache</a></th>
<th><a href='http://pecl.php.net/package/memcached'>pecl/memcached</a></th>
</blockquote></tr>
<tr>
<blockquote><th align='left'>First Release Date</th>
<td align='center'>2004-06-08</td>
<td align='center'>2009-01-29 (beta)</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Actively Developed?</th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>External Dependency</th>
<td align='center'>None</td>
<td align='center'><a href='http://tangent.org/552/libmemcached.html'>libmemcached</a></td>
</blockquote></tr>
<tr>
<blockquote><td align='center'><strong>Features</strong></td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Automatic Key Fixup<sup>1</sup></th>
<td align='center'>Yes</td>
<td align='center'>No</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Append/Prepend</th>
<td align='center'>No</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Automatic Serialzation<sup>2</sup></th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Binary Protocol</th>
<td align='center'>No</td>
<td align='center'>Optional</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>CAS</th>
<td align='center'>No</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Compression</th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Communication Timeout</th>
<td align='center'>Connect Only</td>
<td align='center'>Various Options</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Consistent Hashing</th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Delayed Get</th>
<td align='center'>No</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Multi-Get</th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Session Support</th>
<td align='center'>Yes</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Set/Get to a specific server</th>
<td align='center'>No</td>
<td align='center'>Yes</td>
</blockquote></tr>
<tr>
<blockquote><th align='left'>Stores Numerics</th>
<td align='center'>Converted to Strings</td>
<td align='center'>Yes</td>
</blockquote></tr>
</table></blockquote>

  1. pecl/memcache will convert an invalid key into a valid key for you.  pecl/memcached will return false when trying to set/get a key that is not valid.
  1. You do not have to serialize your objects or arrays before sending them to the set commands.  Both clients will do this for you.