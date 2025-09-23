Client OS (AKA sending host AKA connection initiator)
- Layer 4
	- Assign either (1) sending applications designated port or (2) ephemeral source port
- Layer 3
	- Check destination IP against its own routing table (maps IPs → next hop IPs + egress interface) 
	- It's own routing table only has 2 entries:
		- ex A. `192.168.10.0/24 dev eth0` - an entry that says 'all other devices on my local subnet'
		- ex B. `0.0.0.0/0 via 192.168.10.1 dev eth0` - an fallback entry which will be the longest matching prefix when no other entries found, an entry that says 'send to the default gateway'
	- in both (A) and (B), now it knows (1) the next-hop IP address and (2) egress interface, but it doesn't know the MAC address for layer 2 headers
- Layer 2
	- Checks the next-hop IP against its ARP cache (maps IPs → MAC addresses)
		- if not found, performs an ARP broadcast
			- Ethernet frame with EtherType = ARP and Dest MAC = ff:ff....
			- the switch receives it then floods it (AKA layer 2 broadcasts it) out all ports
				- although routers also receive this, they drop it
			- when hosts receive it, all drop except the target IP, who sends an ARP reply
		- if found, adds MAC address to layer 2 header, sends out, finished.

Layer 2 devices (usually Switches)
- Layer 4
	- Doesn't touch or interact with Layer 4 headers
- Layer 3
	- Doesn't touch or interact with Layer 3 headers
- Layer 2
	- Looks at the destination MAC in layer 2 headers
	- If destination is broadcast, flood it out to all ports in same VLAN
	- if destination is not broadcast, checks it against its MAC table
		- MAC tables map MAC addresses → switch ports
		- MAC tables are built up passively over time by recording all incoming packets
	- If it doesn't find an entry,
		- flood the frame out all ports in the same VLAN.
		- eventually a reply will come from the right place, and so now the MAC table has the correct entry
	- If it finds matching entry, sends out, finished

Layer 3 devices (Routers, Firewalls, etc)
- Layer 4
	- Doesn't touch or interact with Layer 4 headers
- Layer 3
	- Check destination IP with Routing Table (maps longest matching prefix IPs → egress interface + next-hop IP)
	- also decrements TTL by 1 and re-calculates new checksum
	- for NAT routers, use NAT table to
		- if outbound packet, replace ( source private IP ↔  own public IP + port), called multiplex
		- if inbound packet, replace ( destination public IP + port ↔ private IP ), called demultiplex
- Layer 2
	- check next-hop IP against ARP table
	- if not found, ARP broadcast
