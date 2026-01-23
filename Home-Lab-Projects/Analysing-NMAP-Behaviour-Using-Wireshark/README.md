# Analysing NMAP behaviour using Wireshark

Machines:

Kali Linux VM (Attacker)
Labuntu VM (Victim)


First, before starting I ensure that both devices are on the same subnet to ensure pfsense doesn’t intervene with our nmap scans.

Secondly I will install and start an SSH server on my Labuntu VM so we have an open port to scan with nmap.

Next we will scan for the open port on our Kali VM using nmap.

We successfully found our SSH server on TCP port 22.

Now I will run Wireshark on our attacker VM to capture our packet’s behaviour and run a full TCP scan on port 22. (I am using a filter on wireshark so that we only see packets sent between the attacker and victim VM for a clear view)

Looking at the results we can see that nmap has made a full TCP connection:

[SYN] – “can we talk on port 22?”
[SYN, ACK] – “yes port 22 is listening”
[ACK] – “connection established”
[RST, ACK] – “abort connection (RST/Reset), acknowledged (ACK)

Next I will run a SYN Scan (Stealth Scan)

This time we can see that our nmap scan took .44 seconds as opposed to the .19 seconds it took us to run a full TCP scan. We can also see that a full TCP connection was never made, nmap reset/aborted before the full handshake could occur. It’s worth noting that both scans thus far have returned the same result for the attacker.

[SYN] – “can we talk on port 22?”
[SYN, ACK] – “yes port 22 is listening”
[RST] “abort connection”

This time we will use an ACK Scan

The results here are quite different. Rather than telling us if a port is open or closed it tells us if the port is reachable (unfiltered) or behind a firewall (filtered). It does this by sending a TCP packet with only the ACK flag set and usually with a random string of numbers. If we get an RST response then we know the port is reachable but if we get no result the port is likely behind a firewall.

Next up we will run a null scan on an open port (22) and again on a closed port (60)

From these results we can see that nmap sends a TCP packet with no flags set at all and if the port is open the packet is dropped and nothing is returned, however as shown when we scanned a closed port (60) it returns a reset [RST] packet that indicates the port is closed. What is also worth noting is that in the nmap console if we find an open port it also tells us the state of the firewall (filtered/unfiltered)

Next we will look at a FIN Scan

As we can see a FIN scan will send only the FIN (Finish flag) in order to probe open ports in an attempt to bypass basic firewalls by not initiating a full handshake. We can see that when a port is closed it responds with a RST packet however when we scan an open port the packet sent is discarded (no response).

Something interesting we can see in the FIN scan results is that port 22 is shown as open but filtered, however we already know that there is no firewall interfering with our packets by looking at the results of our ACK scan which showed us that our packets were not filtered.

When performing a FIN scan it is looking for either a RST response or none at all to determine the state of the scanned port. It will show as filtered as it didn’t get a response and but it still does not know if that is because the port is open or if the packet was filtered by a firewall, which is why the port is showing “filtered” and “open” when we have already determined there is no firewall blocking our packets.

In short:

ACK Scan – “Did my packet get there?”
FIN Scan – “Did anyone respond?”
