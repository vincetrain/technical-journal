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


## scapy
Sniff for packets
```python
pkt = sniff(
	iface=['interface-name'], 
	filter='berkeley-packet-filter', 
	prn=method-handling-sniffed-packets
)
```

Send packet and sniff
```python

```