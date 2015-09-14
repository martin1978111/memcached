# Development #

If you are working on development, please publish your work and cc the mailing list as much as possible.

The "master" tree in the central repo should always contain runnable, high quality code. We go out of our way to ensure nothing goes to the central repo without a barrage of regression tests and code reviews.

That aside, use at your own risk.

## The "central" github repo: ##

Web View: http://github.com/memcached/memcached

```
git clone git://github.com/memcached/memcached.git
```

## Active Development ##

We are actively developing against the 1.4 and 1.6 trees.

New changes are **highly** preferred for 1.6 only. 1.4 should receive fewer
updates.

The 1.4 tree is the "master" branch", while 1.6 is the "engine-pu" branch.

## Developer repos: ##

  * dormando: git://github.com/dormando/memcached.git
  * dustin: git://github.com/dustin/memcached.git
  * trond: git://github.com/trondn/memcached.git

# Archive #

  * The community's moved to git distributed sccs for its development needs, but the original subversion repository is still available.
    * http://code.sixapart.com/svn/memcached/