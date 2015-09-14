﻿#summary This is What We Do

## Release Cycle ##

  * 3 weeks after each new stable release, we release -rc1 for the next release.
  * New -rc's will be kicked out daily or bidaily if there are fixes.
  * After 3 days in RC, unless there are still bug reports coming in, stable is released.

This should lead to a stable release roughly once per month. Exceptions
can be made, as usual. Major bug finds warrant earlier releases.
Cycles with large code changes all at once might warrant
an earlier cut to -rc1 and a 2-3 week -rc cycle.

The release may be done by any of the core committers to memcached, but the process requires separation of contribution and review (i.e. the author does not commit/review their own changes).

## Road Map ##

Our primary goal is to finalize and release the [Storage Engine Interface](EngineInterface.md). This may end up including other changes.

As this comes together, we'll further dicuss more formal roadmaps, pieced together from the various notes on the old wiki along with new discussions.