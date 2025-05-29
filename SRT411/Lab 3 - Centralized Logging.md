[Launch Splunk Web](https://docs.splunk.com/Documentation/Splunk/9.4.2/SearchTutorial/StartSplunk#Start_Splunk_Enterprise_on_Linux)
## Installing Splunk
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

### Linux
Install the Universal Splunk Forwarder on the host that will be monitored.

Configure forwarding to Splunk receiver
```sh
bin/splunk add forward-server [ip-of-splunk-receiver]:[receiving-port]
```
> Receiving port refers to the previously configured port in `Settings > Forwarding and receiving > Configure receiving`.

Configure monitor inputs
```sh
bin/splunk add monitor [path-to-logs]
```
