[Elasticsearch Referenced Material](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-with-debian-package)
[Logstash Referenced Material](https://www.instaclustr.com/support/documentation/elasticsearch/using-logstash/connecting-logstash-to-elasticsearch/)
## Preparing apt Repository
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```
Install signing key

```sh
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
sudo apt-get update
```
Install apt repository
## Installing & Configuring Elasticsearch
Before installing Elasticsearch, **ensure the machine's hostname is the hostname that is desired**! Elasticsearch autogenerates certificates for HTTPS usage when installed. **Mismatching hostnames from hostname changes after installation will mess this up!**

We can now install the `elasticsearch` package with apt.

**Successful installation will show the `elastic` user's in shell. Consider noting it down or adding it to an environment variable**

The `elasticsearch` binary (and other supporting binaries) are stored at `/usr/share/elasticsearch/bin`.

Elasticsearch configurations can be found inside of `/etc/elasticsearch/elasticsearch.yml`.

```sh
curl --cacert /etc/elasticsearch/certs/http_ca.crt -u [username] https://localhost:9200/_cluster/health
```
Check Elasticsearch health

```yml
node.name: example-node-1
...
cluster.initial_master_nodes: ["example-node-1"]
```
For elasticsearch to work, ensure at least 1 correct initial master node is set inside `/etc/elasticsearch/elasticsearch.yml`.
>This solves error status 503, `"master_not_discovered_exception"`
## Installing & Configuring Kibana
We can install the `kibana` package with apt.

Kibana binaries can be found at `/usr/share/kibana/bin`.

The Kibana webserver will be running at [localhost:5601](localhost:5601)

```sh
elasticsearch/bin/elasticsearch-users useradd [user]
```
Create a user. This is the user we will use with Kibana. You will be prompted to enter a password for the user after running this command.

```sh
elasticsearch/bin/elasticsearch-users roles [user] -a kibana_admin,superuser
```
Add required user roles for unlimited Kibana and Elasticsearch usage
>The `kibana_admin` role allows unlimited to usage of Kibana, including administrative use. 
>The `superuser` role allows unlimited read and write access (and usage) of Elasticsearch and its indices.

```shell
elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana --url https://[url-of-elasticsearch]:9200
```
Create enrollment token for usage with Kibana. This will be pasted inside of the Kibana web page when prompted after a fresh installation.
>`-s` indicates the scope, and can either be `kibana` or `node`
>`--url` indicates the URL to use for connection to Elasticsearch.

```sh
bin/kibana-verification-code
```
Gets verification code for Kibana enrollment.
## Installing & Configuring Logstash
We can install the `logstash` package with apt.

Install Logstash on machines that you wish to log from (i.e, an Apache web server).

Logstash configuration files can be found in `/etc/logstash/`.

Logstash will primarily be configured within `/etc/logstash/conf.d` where pipelines will be defined.

These pipelines serve the purpose of capturing and parsing logs, and forwarding the information to Elasticsearch.
### Logstash Pipelines