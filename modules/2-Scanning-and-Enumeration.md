# Scanning and Enumeration

> ⚡︎ **This chapter has practical labs for [Scanning Networks (1)](https://github.com/Samsar4/Ethical-Hacking-Labs/tree/master/2-Scanning-Networks) and [Enumeration (2)](https://github.com/Samsar4/Ethical-Hacking-Labs/tree/master/3-Enumeration)**

**Network Scanning** - Discovering systems on the network (can be hosts, switches, servers, routers, firewalls and so on) and looking at what ports are open as well as applications/services and their respective versions that may be running.

**In general network scanning have <u>three main objectives</u>:**
1. Scanning for live devices, OS, IPs in use.
    - **`Server at 192.168.60.30`**
2. Looking for Ports open/closed.
    - **`The server 192.168.60.30 have TCP port 23 (Telnet) running`**
3. Search for vulnerabilities on services scanned.
    - **`The Telnet service is cleartext and have many vulnerabilities published`**

**Connectionless Communication** - UDP packets are sent without creating a connection.  Examples are TFTP, DNS (lookups only) and DHCP

**Connection-Oriented Communication** - TCP packets require a connection due to the size of the data being transmitted and to ensure deliverability

### <u>Scanning Methodology</u>

- **Check for live systems** - Ping or other type of way to determine live hosts
- **Check for open ports** - Once you know live host IPs, scan them for listening ports
- **Scan beyond IDS** - If needed, use methods to scan  beyond the detection systems; evade IDS using proxies, spoofing, fragmented packets and so on
- **Perform banner grabbing** - Grab from servers as well as perform OS fingerprinting (versions of the running services)
- **Scan for vulnerabilities** - Use tools to look at the vulnerabilities of open systems
- **Draw network diagrams** - Shows logical and physical pathways into networks
- **Use proxies** - Obscures efforts to keep you hidden
- **Pentest Report** - Document everything that you find

## <u>Identifying Targets</u>

- The easiest way to scan for live systems is through ICMP.

- It has it's shortcomings and is sometimes blocked on hosts that are actually live.

- **Message Types and Returns**
  - Payload of an ICMP message can be anything; RFC never set what it was supposed to be.  Allows for covert channels
  - **Ping sweep** - easiest method to identify multiple hosts on subnet.
    - You can use **nmap** with `-sn` flag to perform a Ping Sweep:
        - **`nmap -sn 192.168.1.0/24`**
        - *This will perform a ping on 256 IP addresses on this subnet in seconds, showing which hosts are up.*
  - **ICMP Echo scanning** - sending an ICMP Echo Request to the network IP address
  - An ICMP return of type 3 with a code of 13 indicates a poorly configured firewall
  - **Ping scanning tools**
    - Nmap
    - Angry IP Scanner
    - Solar-Winds Engineer Toolkit
    - Advanced IP Scanner
    - Pinkie
  - Nmap virtually always does a ping sweep with scans unless you turn it off

* **Important ICMP codes**

  | ICMP Message Type           | Description and Codes                                        |
  | --------------------------- | ------------------------------------------------------------ |
  | 0:  Echo Reply              | Answer to a Type 8 Echo Request                              |
  | 3:  Destination Unreachable | Error message followed by these codes:<br />0 - Destination network unreachable<br />1 - Destination host unreachable<br />6 - Network unknown<br />7 - Host unknown<br />9 - Network administratively prohibited<br />10 - Host administratively prohibited<br />13 - Communication administratively prohibited |
  | 4: Source Quench            | A congestion control message                                 |
  | 5: Redirect                 | Sent when there are two or more gateways available for the sender to use.  Followed by these codes:<br />0 - Redirect datagram for the network<br />1 - Redirect datagram for the host |
  | 8:  Echo Request            | A ping message, requesting an echo reply                     |
  | 11:  Time Exceeded          | Packet took too long to be routed (code 0 is TTL expired)    |


## <u>Port Discovery - Basic Concepts</u>


<img width="60%" src="https://gist.githubusercontent.com/Samsar4/62886aac358c3d484a0ec17e8eb11266/raw/f8b2f839cee428a08506a9831a5d2f066e7301e7/openport.png">


- **Knocking the door**
  - The hacker above sends a SYN packet to port 80 on the server.
    - If server returns **SYN-ACK packet** = the port is **open**
    - If server returns **RST (reset) packet** = the port is **closed**

***

### ☞ Keep in mind the TCP Flags: 
| Flag | Name           | Function                                                     |
| ---- | -------------- | ------------------------------------------------------------ |
| SYN  | Synchronize    | Set during initial communication.  Negotiating of parameters and sequence numbers |
| ACK  | Acknowledgment | Set as an acknowledgement to the SYN flag.  Always set after initial SYN |
| RST  | Reset          | Forces the termination of a connection (in both directions)  |
| FIN  | Finish         | Ordered close to communications                              |
| PSH  | Push           | Forces the delivery of data without concern for buffering    |
| URG  | Urgent         | Data inside is being sent out of band.  Example is cancelling a message |

***

<img width="60%" src="https://gist.githubusercontent.com/Samsar4/62886aac358c3d484a0ec17e8eb11266/raw/649d665844b3a3e5c57983e6003616eff3292280/nmap-probe.png">


- **Checking if Stateful Firewall is present**
  - The hacker above sends an **ACK segment/packet** on the first interaction *(without three-way handshake)*.
    - If server returns **no response** means that might have a stateful firewall handling proper sessions
    - If server returns **RST packet** means that have no stateful firewall

⚠️ **This can be easily achieved using nmap**

# Using Nmap

> ⚡︎ **It is highly recommended to try out and explore the nmap on any virtual environment; I made a couple [practical labs[1]](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/2-Scanning-Networks/4-Nmap.md) [[2]](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/2-Scanning-Networks/5-NmapDecoyIP.md) [[3]](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/3-Enumeration/1-Enumerating-with-Nmap.md) to help you understand the functionality of nmap. <u>Practice is everything.</u>**

### <u>Port Scan Types</u>

- **Full connect** - TCP connect or full open scan - full connection and then tears down with RST
  - Easiest to detect, but most reliable
  - **`nmap -sT`**
- **Stealth** - half-open scan or SYN scan - only SYN packets sent.  Responses same as full.
  - Useful for hiding efforts and evading firewalls
  - **`nmap -sS`**
- **Inverse TCP flag** - uses FIN, URG or PSH flag.  Open gives no response.  Closed gives RST/ACK
  - **`nmap -sN`** (Null scan)
  - **`nmap -sF`** (FIN scan)
- **Xmas** - so named because all flags are turned on so it's "lit up" like a Christmas tree
  - Responses are same as Inverse TCP scan
  - Do not work against Windows machines
  - **`nmap -sX`**
- **ACK flag probe** - multiple methods
  - TTL version - if TTL of RST packet < 64, port is open
  - Window version - if the Window on the RST packet is anything other than 0, port open
  - Can be used to check filtering.  If ACK is sent and no response, stateful firewall present.
  - **`nmap -sA`** (ACK scan)
  - **`nmap -sW`** (Window scan)
- **IDLE Scan** - uses a third party to check if a port is open
  - Looks at the IPID to see if there is a response
  - Only works if third party isn't transmitting data
  - Sends a request to the third party to check IPID id; then sends a spoofed packet to the target with a return of the third party; sends a request to the third party again to check if IPID increased.
    - IPID increase of 1 indicates port closed
    - IPID increase of 2 indicates port open
    - IPID increase of anything greater indicates the third party was not idle
  - **`nmap -sI <zombie host>`**

## <u>Nmap Switches</u>

| Switch          | Description                                                  |
| :---------------: | :------------------------------------------------------------ |
| -sA             | ACK scan                                                     |
| -sF             | FIN scan                                                     |
| -sI             | IDLE scan                                                    |
| -sL             | DNS scan (list scan)                                         |
| -sN             | NULL scan                                                    |
| -sO             | Protocol scan (tests which IP protocols respond)             |
| -sP             | Ping scan                                                    |
| -sR             | RPC scan                                                     |
| -sS             | SYN scan                                                     |
| -sT             | TCP connect scan                                             |
| -sW             | Window scan                                                  |
| -sX             | XMAS scan                                                    |
| -A              | OS detection, version detection, script scanning and traceroute |
| -PI             | ICMP ping                                                    |
| -Po             | No ping                                                      |
| -PS             | SYN ping                                                     |
| -PT             | TCP ping                                                     |
| -oN             | Normal output                                                |
| -oX             | XML output                                                   |
| -T0 through -T2 | Serial scans.  T0 is slowest                                 |
| -T3 through -T5 | Parallel scans.  T3 is slowest                               |

- Nmap runs by default at a T3 level
- **Fingerprinting** - another word for port sweeping and enumeration

## Port Specification

<table data-rows="10" data-cols="3" data-tve-custom-colour="87922012"><thead><tr><th data-tve-custom-colour="11099273"><div><p>Switch</p></div></th><th data-tve-custom-colour="5315753"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="723491"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="69668880"><div><p>-p</p></div></td><td data-tve-custom-colour="35630518"><div><p>nmap 192.168.1.1 -p 21</p></div></td><td data-tve-custom-colour="42544505"><div><p>Port scan for port x</p></div></td></tr><tr><td data-tve-custom-colour="23760083"><div><p>-p</p></div></td><td data-tve-custom-colour="91525947"><div><p>nmap 192.168.1.1 -p 21-100</p></div></td><td data-tve-custom-colour="85478031"><div><p>Port range</p></div></td></tr><tr><td data-tve-custom-colour="60930584"><div><p>-p</p></div></td><td data-tve-custom-colour="95405266"><div><p>nmap 192.168.1.1 -p U:53,T:21-25,80</p></div></td><td data-tve-custom-colour="16642894"><div><p>Port scan multiple TCP and UDP ports</p></div></td></tr><tr><td data-tve-custom-colour="56382335"><div><p>-p-</p></div></td><td data-tve-custom-colour="1929917"><div><p>nmap 192.168.1.1 -p-</p></div></td><td data-tve-custom-colour="68238518"><div><p>Port scan all ports</p></div></td></tr><tr><td data-tve-custom-colour="2630941"><div><p>-p</p></div></td><td data-tve-custom-colour="59723601"><div><p>nmap 192.168.1.1 -p http,https</p></div></td><td data-tve-custom-colour="38965266"><div><p>Port scan from service name</p></div></td></tr><tr><td data-tve-custom-colour="86193851"><div><p>-F</p></div></td><td data-tve-custom-colour="83865922"><div><p>nmap 192.168.1.1 -F</p></div></td><td data-tve-custom-colour="52914446"><div><p>Fast port scan (100 ports)</p></div></td></tr><tr><td data-tve-custom-colour="9925025"><div><p>--top-ports</p></div></td><td data-tve-custom-colour="5767817"><div><p>nmap 192.168.1.1 --top-ports 2000</p></div></td><td data-tve-custom-colour="97877956"><div><p>Port scan the top x ports</p></div></td></tr><tr><td data-tve-custom-colour="77359869"><div><p>-p-65535</p></div></td><td data-tve-custom-colour="97638659"><div><p>nmap 192.168.1.1 -p-65535</p></div></td><td data-tve-custom-colour="80200270"><div><p>Leaving off initial port in range <br/>makes the scan start at port 1</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="77359869"><div><p>-p0-</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="97638659"><div><p>nmap 192.168.1.1 -p0-</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="80200270"><div><p>Leaving off end port in range</p><p>makes the scan go through to port 65535</p></div></td></tr></tbody></table><p>&nbsp;</p>

## Service and Version Detection

<table data-rows="10" data-cols="3" data-tve-custom-colour="20630216"><thead><tr><th data-tve-custom-colour="32713170"><div><p>Switch</p></div></th><th data-tve-custom-colour="9791280"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="50448944"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="52718078"><div><p>-sV</p></div></td><td data-tve-custom-colour="65314143"><div><p>nmap 192.168.1.1 -sV</p></div></td><td data-tve-custom-colour="15678704"><div><p>Attempts to determine the version of the service running on port</p></div></td></tr><tr><td data-tve-custom-colour="17175428"><div><p>-sV --version-intensity</p></div></td><td data-tve-custom-colour="33430509"><div><p>nmap 192.168.1.1 -sV --version-intensity 8</p></div></td><td data-tve-custom-colour="61291488"><div><p>Intensity level 0 to 9. Higher number increases possibility of correctness</p></div></td></tr><tr><td data-tve-custom-colour="41449000"><div><p>-sV --version-light</p></div></td><td data-tve-custom-colour="64128505"><div><p>nmap 192.168.1.1 -sV --version-light</p></div></td><td data-tve-custom-colour="16090168"><div><p>Enable light mode. Lower possibility of correctness. Faster</p></div></td></tr><tr><td data-tve-custom-colour="72668320"><div><p>-sV --version-all</p></div></td><td data-tve-custom-colour="99337597"><div><p>nmap 192.168.1.1 -sV --version-all</p></div></td><td data-tve-custom-colour="38992774"><div><p>Enable intensity level 9. Higher possibility of correctness. Slower</p></div></td></tr><tr><td data-tve-custom-colour="78539889"><div><p>-A</p></div></td><td data-tve-custom-colour="54257225"><div><p>nmap 192.168.1.1 -A</p></div></td><td data-tve-custom-colour="33531863"><div><p>Enables OS detection, version detection, script scanning, and traceroute</p></div></td></tr></tbody></table><p>&nbsp;</p>

## OS Detection

<table data-rows="10" data-cols="3" data-tve-custom-colour="40603157"><thead><tr><th data-tve-custom-colour="40504158"><div><p>Switch</p></div></th><th data-tve-custom-colour="39303821"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="21543609"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="81681967"><div><p>-O</p></div></td><td data-tve-custom-colour="53779077"><div><p>nmap 192.168.1.1 -O</p></div></td><td data-tve-custom-colour="7125853"><div><p>Remote OS detection using TCP/IP <br/>stack fingerprinting</p></div></td></tr><tr><td data-tve-custom-colour="39319104"><div><p>-O --osscan-limit</p></div></td><td data-tve-custom-colour="41858044"><div><p>nmap 192.168.1.1 -O --osscan-limit</p></div></td><td data-tve-custom-colour="47630793"><div><p>If at least one open and one closed <br/>TCP port are not found it will not try <br/>OS detection against host</p></div></td></tr><tr><td data-tve-custom-colour="43782516"><div><p>-O --osscan-guess</p></div></td><td data-tve-custom-colour="61671299"><div><p>nmap 192.168.1.1 -O --osscan-guess</p></div></td><td data-tve-custom-colour="46353883"><div><p>Makes Nmap guess more aggressively</p></div></td></tr><tr><td data-tve-custom-colour="38710923"><div><p>-O --max-os-tries</p></div></td><td data-tve-custom-colour="50876041"><div><p>nmap 192.168.1.1 -O --max-os-tries 1</p></div></td><td data-tve-custom-colour="92071500"><div><p>Set the maximum number x of OS <br/>detection tries against a target</p></div></td></tr><tr><td data-tve-custom-colour="36246445"><div><p>-A</p></div></td><td data-tve-custom-colour="49088290"><div><p>nmap 192.168.1.1 -A</p></div></td><td data-tve-custom-colour="81951753"><div><p>Enables OS detection, version detection, script scanning, and traceroute</p></div></td></tr></tbody></table><p>&nbsp;</p>

## Timing and Performance

<table data-rows="8" data-cols="3" data-tve-custom-colour="67426359"><thead><tr><th data-tve-custom-colour="56239768"><div><p>Switch</p></div></th><th data-tve-custom-colour="61172434"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="70450063"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="96848284"><div><p>-T0</p></div></td><td data-tve-custom-colour="16390657"><div><p>nmap 192.168.1.1 -T0</p></div></td><td data-tve-custom-colour="99779436"><div><p>Paranoid (0) Intrusion Detection <br/>System evasion</p></div></td></tr><tr><td data-tve-custom-colour="90598822"><div><p>-T1</p></div></td><td data-tve-custom-colour="56639310"><div><p>nmap 192.168.1.1 -T1</p></div></td><td data-tve-custom-colour="68900575"><div><p>Sneaky (1) Intrusion Detection System <br/>evasion</p></div></td></tr><tr><td data-tve-custom-colour="2255571"><div><p>-T2</p></div></td><td data-tve-custom-colour="1180821"><div><p>nmap 192.168.1.1 -T2</p></div></td><td data-tve-custom-colour="54409344"><div><p>Polite (2) slows down the scan to use <br/>less bandwidth and use less target <br/>machine resources</p></div></td></tr><tr><td data-tve-custom-colour="59873415"><div><p>-T3</p></div></td><td data-tve-custom-colour="88591673"><div><p>nmap 192.168.1.1 -T3</p></div></td><td data-tve-custom-colour="80756194"><div><p>Normal (3) which is default speed</p></div></td></tr><tr><td data-tve-custom-colour="63637259"><div><p>-T4</p></div></td><td data-tve-custom-colour="45398482"><div><p>nmap 192.168.1.1 -T4</p></div></td><td data-tve-custom-colour="16899626"><div><p>Aggressive (4) speeds scans; assumes <br/>you are on a reasonably fast and <br/>reliable network</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="63637259"><div><p>-T5</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="45398482"><div><p>nmap 192.168.1.1 -T5</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="16899626"><div><p>Insane (5) speeds scan; assumes you <br/>are on an extraordinarily fast network</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="73052435">&nbsp;</td><td colspan="1" rowspan="1" data-tve-custom-colour="13330189">&nbsp;</td><td colspan="1" rowspan="1" data-tve-custom-colour="85354161">&nbsp;</td></tr></tbody></table><p>&nbsp;</p>

<table data-rows="9" data-cols="3" data-tve-custom-colour="75605587"><thead><tr><th data-tve-custom-colour="13586691"><div><p>Switch</p></div></th><th data-tve-custom-colour="67670104"><div><p data-unit="px">Example input</p></div></th><th data-tve-custom-colour="22001055"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="36797711"><div><p>--host-timeout&nbsp;&lt;time&gt;</p></div></td><td data-tve-custom-colour="40415439"><div><p>1s; 4m; 2h</p></div></td><td data-tve-custom-colour="65853516"><div><p>Give up on target after this long</p></div></td></tr><tr><td data-tve-custom-colour="45657658"><div><p>--min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout&nbsp;&lt;time&gt;</p></div></td><td data-tve-custom-colour="82394169"><div><p>1s; 4m; 2h</p></div></td><td data-tve-custom-colour="39146150"><div><p>Specifies probe round trip time</p></div></td></tr><tr><td data-tve-custom-colour="3070409"><div><p>--min-hostgroup/max-hostgroup&nbsp;&lt;size&lt;size&gt;</p></div></td><td data-tve-custom-colour="71004712"><div><p>50; 1024</p></div></td><td data-tve-custom-colour="22579534"><div><p>Parallel host scan group <br/>sizes</p></div></td></tr><tr><td data-tve-custom-colour="13777720"><div><p>--min-parallelism/max-parallelism&nbsp;&lt;numprobes&gt;</p></div></td><td data-tve-custom-colour="6830765"><div><p>10; 1</p></div></td><td data-tve-custom-colour="68135110"><div><p>Probe parallelization</p></div></td></tr><tr><td data-tve-custom-colour="15515435"><div><p>--scan-delay/--max-scan-delay&nbsp;&lt;time&gt;</p></div></td><td data-tve-custom-colour="36552978"><div><p>20ms; 2s; 4m; 5h</p></div></td><td data-tve-custom-colour="54476562"><div><p>Adjust delay between probes</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="15515435"><div><p>--max-retries &lt;tries&gt;</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="36552978"><div><p>3</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="54476562"><div><p>Specify the maximum number <br/>of port scan probe retransmissions</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="15515435"><div><p>--min-rate&nbsp;&lt;number&gt;</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="36552978"><div><p>100</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="54476562"><div><p>Send packets no slower than&nbsp;&lt;numberr&gt; per second</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="15515435"><div><p>--max-rate &lt;number&gt;</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="36552978"><div><p>100</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="54476562"><div><p>Send packets no faster than&nbsp;&lt;number&gt; per second</p></div></td></tr></tbody></table><p>&nbsp;</p>

## NSE Scripts

<table data-rows="8" data-cols="3" data-tve-custom-colour="84942293"><thead><tr><th data-tve-custom-colour="54206681"><div><p>Switch</p></div></th><th data-tve-custom-colour="94833622"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="44020045"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="28223530"><div><p>-sC</p></div></td><td data-tve-custom-colour="1861219"><div><p>nmap 192.168.1.1 -sC</p></div></td><td data-tve-custom-colour="37113719"><div><p>Scan with default NSE scripts. Considered useful for discovery and safe</p></div></td></tr><tr><td data-tve-custom-colour="4025240"><div><p>--script default</p></div></td><td data-tve-custom-colour="13420588"><div><p>nmap 192.168.1.1 --script default</p></div></td><td data-tve-custom-colour="41758"><div><p>Scan with default NSE scripts. Considered useful for discovery and safe</p></div></td></tr><tr><td data-tve-custom-colour="49874311"><div><p>--script</p></div></td><td data-tve-custom-colour="54335880"><div><p>nmap 192.168.1.1 --script=banner</p></div></td><td data-tve-custom-colour="63692658"><div><p>Scan with a single script. Example banner</p></div></td></tr><tr><td data-tve-custom-colour="29010211"><div><p>--script</p></div></td><td data-tve-custom-colour="38815019"><div><p>nmap 192.168.1.1 --script=http*</p></div></td><td data-tve-custom-colour="38719696"><div><p>Scan with a wildcard. Example http</p></div></td></tr><tr><td data-tve-custom-colour="50309291"><div><p>--script</p></div></td><td data-tve-custom-colour="27379683"><div><p>nmap 192.168.1.1 --script=http,banner</p></div></td><td data-tve-custom-colour="67727190"><div><p>Scan with two scripts. Example http and banner</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="50309291"><div><p>--script</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="27379683"><div><p>nmap 192.168.1.1 --script "not intrusive"</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="67727190"><div><p>Scan default, but remove intrusive scripts</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="50309291"><div><p>--script-args</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="27379683"><div><p>nmap --script snmp-sysdescr --script-args snmpcommunity=admin 192.168.1.1</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="67727190"><div><p>NSE script with arguments</p></div></td></tr></tbody></table><p>&nbsp;</p>

## Useful NSE Script Examples

<table data-rows="8" data-cols="2" data-tve-custom-colour="81834986"><thead><tr><th data-tve-custom-colour="46164884"><div><p>Command</p></div></th><th data-tve-custom-colour="84676748"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="24377933"><div><p>nmap -Pn --script=http-sitemap-generator scanme.nmap.org</p></div></td><td data-tve-custom-colour="22583833"><div><p>http site map generator</p></div></td></tr><tr><td data-tve-custom-colour="28668006"><div><p>nmap -n -Pn -p 80 --open -sV -vvv --script banner,http-title -iR 1000</p></div></td><td data-tve-custom-colour="71143089"><div><p>Fast search for random web servers</p></div></td></tr><tr><td data-tve-custom-colour="24049541"><div><p>nmap -Pn --script=dns-brute domain.com</p></div></td><td data-tve-custom-colour="51645097"><div><p>Brute forces DNS hostnames guessing subdomains</p></div></td></tr><tr><td data-tve-custom-colour="7314780"><div><p>nmap -n -Pn -vv -O -sV --script smb-enum*,smb-ls,smb-mbenum,smb-os-discovery,smb-s*,smb-vuln*,smbv2* -vv 192.168.1.1</p></div></td><td data-tve-custom-colour="80246068"><div><p>Safe SMB scripts to run</p></div></td></tr><tr><td data-tve-custom-colour="17385457"><div><p>nmap --script whois* domain.com</p></div></td><td data-tve-custom-colour="52762961"><div><p>Whois query</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="17385457"><div><p>nmap -p80 --script http-unsafe-output-escaping scanme.nmap.org</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="52762961"><div><p>Detect cross site scripting vulnerabilities</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="17385457"><div><p>nmap -p80 --script http-sql-injection scanme.nmap.org</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="52762961"><div><p>Check for SQL injections</p></div></td></tr></tbody></table><p>&nbsp;</p>

## Firewall / IDS Evasion and Spoofing

<table data-rows="9" data-cols="3" data-tve-custom-colour="74018057"><thead><tr><th data-tve-custom-colour="66763798"><div><p>Switch</p></div></th><th data-tve-custom-colour="2497752"><div><p data-unit="px">Example</p></div></th><th data-tve-custom-colour="28891756"><div><p>Description</p></div></th></tr></thead><tbody><tr><td data-tve-custom-colour="42379653"><div><p>-f</p></div></td><td data-tve-custom-colour="85690233"><div><p>nmap 192.168.1.1 -f</p></div></td><td data-tve-custom-colour="23877951"><div><p>Requested scan (including ping scans) use tiny fragmented IP packets. Harder for packet filters</p></div></td></tr><tr><td data-tve-custom-colour="9831139"><div><p>--mtu</p></div></td><td data-tve-custom-colour="8445695"><div><p>nmap 192.168.1.1 --mtu 32</p></div></td><td data-tve-custom-colour="13714864"><div><p>Set your own offset size</p></div></td></tr><tr><td data-tve-custom-colour="53184099"><div><p>-D</p></div></td><td data-tve-custom-colour="27224948"><div><p>nmap -D 192.168.1.101,192.168.1.102,<br/>192.168.1.103,192.168.1.23 192.168.1.1</p></div></td><td data-tve-custom-colour="6906852"><div><p>Send scans from spoofed IPs</p></div></td></tr><tr><td data-tve-custom-colour="36290366"><div><p>-D</p></div></td><td data-tve-custom-colour="86484685"><div><p>nmap -D decoy-ip1,decoy-ip2,your-own-ip,decoy-ip3,decoy-ip4 remote-host-ip</p></div></td><td data-tve-custom-colour="1334841"><div><p>Above example explained</p></div></td></tr><tr><td data-tve-custom-colour="87774529"><div><p>-S</p></div></td><td data-tve-custom-colour="3864884"><div><p>nmap -S www.microsoft.com www.facebook.com</p></div></td><td data-tve-custom-colour="76224450"><div><p>Scan Facebook from Microsoft (-e eth0 -Pn may be required)</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="87774529"><div><p>-g</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="3864884"><div><p>nmap -g 53 192.168.1.1</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="76224450"><div><p>Use given source port number</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="87774529"><div><p>--proxies</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="3864884"><div><p>nmap --proxies http://192.168.1.1:8080, http://192.168.1.2:8080 192.168.1.1</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="76224450"><div><p>Relay connections through HTTP/SOCKS4 proxies</p></div></td></tr><tr><td colspan="1" rowspan="1" data-tve-custom-colour="87774529"><div><p>--data-length</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="3864884"><div><p>nmap --data-length 200 192.168.1.1</p></div></td><td colspan="1" rowspan="1" data-tve-custom-colour="76224450"><div><p>Appends random data to sent packets</p></div></td></tr></tbody></table><p>&nbsp;</p>


* **Example IDS Evasion command**
  - `nmap -f -t 0 -n -Pn –data-length 200 -D`
  - 192.168.1.101,192.168.1.102,192.168.1.103,192.168.1.23 192.168.1.1

* Source: https://www.stationx.net/nmap-cheat-sheet/



## <u>hping</u>
> ⚡︎ **Check the hping3 [practical lab](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/2-Scanning-Networks/1-hping3.md)**

Hping3 is a scriptable program that uses the Tcl language, whereby packets can be received and sent via a binary or string representation describing the packets.

- Another powerful ping sweep and port scanning tool
- Also can craft UDP/TCP packets
- You can make a TCP flood
- hping3 -1 IP address

| Switch  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| -1      | Sets ICMP mode                                               |
| -2      | Sets UDP mode                                                |
| -8      | Sets scan mode.  Expects port range without -p flag          |
| -9      | Listen mode.  Expects signature (e.g. HTTP) and interface (-I eth0) |
| --flood | Sends packets as fast as possible without showing incoming replies |
| -Q      | Collects sequence numbers generated by the host              |
| -p      | Sets port number                                             |
| -F      | Sets the FIN flag                                            |
| -S      | Sets the SYN flag                                            |
| -R      | Sets the RST flag                                            |
| -P      | Sets the PSH flag                                            |
| -A      | Sets the ACK flag                                            |
| -U      | Sets the URG flag                                            |
| -X      | Sets the XMAS scan flags                                     |

## <u>Evasion</u>

- To evade IDS, sometimes you need to change the way you scan
- One method is to fragment packets (nmap -f switch)
- **OS Fingerprinting**
  - **Active**  - sending crafted packets to the target
  - **Passive** - sniffing network traffic for things such as TTL windows, DF flags and ToS fields
- **Spoofing** - can only be used when you don't expect a response back to your machine
- **Source routing** - specifies the path a packet should take on the network; most systems don't allow this anymore
- **IP Address Decoy** - sends packets from your IP as well as multiple other decoys to confuse the IDS/Firewall as to where the attack is really coming from. 
  - **`nmap -D RND:10 x.x.x.x`**
  - **`nmap -D decoyIP1,decoyIP2....,sourceIP,.... [target]`**
>  ⚡︎ **Check the IP Address Decoy [practical lab](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/2-Scanning-Networks/5-NmapDecoyIP.md) using nmap**
- **Proxy** - hides true identity by filtering through another computer.  Also can be used for other purposes such as content blocking evasion, etc.
  - **Proxy chains** - chaining multiple proxies together
    - Proxy Switcher
    - Proxy Workbench
    - ProxyChains
- **Tor** - a specific type of proxy that uses multiple hops to a destination; endpoints are peer computers
- **Anonymizers** - hides identity on HTTP traffic (port 80)

## <u>Banner Grabbing</u>

- **Active** - sending specially crafted packets and comparing responses to determine OS
- **Passive** - reading error messages, sniffing traffic or looking at page extensions
- Easy way to banner grab is connect via **telnet** on port (e.g. 80 for web server)
- **Netcat** can also be used to banner grab
  - `nc <IP address or FQDN> <port number>`
- Can be used to get information about OS or specific server info (such as web server, mail server, etc.)

* **Example of Banner grabbing on netcat - extracting request HTTP header**  
  1. `nc` command with `target IP` address and `port 80`
  2. issue the `GET / HTTP/1.0` (this GET request will send to the web server).
  3. The server responded with some interesting information.
```
nc 192.168.63.143 80
GET / HTTP/1.0            

HTTP/1.1 200 OK
Date: Sun, 12 Aug 2018 13:36:59 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2
X-Powered-By: PHP/5.2.4-2ubuntu5.10
Content-Length: 891
Connection: close
Content-Type: text/html

<html><head><title>Metasploitable2 - Linux</title></head><body>
<pre>

                _                  _       _ _        _     _      ____  
 _ __ ___   ___| |_ __ _ ___ _ __ | | ___ (_) |_ __ _| |__ | | ___|___ \ 
| '_ ` _ \ / _ \ __/ _` / __| '_ \| |/ _ \| | __/ _` | '_ \| |/ _ \ __) |
| | | | | |  __/ || (_| \__ \ |_) | | (_) | | || (_| | |_) | |  __// __/ 
|_| |_| |_|\___|\__\__,_|___/ .__/|_|\___/|_|\__\__,_|_.__/|_|\___|_____|
                            |_|                                          


Warning: Never expose this VM to an untrusted network!

Contact: msfdev[at]metasploit.com

Login with msfadmin/msfadmin to get started


</pre>
<ul>
<li><a href="/twiki/">TWiki</a></li>
```
  

## <u>Vulnerability Scanning</u>

- Can be complex or simple tools run against a target to determine vulnerabilities
- Industry standard is Tenable's **Nessus**

- Other options include
  - GFI LanGuard
  - Qualys
  - FreeScan - best known for testing websites and applications
  - OpenVAS - best competitor to Nessus and is free

# <u>Enumeration Concepts</u>
Enumeration is the process of extracting user names, machine names, network resources, shares, and services from a system, and its conducted in an intranet environment.

- Get user names using email IDs
- Get information using default passwords
- Get user names using SNMP
- Brute force AD
- Get user groups from Windows
- Get information using DNS zone transfers
- NetBios, LDAP, NTP

In this phase, the attacker creates an active connection to the system and performs directed queries to gain more information about the target. The gathered information is used to identify the vulnerabilities or weak points in system security and tries to exploit in the System gaining phase.

- **Defined as listing the items that are found within a specific target**
- **Always is active in nature**
- **Direct access**
- **Gain more information**

## <u>NetBIOS Enumeration</u>

- NetBIOS provides name servicing, connectionless communication and some Session layer stuff
- The browser service in Windows designed to host information about all machines within domain or TCP/IP network segment
- NetBIOS name is a **16-character ASCII string** used to identify devices

**Enumerating NetBIOS**:
- You can use **`zenmap or zenmap`** to check which OS the target is using, and which ports are open:
    - **`nmap -O <target>`**

- If theres any **UDP port 137** or **TCP port 138/139** open, we can assume that the target is running some type of NetBIOS service.

- On Windows is **`nbtstat`** command:

***`nbtstat`** displays protocol statistics and current TCP/IP connections using NetBIOS over TCP/IP.*

  - **`nbtstat`** gives your own info
  - **`nbtstat -a`** list the remote machine's name table given its **name**
  - **`nbtstat -A `** - list the remote machine's name table given its **IP address**
  - **`nbtstat -n`** gives local table
  - **`nbtstat -c`** gives cache information

![f](https://networkencyclopedia.com/wp-content/uploads/2019/08/nbtstat.jpg)

| Code | Type   | Meaning                   |
| ---- | ------ | ------------------------- |
| <1B> | UNIQUE | Domain master browser     |
| <1C> | UNIQUE | Domain controller         |
| <1D> | GROUP  | Master browser for subnet |
| <00> | UNIQUE | Hostname                  |
| <00> | GROUP  | Domain name               |
| <03> | UNIQUE | Service running on system |
| <20> | UNIQUE | Server service running    |

- NetBIOS name resolution doesn't work on IPv6
- **Other Tools**
  - SuperScan
  - Hyena
  - NetBIOS Enumerator (is a nbtstat with GUI)
  - NSAuditor

## <u>Windows System Basics</u>

- Everything runs within context of an account
- **Security Context** - user identity and authentication information
- **Security Identifier** (SID) - identifies a user, group or computer account
- **Resource Identifier** (RID) - portion of the SID identifying a specific user, group or computer
- The end  of the SID indicates the user number
  - Example SID:  S-1-5-21-3874928736-367528774-1298337465-**500**
  - **Administrator Account** - SID of 500
    - Command to get SID of local user: 
      - **`wmic useraccount where name='username' get sid`**
  - **Regular Accounts** - start with a SID of 1000
  - **Linux Systems** used user IDs (UID) and group IDs (GID).  Found in /etc/passwd
- **SAM Database** - file where all local passwords are stored (encrypted)
  - Stored in C:\Windows\System32\Config
- **Linux Enumeration Commands**
  - **`finger`** - info on user and host machine
  - **`rpcinfo`** and **`rpcclient`** - info on RPC in the environment
  - **`showmount`** - displays all shared directories on the machine

## <u>SNMP Enumeration</u>
> ⚡︎ **Check the SNMP Enumeration [practical lab](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/3-Enumeration/2-SNMP-Enumeration.md)**

SNMP enumeration is the process of enumerating the users accounts and devices on a SNMP enabled computer.

SNMP service comes with two passwords, which are used to configure and access the SNMP agent from the management station. They are: Read community string and Read/Write community string. These strings (passwords) come with a default value, which is same for all the systems. They become easy entry points for attackers if left unchanged by administrator.

Attackers enumerate SNMP to extract information about network resources such as hosts, routers, devices, shares(...) Network information such as ARP tables, routing tables, device specific information and traffic statistics.

- **Runs on Port 161 UDP**
- **Management Information Base** (MIB) - database that stores information
- **Object Identifiers** (OID) - identifiers for information stored in MIB
- **SNMP GET** - gets information about the system
- **SNMP SET** - sets information about the system
- **Types of objects**
  - **Scalar** - single object
  - **Tabular** - multiple related objects that can be grouped together
- SNMP uses community strings which function as passwords
- There is a read-only and a read-write version
- Default read-only string is **public** and default read-write is **private**
- These are sent in cleartext unless using SNMP v3
- **CLI Tools**
  - Metasploit module snmp_enum *(method used on the [practical lab](https://github.com/Samsar4/Ethical-Hacking-Labs/blob/master/3-Enumeration/2-SNMP-Enumeration.md))*
  - snmpwalk
- **GUI Tools**
  - Engineer's Toolset
  - SNMPScanner
  - OpUtils 5
  - SNScan

**Example of SNScan**:
  - *Note: the first scanned item is a printer running SNMP.*
<img width="80%" src="https://gist.githubusercontent.com/Samsar4/62886aac358c3d484a0ec17e8eb11266/raw/292c0411bb379aaf25e403741f64e62bbe4bc6a0/snmp-snas.png">


## <u>LDAP Enumeration</u>

- **Runs on TCP ports 389 and 636 (over SSL)**
- Connects on 389 to a Directory System Agent (DSA)
- Returns information such as valid user names, domain information, addresses, telephone numbers, system data, organization structure and other items

- To identify if the target system is using LDAP services you can use **nmap** with `-sT` flag for TCP connect/Full scan and `-O` flag for OS detection. 

**`sudo nmap -sT -O 192.168.63.142`**

```
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap <--------------------------------------
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl <--------------------------------------
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49159/tcp open  unknown
MAC Address: 00:00:11:33:77:44
Running: Microsoft Windows 2012
OS CPE: cpe:/o:microsoft:windows_server_2012:r2
OS details: Microsoft Windows Server 2012 or Windows Server 2012 R2
Network Distance: 1 hop
```
- **Tools for Enumeration LDAP:**
  - Softerra
  - JXplorer
  - Lex
  - LDAP Admin Tool

- **JXplorer example**:
<img width="80%" src="https://a.fsdn.com/con/app/proj/jxplorer/screenshots/16630.jpg/max/max/1">

## <u>NTP Enumeration</u>

- **Runs on UDP 123**
- Querying can give you list of systems connected to the server (name and IP)
- **Tools**
  - NTP Server Scanner
  - AtomSync
  - Can also use Nmap and Wireshark
- **Commands** include `ntptrace`, `ntpdate`, `ntpdc` and `ntpq`

**Nmap example for NTP enumeration:**
- `-sU` UDP scan
- `-pU` port UDP 123 (NTP)
- `-Pn` Treat all hosts as online -- skip host discovery
- `-n` Never do DNS resolution
- The [nmap script](https://nmap.org/nsedoc/scripts/ntp-monlist.html) `ntp-monlist` will run against the ntp service which only runs on UDP 123  

**`nmap -sU -pU:123 -Pn -n --script=ntp-monlist <target>`**

```
PORT    STATE SERVICE REASON
123/udp open  ntp     udp-response
| ntp-monlist:
|   Target is synchronised with 127.127.38.0 (reference clock)
|   Alternative Target Interfaces:
|       10.17.4.20
|   Private Servers (0)
|   Public Servers (0)
|   Private Peers (0)
|   Public Peers (0)
|   Private Clients (2)
|       10.20.8.69      169.254.138.63
|   Public Clients (597)
|       4.79.17.248     68.70.72.194    74.247.37.194   99.190.119.152
|       ...
|       12.10.160.20    68.80.36.133    75.1.39.42      108.7.58.118
|       68.56.205.98
|       2001:1400:0:0:0:0:0:1 2001:16d8:dd00:38:0:0:0:2
|       2002:db5a:bccd:1:21d:e0ff:feb7:b96f 2002:b6ef:81c4:0:0:1145:59c5:3682
|   Other Associations (1)
|_      127.0.0.1 seen 1949869 times. last tx was unicast v2 mode 7
```

- As you can see on the output above, information of all clients that is using NTP services on the network shown IPv4 and IPv6 addresses.

## <u>SMTP Enumeration</u>
- **Ports used**:
  - **SMTP: TCP 25** --> [outbound email]
  - **IMAP: TCP 143 / 993**(over SSL) --> [inbound email]
  - **POP3: TCP 110 / 995**(over SSL) --> [inbound email]
> - In simple words: **users typically use a program that uses SMTP for sending e-mail and either POP3 or IMAP for receiving e-mail.**

- **Enumerating with nmap**:
- `-p25` port 25 (SMTP)
- `--script smtp-commands` nmap script - attempts to use EHLO and HELP to gather the Extended commands supported by an SMTP server.

**`nmap -p25 --script smtp-commands <target IP>`**

```
PORT   STATE SERVICE
25/tcp open  smtp
| smtp-commands: WIN-J83C1DR5CV1.ceh.global Hello [10.10.10.10], TURN, SIZE 2097152, ETRN, PIPELINING, DSN, ENHANCEDSTATUSCODES, 8bitmime, BINARYMIME, CHUNKING, VRFY, OK, 
|_ This server supports the following commands: HELO EHLO STARTTLS RCPT DATA RSET MAIL QUIT HELP AUTH TURN ETRN BDAT VRFY 

Nmap done: 1 IP address (1 host up) scanned in 0.86 seconds
```

- It is possible to connect to SMTP through Telnet connection, instead using port 23(Telnet) we can set the port 25(SMTP) on the telnet command:
  - **`telnet <target> 25`**

- Case we got connected, we can use the **SMTP commands** to explore as shown below:
  - ![smtp](https://gist.githubusercontent.com/Samsar4/62886aac358c3d484a0ec17e8eb11266/raw/2ea0450933a1899b77a7519a46b4aee3b1597759/smtp-commands.png)

### Some SMTP Commands:

Command | Description
:--:|--
HELO | It’s the first SMTP command: is starts the conversation identifying the sender server and is generally followed by its domain name.
EHLO | An alternative command to start the conversation, underlying that the server is using the Extended SMTP protocol.
MAIL FROM | With this SMTP command the operations begin: the sender states the source email address in the “From” field and actually starts the email transfer.
RCPT TO | It identifies the recipient of the email
DATA | With the DATA command the email content begins to be transferred; it’s generally followed by a 354 reply code given by the server, giving the permission to start the actual transmission.
VRFY | The server is asked to verify whether a particular email address or username actually exists.
EXPN | asks for a confirmation about the identification of a mailing list.

**Other tools**:
- smtp-user-enum
  - Username guessing tool primarily for use against the default Solaris SMTP service. Can use either EXPN, VRFY or RCPT TO.
  