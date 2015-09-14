﻿#summary Committee

TODO: Fold this into ServerMaint to reduce frontpage links?

# Capacity Planning #

Setting up graphs (See [Tools](Tools.md) and similar) for longterm monitoring of many memcached values is an important part of capacity planning. Watch trends over time and draw lines to decide when it's time to invest in seeing what your aplication is up to or adding more memory to the cluster.

# Upgrades #

We're very careful about the high quality of memcached releases, but you should exercise proper caution when upgrading. Run a new release in any QA or dev environment you may have for a while, then upgrade a single server in production. If things look okay, slowly roll it out to the rest.

Running clusters with mixed versions can be a headache for administrators when using monitoring.

# Finding Outliers #

If you're carefully graphing all servers, or using tools to monitor all servers, watch for outliers. Bugs in clients, or small numbers of hot keys can cause some servers to get much more traffic than others. Identify these before they become a hazard.
