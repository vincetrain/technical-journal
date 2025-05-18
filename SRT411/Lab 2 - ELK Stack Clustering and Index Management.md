## Creating a Master Node
Install and configure Elasticsearch normally.

Configure Elasticsearch to run as a master node by including the following inside of `/etc/elasticsearch/elasticsearch.yml`:
```yml
...
#
# Configure node roles:
#
node.roles: master,ingest
...
```
> This works when there is NO EXISTING SHARD DATA on the node.
> The ingest role allows monitoring of stack manage

