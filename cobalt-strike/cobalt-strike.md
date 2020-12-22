# Cobalt Strike Notes

**_Table of Contents_**:
1) Operations
2) Infrastructure
3) C2
4) Weponization
5) Initial Access
6) Post Exploitation
7) Priviledge Escalation
8) Lateral Movement
9) Pivoting


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

-------------

## 3) C2 (Command and Control)

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