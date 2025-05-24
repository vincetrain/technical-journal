## netcat basics
Listen for connections with netcat
```bash
nc -l [ip] [port]
```

Connect with netcat
```sh
nc [ip] [port]
```

Output netcat listener to file
```sh
nc -l [ip] [port] > file.txt
```

Input file to netcat
```sh
nc [ip] [port] < file.txt
```
## ScaPy
Sniff for packets
```python
pkt = sniff(
	iface=['interface-name'], 
	filter='berkeley-packet-filter', 
	prn=method-handling-sniffed-packets
)
```

Send packet and sniff for first response
```python
revc_pkt = sr1(send_pkt) # sends send_pkt, and stores the first response packet to recv_pkt
```
### Packet Structuring
Every packet in Scapy 
