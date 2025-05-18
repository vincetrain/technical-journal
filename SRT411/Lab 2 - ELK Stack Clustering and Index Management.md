[Add and Remove Elasticsearch Nodes](https://www.elastic.co/docs/deploy-manage/maintenance/add-and-remove-elasticsearch-nodes)

[Node roles](https://www.elastic.co/docs/deploy-manage/distributed-architecture/clusters-nodes-shards/node-roles)

[Cluster-level shard allocation filtering](https://www.elastic.co/docs/reference/elasticsearch/configuration-reference/cluster-level-shard-allocation-routing-settings#cluster-shard-allocation-filtering)

[systemd/User How it works](https://wiki.archlinux.org/title/Systemd/User#How_it_works)

[systemd.unit](https://man.archlinux.org/man/systemd.unit.5.en)

[systemd.service](https://man.archlinux.org/man/systemd.service.5)
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

If you already have shard data on the node, have at least 2 data nodes connected (by default, Elasticsearch will have 2 replicas of shards) and create the following PUT request to Elasticsearch.
```http
PUT _cluster/settings
{
	"persistent": {
		"cluster.routing.allocation.exclude._ip": "ip-address-of-master-with-shards"
	}
}
```
> This tells Elasticsearch to exclude shard allocation from the node residing at the provided IP address.
> Keep an eye on the Stack Monitoring page as it provides crucial information about how many shards are stored on each node. Once the master node has 0 shards and has Green health, it is okay to change `node.roles` and restart.
## Creating a Data Node
Install and configure Elasticsearch on a new machine. It is recommended to **install via the archive** as installing from a pre-packaged Debian makes this process more complicated than needed.
> ENSURE VERSIONS ARE COMPATIBLE WITH OR MATCH THE MASTER NODE!

In this installation version of Elasticsearch, configuration files (like `elasticsearch.yml`) will be in `/config` of where Elasticsearch was installed to. 

I would recommend creating a new Linux user and storing the Elasticsearch archive inside of the user's home directory. A systemd service will be created to run Elasticsearch on startup.
> The rest of this section follows installing Elasticsearch as Linux user `elasticsearch`.

Before attempting to join this node to the cluster, consider curling or pinging the Elasticsearch master node to ensure connectivity.

### Configuring the node and joining the cluster
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

On an Elasticsearch **master node**, run the `bin/elasticsearch-create-enrollment-token` binary to generate an Elasticsearch enrollment token to be used for joining this node.
```sh
bin/elasticsearch-create-enrollment-token -s node
```

Now copy this enrollment token onto the data node and use the `bin/elasticsearch` binary to join the cluster.
```sh
bin/elasticsearch --enrollment-token [paste-enrollment-token-here]
```

### Configuring Elasticsearch as a service
This portion will work for those who are using `systemd`.

Start by switching to the user that owns and will run Elasticsearch.
```bash
ssh elasticsearch@localhost ## do NOT use su to switch users. SSH locally instead.
```

We will store our Elasticsearch service inside of `/home/elasticsearch/.local/share/systemd/user/`.
```bash
mkdir -p ~/.local/share/systemd/user/
touch ~/.local/share/systemd/user/elasticsearch.service
```

Define the service to run Elasticsearch
```service
[Unit]
Description=Elasticsearch
[Service]
ExecStart=/path/to/elasticsearch/bin/elasticsearch
[Install]
WantedBy=default.target
```

To perform operations on this service, we can use:
```bash
systemctl [verb] --user
```
> These only work when ran as the user owning the service, or in this case the `elasticsearch` user.

To enable this service to run on startup, we must first enable the service and allow the user to linger.
```bash
systemctl enable --user elasticsearch
loginctl enable-linger elasticsearch
```