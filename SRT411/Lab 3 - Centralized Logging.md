[Launch Splunk Web](https://docs.splunk.com/Documentation/Splunk/9.4.2/SearchTutorial/StartSplunk#Start_Splunk_Enterprise_on_Linux)
## Installing Splunk
If installing through `apt`, the home directory of Splunk will be found in `/opt/splunk`.

Start Splunk and configure for the first time
```sh
bin/splunk start
```
> Splunk will be available at over 0.0.0.0:8000.

Stop Splunk
```sh
bin/splunk stop
```
## Configuring Forwarders
In Splunk, navigate to `Settings > Forwarding and receiving > Configure receiving`, and configure a port to receive data on.
### Linux
Install the Universal Splunk Forwarder on the host that will be monitored.

If installing through `apt`, the home directory of Splunk will be found in `/opt/splunkforwarder`.

Configure forwarding to Splunk receiver
```sh
bin/splunk add forward-server [ip-of-splunk-receiver]:[receiving-port]
```
> Receiving port refers to the previously configured port in `Settings > Forwarding and receiving > Configure receiving`.

Configure monitor inputs with specific source type names in `/etc/system/local/inputs.conf`.
```conf
[monitor:///path/to/monitored]
sourcetype = source_name
```

### Storing data in different indices per source type
