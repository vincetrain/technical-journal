
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
> The `ingest` and `remote_cluster_client` roles allow monitoring of stack management.

## Creating a Data Node
Install and configure Elasticsearch on a new machine. It is recommended to install via the packed archive (or use Docker) as installing from a pre-packaged Debian makes this process more complicated than needed.
> ENSURE VERSIONS ARE COMPATIBLE WITH OR MATCH THE DATA NODE!

In this installation version of Elasticsearch, configuration files (like `elasticsearch.yml`) will be in `/config` of where Elasticsearch was installed to. 

I would recommend creating a new user and storing the Elasticsearch archive inside of the user's home directory. A systemd service will be created to run Elasticsearch on startup.

For this node to connect to a cluster as a data node, modify the configuration of `elasticsearch.yml` to define the `cluster.name` (must match with the cluster we are joining), `node.name` (must be unique), and the `node.roles` (set this to data). Everything else may be left untouched or configured however.
```yml
...
#
# Use a descriptive name for your cluster:
#
cluster.name: my-cluster
...
#
# Use a descriptive name for the node:
#
node.name: srt411data1
...
#
# Define node roles:
#
node.roles: data
```
> This is the bare minimum Elasticsearch configuration required to join a cluster as a data node. 
> You may also define your own security features, but enrolling a node with an enrollment token will automatically configure security options.

On an Elasticsearch **master node**, run the `bin/elasticsearch/`