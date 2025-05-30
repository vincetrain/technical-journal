[Launch Splunk Web](https://docs.splunk.com/Documentation/Splunk/9.4.2/SearchTutorial/StartSplunk#Start_Splunk_Enterprise_on_Linux)

[List of pretrained source types](https://docs.splunk.com/Documentation/Splunk/9.4.2/Data/Listofpretrainedsourcetypes)

[props.conf](https://docs.splunk.com/Documentation/Splunk/latest/admin/propsconf)

[Configure the universal forwarder using configuration files](https://help.splunk.com/en/splunk-enterprise/forward-and-process-data/universal-forwarder-manual/9.4/configure-the-universal-forwarder/configure-the-universal-forwarder-using-configuration-files)

[inputs.conf](https://docs.splunk.com/Documentation/Splunk/9.4.2/Admin/Inputsconf)

[outputs.conf](https://docs.splunk.com/Documentation/Splunk/9.4.2/Admin/Outputsconf)

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
## Configuring Log Forwarding
In Splunk Enterprise, navigate to `Settings > Forwarding and receiving > Configure receiving`, and configure a port to receive data on.
### Universal Forwarder
Install the Universal Splunk Forwarder on the host that will be monitored.

If installing through `apt`, the home directory of Splunk will be found in `/opt/splunkforwarder`.
If installing on Windows, the home directory of Splunk will be found in `C:\Program Files\SplunkUniversalForwarder` by default.

Configure forwarding to Splunk receiver
```sh
bin/splunk add forward-server [ip-of-splunk-receiver]:[receiving-port]
```
> Receiving port refers to the previously configured port in `Settings > Forwarding and receiving > Configure receiving`.
> This can be further configured in `etc/system/local/outputs.conf`

Configure specific outputs in `etc/system/local/outputs.conf`
```conf
[tcpout:output_group]
server = ip.of.server.1,ip.of.server.2
```

Configure basic monitoring
```sh
bin/splunk add monitor /path/to/monitored
```

Configure monitor inputs with specific source type names in `etc/system/local/inputs.conf`.
```conf
[monitor:///path/to/monitored]
sourcetype = source_name
index = index_name
_TCP_ROUTING = output_group
```
> Splunk comes pre-shipped with source types that define automatic parsing behavior when indexed. [List of pretrained source types](https://docs.splunk.com/Documentation/Splunk/9.4.2/Data/Listofpretrainedsourcetypes).
> Different types of inputs can be monitored. [inputs.conf](https://docs.splunk.com/Documentation/Splunk/9.4.2/Admin/Inputsconf).
> If monitoring Windows Event Logs, it is wise to install Universal Forwarder as a local user.
### Changing metadata fields

Configure `etc/system/local/transforms.conf` to define transform filter.
```conf
[set_metadata_subclass]
REGEX = .
DEST_KEY = _MetaData:Index
FORMAT = index_name
```
> `set_metadata_subclass` can be any name, but should be unique in the `transforms.conf`.
> `DEST_KEY` specifies what key in the metadata to apply the transform on.
> `FORMAT` specifies what to format the `DEST_KEY` as (in this example, we are setting the Index metadata key to `index_name`).

Configure `etc/system/local/props.conf` to apply the transform filter on source type data.
```conf
[source-type]
TRANSFORM-set_metadata = set_metadata_subclass
```
