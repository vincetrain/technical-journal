[Install Elasticsearch with a Debian package](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-with-debian-package)

[Create or update roles (Elasticsearch)](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-security-put-role)

[Secure communication with an on-premise Elasticsearch cluster](https://www.elastic.co/docs/reference/logstash/secure-connection#es-security-onprem)  

[Secure communication with Logstash (Filebeat)](https://www.elastic.co/docs/reference/beats/filebeat/configuring-ssl-logstash)
## Preparing apt Repository
Install signing key
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

Install apt repository
```sh
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
sudo apt-get update
```
## Installing & Configuring Elasticsearch
Before installing Elasticsearch, **ensure the machine's hostname is the hostname that is desired**! Elasticsearch autogenerates certificates for HTTPS usage when installed. **Mismatching hostnames from hostname changes after installation will mess this up!**

We can now install the `elasticsearch` package with apt.

**Successful installation will show the `elastic` user's in shell. Consider noting it down or adding it to an environment variable**

The `elasticsearch` binary (and other supporting binaries) are stored at `/usr/share/elasticsearch/bin`.

Elasticsearch configurations can be found inside of `/etc/elasticsearch/elasticsearch.yml`.

Check Elasticsearch health
```sh
curl --cacert /etc/elasticsearch/certs/http_ca.crt -u [username] https://localhost:9200/_cluster/health
```

For Elasticsearch to work, ensure at least 1 correct initial master node is set inside `/etc/elasticsearch/elasticsearch.yml`.
```yml
node.name: example-node-1
...
cluster.initial_master_nodes: ["example-node-1"]
```
>This solves error status 503, `"master_not_discovered_exception"`
## Installing & Configuring Kibana
We can install the `kibana` package with apt.

Kibana binaries can be found at `/usr/share/kibana/bin`.

The Kibana webserver will be running at [localhost:5601](localhost:5601)

Create a user. This is the user we will use with Kibana. You will be prompted to enter a password for the user after running this command.
```sh
elasticsearch/bin/elasticsearch-users useradd [user]
```

Add required user roles for unlimited Kibana and Elasticsearch usage
```sh
elasticsearch/bin/elasticsearch-users roles [user] -a kibana_admin,superuser
```
>The `kibana_admin` role allows unlimited to usage of Kibana, including administrative use. 
>The `superuser` role allows unlimited read and write access (and usage) of Elasticsearch and its indices.

Create enrollment token for usage with Kibana. This will be pasted inside of the Kibana web page when prompted after a fresh installation.
```shell
elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana --url https://[url-of-elasticsearch]:9200
```
>`-s` indicates the scope, and can either be `kibana` or `node`
>`--url` indicates the URL to use for connection to Elasticsearch.

Gets verification code for Kibana enrollment.
```sh
bin/kibana-verification-code
```
### Setup SSL with Kibana
Generate local CA using elasticsearch-certutil
```sh
elasticsearch/bin/elasticsearch-certutil 
```
### Stack Monitoring
To setup stack monitoring run the `bin/kibana-encryption-keys` command, and add the generated keys to `/etc/kibana/kibana.yml`
```sh
bin/kibana-encryption-keys generate
```

After this is done, stack monitoring can be configured easily in Kibana's "Management>Stack Monitoring" section.
## Installing & Configuring Logstash
We can install the `logstash` package with apt.

Install Logstash on machines that you wish to log from (i.e, an Apache web server).

Logstash configuration files can be found in `/etc/logstash/`.

Logstash binaries can be found at `/usr/share/logstash/bin`.

For Logstash, we add an Elasticsearch role with required permissions.
```HTTP
POST /security/role/logstash_user
{
	"description": "Grants necessary privileges for logstash functionality",
	"cluster": ["all"],
	"indices": [
		{
			"names": ["your-index"],
			"privileges": ["all"]
		}
	]
}
```
Refer to [[#Installing & Configuring Kibana]] to setup a user for Logstash

Because Elasticsearch was autoconfigured with SSL, we need to copy Elasticsearch's SSL certificate to our Logstash instance.
```sh
mkdir /etc/logstash/certs
cp /etc/elasticsearch/certs/http_ca.crt /etc/logstash/certs
## you may also want to chmod or chown the cert depending on how you're using logstash
```
### Logstash Pipelines
Logstash pipelines are, by default, stored in `/etc/logstash/conf.d`, but it is possible to store pipelines elsewhere.

To run pipelines at a specified directory, us the `-f` flag and point it towards the pipeline path.
> It may be worth adding `/usr/share/logstash/bin` to PATH for convenience.

If using Vim, ensure the following configurations are set for tabs to be recognized and usable in pipelines
```vimrc
set tabstop=4
set shiftwidth=4
set expandtab
```

Empty Logstash configuration template
```conf
input {

}

filter {

}

output {

}

```

Make Logstash read from a file
```conf
input {
	file {
		path => "/path/to/logs"
		# Optional, define sincedb_path as null to re-read ALL logs every run
		start_position => "beginning"
		sincedb_path => "/dev/null"
	}
}
```
>sincedb_path should be "NUL" on Windows

Make Logstash output to Elasticsearch with SSL using previous certs
```conf
output {
	elasticsearch {
		hosts => ["https://elasticsearch-url:9200"]
		ssl_enabled => true
		ssl_certificate_authorities => ["/etc/logstash/certs/http_ca.crt"]
		user => "logstash_user"
		password => "your-password-here"
		index => "example-index"
	}
}
```
> Also consider adding the `logstash` group to the user who will be running Logstash if running via its binary (requires re-login/reboot to work).
## Filebeat Configuration
For Filebeat to work over SSL, use `elasticsearch/bin/elasticsearch-certutil` to create a certificate.

Configure `filebeat.yml` to 

### Logging Apache2 Events
System logs are stored at `/var/log/`. To allow Logstash to access these logs, give the `logstash` user the `adm` role.
```sh
usermod -aG adm logstash
```

Apache has two types of logs (access and error), so we will create two individual pipelines for each type of error.