[Launch Splunk Web](https://docs.splunk.com/Documentation/Splunk/9.4.2/SearchTutorial/StartSplunk#Start_Splunk_Enterprise_on_Linux)
## Installing Splunk
When installing through `apt`, the home directory of Splunk will be found in `/opt/splunk`.

Start Splunk
```sh
bin/splunk start
```
> Splunk will be available at over 0.0.0.0:8000.
> Fresh installations will require 



## Configuring Forwarders
In Splunk, navigate to `Settings > Forwarding and receiving > Configure receiving`, and configure a port to receive data on.

### Linux
Install the Universal Splunk Forwarder on the host that will be monitored.

When installing through `apt`, the home directory of Splunk will be found in `/opt/splunkforwarder`.

Configure forwarding to Splunk receiver
```sh
bin/splunk add forward-server [ip-of-splunk-receiver]:[receiving-port]
```
> Receiving port refers to the previously configured port in `Settings > Forwarding and receiving > Configure receiving`.

Configure monitor inputs
```sh
bin/splunk add monitor [path-to-logs]
```
