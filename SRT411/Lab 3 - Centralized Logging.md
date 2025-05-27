## Installing Splunk
For this lab I will be installing Splunk Enterprise using the portable installation on the website.

This involves creating a new user and a new systemctl service. Refer to [[Lab 2 - ELK Stack Clustering and Index Management#Configuring Elasticsearch as a service]].

Start Splunk for the first time
```sh
bin/splunk start --accept-license --answer-yes
```
> Splunk will be available at over localhost:8000.