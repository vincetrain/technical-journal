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
## Scapy
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
Every packet in Scapy will have 3 mandatory layers consisting of OSI layers 2-4, and then a payload consisting of any OSI layer 5-7 data.
The structure is as follows:
- **Ether()** : OSI Layer 2 (Data-link)
- **IP()** : OSI Layer 3 (Network)
- **TCP() or UDP() or ICMP()** : OSI Layer 4 (Transport) Protocol
	  Although ICMP is a Layer 3 protocol, it is included here.
- **Payload** : OSI Layers 5-7 (Session, Presentation, Application)
This document will focus mainly on how OSI layers 2-4 are represented in Scapy packets.

Individual layers of a packet in Scapy can be referred to as follows:
```python
print(pkt[Ether]) # Prints the entire Data-link layer of the packet.
```

Entire packets in Scapy may be constructed as follows:
```python
pkt = Ether()/IP()/TCP()/Raw()
```
> Layers can be removed where needed. 
> For example, if we want to use ARP to determine destination address:
> `pkt = IP()/TCP()
#### Ether()
```python
datalink = Ether()
datalink.src = 'src-mac-address'
datalink.dst = 'dst-mac-address'
```
#### IP()
```python
network = IP()
network.src = 'src-ip-address'
network.st = 'dst-ip-address'
```
#### TCP()
```python
transport = TCP()
transport.sport = source_port_integer
transport.dport = destination_port_integer
transport.flags = flags_string
transport.seq = sequence_integer
transport.ack = ack_integer
```
#### UDP()
#### ICMP()
```python
icmp = ICMP()
```