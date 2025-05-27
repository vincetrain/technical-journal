[Launch Splunk Web](https://docs.splunk.com/Documentation/Splunk/9.4.2/SearchTutorial/StartSplunk#Start_Splunk_Enterprise_on_Linux)
## Installing Splunk
For this lab I will be installing Splunk Enterprise using the portable installation on the website.

This involves creating a new user and a new systemctl service. Refer to [[Lab 2 - ELK Stack Clustering and Index Management#Configuring Elasticsearch as a service]].
> This should be done AFTER we've started Splunk for the first time and accepted the license/created the administrator user.

Start Splunk and configure for the first time
```sh
bin/splunk start
```
> Splunk will be available at over 0.0.0.0:8000.

Configure systemd service for Splunk
```service
[Unit]
Description=Splunk
[Service]
ExecStart=/home/splunk/splunk/bin/splunk start
RemainAfterExit=yes
ExecStop=/home/splunk/splunk/bin/splunk stop
[Install]
WantedBy=default.target
```
> We enable RemainAfterExit as Splunk forks its own splunkd daemon. This is so we don't lose track of splunk (allows us to systemctl stop splunk).

## Configuring Forwarders
In Splunk, navigate to `Settings > Forwarding and receiving > Configure receiving`, and configure a port to receive data on.

Install the Universal Splunk Forwarder on 