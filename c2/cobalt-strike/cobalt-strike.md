# Cobalt Strike Notes

Things to Look more into...
* Metasploit staging framework

**_Table of Contents_**:
1) [Operations](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#1-operations)
2) [Infrastructure](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#2-infrastructure)
3) [C2](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#3-c2-command-and-control)
4) [Weponization](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#4-weponization)
5) [Initial Access](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#5-initial-access)
6) [Post Exploitation](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#6-post-exploitation)
7) [Priviledge Escalation](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#7-priviledge-escalation)
8) [Lateral Movement](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#8-lateral-movement)
9) [Pivoting](https://github.com/ezra-buckingham/tool-notes/blob/main/cobalt-strike/cobalt-strike.md#9-pivoting)


**_Overview_**:

This tool is not a magic wand to every scenario and remember that you must always be considering evasion tactics (you MUST know your tools and the best option to evade) as there are so many different monitoring solutions as well as teams on networks that defend against APT groups. 

![Attack Chain](./assets/AttackChain.png "Attack Chain")

**_Architecture for Cobalt Strike_**:

Has a team server (runs on linux) and a client tool. All the client tools can connect to the team server and that team server will contain all tools needed.

**_Reason for Cobalt Strike_**:

Create relevant and credible adversary simulations that:
* Create battle hardened security analysts that can approach any attack
* Drive objective and meaningful security advances
* Educate security professionals and decision makers on advanced threat tactics

**_Basic Usage_**

Starting the team server: `./teamserver <ip address of machine> <password>`

Starting the client: `./cobaltstrike`

Notes:
* The IP address is where you will be able to access the team server

**_Tools Within Cobalt Strike_**:
* **Beacon**
  * Cobalt Strike's Payload
  * Two comunication strategies
    * Async (low and slow)
    * Interactive (real time control)
  * Uses HTTP/S or DNS to egress a network
  * Uses SMB or TCP for p2p c2
  * Remote admin tool features

* **Malleable C2**
  * A domain-specific language to give you control over the indicators in the Beacon payload
    * Network traffic
    * In-memory content, characteristics, and behavior
    * Process injection behavior

* **Aggressor Script**
  * Scripting language built into CS that allows you to modify and extend the Cobalt Strike client

* **Logging**
  * Everything gets logged in on the team server (see `logs/` folder)

* **Reporting**
  * You can generate reports that help train blue team
    * Examples: Indicators of Compromise, Activity Report, Sessions Report, & Tactics/Techniques/Procedures Report
  * Found in the "Reporting" menu
  * You can even merge data from multiple team servers (with clock correction)
-------------

## 1) Operations

### Distributed Operations
There is a concept of "distributed operations" in which there are multiple team servers running that have different objectives and none of them know about each other. 

_Why would we do this?_ 

You do not want a single point of failure during operations.

_What does this mean?_
* You can connect to multiple team servers at once
* You can rename each server so that you know exactly what server does what
* Once connecting to a new server, you get a toolbar at the very bottom allowing you to toggle between servers quickly

_Best Practices for Infrastructure_
* Staging Servers
  * Host client-side attacks and inital callbacks
  * Inital priviledge escalation
  * Expect these servers to be caught... _quickly_
* Post-Exploitation Servers
  * Post-exploitation and lateral movement
* Long Haul Servers
  * "Low and Slow" persistent callbacks
  * Pass access to post-exploitation as needed
  * This is your lifeline should your other servers get caught

![Distributed Operations](./assets/DistributedOps.png "Dist Ops")

### Scaling Red Operations

The best way to scale operations is to split into Target cells and Access Management Cells. You do this because it is important to manage all your access in one spot reliably.

**Potential Team cells to organize your team for Scalability**

_Target Cells_
* Responsible for objectives on specific networks
* Gain access, post-exploitation, and spread latterally
* MAintain local infrastructure these tasks

_Access Management Cell_
* Holds accesses for all networks
* Gain access and recieve access from cells
* Pass accesses to target cells as needed
* Maintain global infrastructure for persistent cells

**Potential Team Roles for Scalability**

_Access_
* Get in and expand foothold

_Post Exploitation_
* Data mining, monitor users, key log, etc 

_Local Access Manager (Shell Sherpa)_
* Manage callbakcs
* Setup infra
* Persistence
* Pass sessions to and from global access manager

-------------

## 2) Infrastructure

### Listener Management
What is a listener?
* A configuration for a cobalt strike payload (and payload server)
* Different types of listeners
  * Egress - a payload the beacons out of a network
  * Peer-to-peer - a payload that communicates through a parent payload
  * Alias - a reference to a payload handler elsewhere (eg in another toolset)
* Manage using Cobalt Srike > Listeners

### Beacon Payloads
_Payload Staging_
* What is stager?
  * A tiny program that downloads a payload and passes execution to it (needed for size-constrained attacks)
  * Based on Metasploit framework staging protocol 
* Stageless means payload without a stager
* Stagers are (most of newer versions of Cobalt rely less on these):
  * Less secure
  * More brittle
  * Easier to detect

_HTTP/S Beacon_
* A Payload contained on the target machine that makes a GET request to the C2 server to get instructions for the payload to execute (in an enxrypted blob of data)
* The get request, if containing instructions for the HTTP beacon, will after completion of the tasks, will wespond with a POST request to the C2 server with the output (in an encrypted blob of data) from the tasks
* When configuring one, you can give a comma seperated list of callback hosts to callback many different hosts
  * Can give IPv4, IPv6, or a FQDN
  * Can also give a proxy config for the beacon
* Remember that only a single service can hold a single port so you cannot have a beacon on port 80 and a web server running at the same time

_DNS Beacon_
* Uses a DNS lookup to send back instructions to the beacon on tasks to complete
* This was used as a way to minimize the number of HTTP requests that are made to the C2 server (if no task was required, the beacon response from the A record DNS request would be 0.0.0.0, but if there were required tasks, the server would respond with an IP address to get from)
* Sometimes this inital A record request will work BUT then the HTTP request following will be blocked. You can get around this, by using the `mode [dns | dns6 | dns-txt]` in CS and retreiving tasks in smaller byte chunks at a time by making a bunch of A record requests and gaining access to the payload again

_DNS C2 Details_
* The CS DNS TXT Stager will require 1,000+ TXT records to download the Beacon
* If the host can resolve names, then you can get out
* The DNS Beacon payload has optinos for DNS A, DNS AAAA, and DNS TXT for C2 and keep in mind that each mode reduces maximum hostname length used to send data back to CS

_DNS Beacon Set Up_
1. Edit Zone File for a domain you control
2. Create an A record for CS server
3. Create an NS record that points to FQDN of the CS server

![DNS](./assets/DNSBeacon.png "DNS")

_SMB Beacon_
* This beacon utilizes the named pipes funcationality for windows
* It uses a child beacon and a parent beacon (an egress beacon and peer-to-peer beacons)
* It uses the port 445 SMB protocol to communcate peer-to-peer
* You can see the lineage of beacons by going to Cobalt Strike > Visualization > Pivot Graph

_SMB Beacon Usage_
* You can link and unlink to a SMB beacon to keep your payload on the machine, but dormant
  * Link Beacon to peer `link [host] [pipe]`
  * De-link Beacon to peer `unlink [host] [pid]`
* You can also create a TCP p2p beacon and do the same as the SMB p2p beacon
  * Link Beacon to peer `connect [host] [port]`
  * De-link Beacon to peer ``unlink [host] [pid]`

![SMB](./assets/SMBBeacon.png "SMB")

_How to deliver the Beacon_
* It can deliver via a scripted web delivery in which Cobalt Strike will deliver a one-line script that will allow you pull the beacon over the web to the target

_Redirectors_
* You may use a redirector to obfuscate exactly where your team server is located
* You can do this by using iptables, socat, or other tool to forward traffic to your CS team server
  `socat TCP4-LISTEN:80,fork TCP4:[teamserver]:80`
* You can use Apache ot Nginx reverse proxy config
* You can use a CDN (ex. AWS Cloud Front) as a redirector for HTTPS traffic (called "domain fronting")
  * You will need to use a valid SSL certificate
  * Allow the HTTP POST and HTTP GET vers
  * Consider HTTP-GET only C2
  * Disable all cache options
  * Be aware of transformed requests! (changing cookie value)
  * Also this allows us to make the server look fully legit because a CDN is used by hundreds of legitimate companies
  * You can make the beacon override the requested host from "google.com" to "malicious.com"
    * Some proxy servers will normalize the data and not allow domain fronting to work as well as it used to
    * SSL will change this because it goes in an encrypted format and the proxy may not always be able to see that (only if the organization does man in the middle communication with the proxy will they catch the domain forwarding)

-------------

## 3) C2 (Command and Control)

### Malleable C2

_Overview_
* Domain-specific language to give you control over the indicators in the beacon payload
  * Network traffic
  * In-memory content, characteristics, and behavior
  * Process injection behavior
* This isbound to the server when staring the team server
* You can set this profile up to obfuscate the data when sending across the network
* You can set the profile to chunk the output so that it is less suspicious (HTTP Get-Only C2)
  * This can be done by doing `set verb "GET";` in your `http-post` option (this is not the actual option to set for chunking)
  * Setting to HTTP GET only will make it not very performant compared to POST
* You can also set profile variants that have slightly different variations in the profile (this is set when you create a listener) and is done by using the `<block> "variant name" {}`
* Always lint your profile using the `./c2lint <c2ProfilePath>` tool
  * It can give you some powerful linting to show you what errors you may have if using that profile in with a beacon

_Profile Components_
* Options
  * the `set` keyword will set the http option and the option will always be wrapped in double-quotes
  * the `set` keyword can also be placed into a type of request such as `http-get` like the example below
* Blocks
  * A way of grouping indicators by a specific method/behavior
  * You can also change the behavior based on if it is a request coming from the server or client

```
http-get {
  #indicators here
  # How beacon downloads tasks
  client { }
  server { }
}
http-post {
  #indicators here
  # How beacon uploads output back to team server
}
http-stager {
  #indicators here
  # Shapes content of the staging process
}
```

* Extraneous indicators
  * Think of these as the HTTP headers that you can set as extraneous
* Transforms
  * These fall under the metadata and tell the client beacon to transform the data (and they are done in order)
  * Think of these as functions that will be executed by the beacon before sending

_Example of Profile_

```
set useragent "Mozilla/5.0 (compatible; MISE 8.0;)"

http-get {
  set uri "/image/";
  client {
    header "Accept" "text/html,application/xhtml";
    header "Referer "htpp://www.google.com";
    header "Pragma" "no-cache";
    header "Cache-Control" "no-cache";
    metadata {
      netbios;
      append "-.jpg";
      uri-append;
    }
  }
}
```

![c2 Key Blocks](./assets/MalleableC2.png "c2 Key Blocks")

### Egress & Network Evasion

_The Problems We Encounter_
* Some firewalls may deny all outbound traffic
* Some networks only allow egress through a proxy device
  * This means that attack traffic must conform to an expected protocol as well as other checks
* Additionally, there may be network monitoring that we must evade
  * There are systems that run looking for IOC's or Indicators of Compromise on the network
  * The network may alerady have Cobalt Strike ID'd as an IOC 

_HTTP/S Proxy Details in Cobalt Strike_
* This is all built on the WinINet (Internet Explorer is built on this)
* It will use the IE proxy settings for the current user
  * If IE can get out, you can get out unless the current user has no proxy config
  * The stagers and payloads in CS will prompt the user if proxy authentication fails
* You can use custom proxy settings if necesary
  * This can be risky because it shows up in the HTTP traffic and if there are creds in that payload, it can get exposed and leaked (risk/reward)
* **Note**: WinINet library may have TLS limitations that are incompatible with your redirector

_Profile Tips_
* Do not use public profiles examples in production
  * This will make it easier to get caught
* Do not allow an empty http-get > server > output > or http-post > server > output response
  * Use `prepend` to prepend junk data and `mask` to transform and randomize it
  * This is important because we need it to look like legitimate traffic (think of a get request with content length of 0)
* Change URIs and use `prepend` to mask indications of comproimise in `http-stager` block (if you are going to allow staging)
  * Because you will be staging a payload, you NEED to try and mask that URI so it doesn't look suspicious
  * Avoid using content type of `application/octet-stream` because that is not very normal/common
* Use `http-config` block to standardize server headers and header order in all HTTP server responses
* Use plausible `set useragent` value for the target network
* Use HTTP GET-nly C2 for difficult egress situations
  * Consider using it when using stagers in an unkown network because that will cause the least friction

_Network Security Monitoring Tips_
* Use an Apache, Nginx, or a CDN as a redirector 
  * This will smooth the Cobalt Strike-specific indicators, give you a better JA3S fingerprint, etc
  * JA3 (add the S if talking about the server) is a way at looking at the handshake process of TLS traffic and generating a hash based on the alogrithms used on both the client and server
    * Our team server responds with a JA3S figerprint saying that the server is runnning Java on Debian
    * However, using a CDN will show a JA3S fingerprint saying that the server is running Apache or something more normal
* Invest in your infrastructure
  * Host redirectors on different providers
  * Domains are better with age and categorization
    * Newer & Uncategorized domains are sus
  * No not use IPv4 addresses for C2
  * Have a valid SSL Cert (DO NOT Use the CS default cert)
* Operate "low and slow"
  * Set higher beacon sleep intervals
  * Find the interval that allows you just enough access to accomplish your objectives

_Common DNS C2 Prevention and Tips for Overcoming those Tactics_
* Use Split-Split DNS
  * Having your proxy do the DNS lookup if needed for servers outside of network (client unable to query against DNS)
  * **Solution:** Do not use DNS C2
* Look at the volume of DNS requests
  * Seeing a massive amount of DNS traffic would look sketch
  * **Solution:** Use DNS C2 as low and slow fallback option ONLY
* Look for Cobalt Strike DNS C2 IOCs
  * **Solution:** Use `dns_stager_prepend` and `dns_stager_subhost`
* Look for Bogon IP addresses
  * For example, seeing `0.0.0.0` would not make sense so this could look suspicious
  * **Solution:** Change `dns_idle` in profile and avoid `mode dns` as this will send bogon responses
* Look for length of request hostnames
  * **Solution:** Set `dns_max_txt` to limit TXT length and set `maxdns` to limit hostname length

_Tips for P2P Beacons_
* Use the SMB Beacon and TCP Beacon for...
  * Processes that run as SYSTEM
  * Targets that cannot reach the internet
* Use the TCP Beacon bound to localhost for local-host only priviledge escalation actions

_Tips for Communication Paths_
* Don't
  * Servers should not egress
  * SMB Beacon from server to workstation is WEIRD
  * Scan to find targets (creates lots of noise on the wire)
* Do
  * Egress from plausible user processes on workstations
    * Think of all the processes that WinInet runs (see image below)
  * Use SMB Beacon from workstations to servers
  * Use built-in Windows tools and APIs to find targets

![Plausible Processes](./assets/PlausibleProcesses.png "Plausible Processes")

_Infrastructure OPSEC (Finding other CS Servers & How to hide yours)_
* How to find them on the internet?
  * The Default (self-signed) SSL Cert
    * **Solution:** Use valid SSL cert; Use Apache, Nginx, or CDN; Only alow HTTP/S connections from redirectors
  * `0.0.0.0` DNS response
    * **Solution:** Set `dns_idle` in Malleable C@
  * Port 50050 open
    * **Solution:** Firewall port 50050 and access via SSH
  * Empty index page, 404, `text/plain` Content-Type
    * **Solution:** Host content on your redirectors
* How to verify team server?
  * Connect to it and ask foor a payload (staging)
  * Eg) `wget -U "Internet Explorer" http://<server>/vI6D`
  * **Solution:** Set `host_stage` to false in Malleable C2 (this disables hosted payload for staging purposes)

_Payload Security Features_
* Beacon payload authenticates the team server
* Beacon tasks and output are encrypted
* Beacon has replay protection for tasks
* Payload stagers DO NOT have security features

![Payload Sec](./assets/PayloadSec.png "Payload Sec")

-------------

## 4) Weponization

-------------

## 5) Initial Access

-------------

## 6) Post Exploitation

-------------

## 7) Priviledge Escalation

-------------

## 8) Lateral Movement

-------------

## 9) Pivoting