## Creating a Data Node
Install and configure Elasticsearch on a new machine. It is recommended to install via the packed archive (or use Docker) as installing from a pre-packaged Debian makes this process more complicated than needed.

I would recommend creating a new user and storing the Elasticsearch archive inside of the user's home directory. A systemd service will be created to run Elasticsearch on startup.  
## Creating a Master Node
Configure Elasticsearch to run as a master node by including the following inside of `/etc/elasticsearch/elasticsearch.yml`:
```yml
...
#
# Configure node roles:
#
node.roles: master,ingest,remote_cluster_client
...
```
> This works when there is NO EXISTING SHARD DATA on the node.
> The ingest and remote_cluster_client roles allow monitoring of stack management.

