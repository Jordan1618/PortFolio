## Galera Cluster, What is it ?

Galera Cluster is a synchronous, multi-master replication solution for MySQL/MariaDB. Every node can accept writes; transactions are certified and replicated to all nodes before commit, so there's no replication lag and all nodes stay consistent. Write conflicts between nodes cause a transaction rollback (deadlock error), which the application must handle by retrying.

Quorum: requires a majority of nodes alive to stay "Primary" and accept writes — hence odd numbers of nodes (3+ minimum), or a lightweight arbitrator (garbd) as a tie-breaker to avoid split-brain.

Node sync: new or rejoining nodes catch up via IST (incremental, just the missing transactions) or SST (full data copy via mariabackup/rsync) if too far behind.

Deployment (3 nodes): install mariadb-server + galera-4, configure wsrep settings (cluster address, node address/name, SST method) identically on each node, bootstrap the first node with `galera_new_cluster`, then start the others normally to auto-sync. Check status via `wsrep_cluster_size` and `wsrep_cluster_status`.

In production, it's paired with HAProxy (load-balancing, health-checked on wsrep sync state) and Keepalived (VIP for HA), similar to setups you've already built. Best practice: often route writes to a single active node via HAProxy to avoid certification conflicts, using multi-master purely for HA rather than write load-balancing.