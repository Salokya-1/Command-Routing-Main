[SRWE_EXAM_CONFIG.md](https://github.com/user-attachments/files/28179966/SRWE_EXAM_CONFIG.md)
# Command-Routing-Main# SRWE Exam — Full Configuration Guide

## Network Topology Summary

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| CoreRouter | S0/0/0 | 209.165.44.2 | 255.255.255.248 |
| CoreRouter | Gig0/0 | 192.168.10.1 | 255.255.255.0 |
| CoreRouter | Gig0/1 | 192.168.20.1 | 255.255.255.0 |
| CoreRouter | Gig0/2 | 192.168.5.1 | 255.255.255.0 |
| Rtr1 | Gig0/0 | 192.168.10.2 | 255.255.255.0 |
| Rtr1 | Gig0/1 | 192.168.25.1 | 255.255.255.0 |
| Rtr1 | Gig0/2 | 192.168.15.2 | 255.255.255.0 |
| Rtr2 | Gig0/0 | 192.168.20.2 | 255.255.255.0 |
| Rtr2 | Gig0/2 | 192.168.15.3 | 255.255.255.0 |
| TFTP & TACACS+ Server | Fa0 | 192.168.5.2 | 255.255.255.0 |
| Web Server | Fa0 | 192.168.25.2 | 255.255.255.0 |
| File Server | Fa0 | 192.168.25.3 | 255.255.255.0 |
| Email Server | Fa0 | 192.168.25.4 | 255.255.255.0 |

---

## a) Password Recovery — CoreRouter

> Enter ROMmon by pressing CTRL+C or BREAK during boot

```
rommon 1> confreg 0x2142
rommon 2> reset

copy startup-config running-config

enable secret cisco

interface gig0/0
no shutdown
interface gig0/1
no shutdown
interface gig0/2
no shutdown

config-register 0x2102
copy running-config tftp
192.168.5.2
```

---

## b) AAA Authentication — CoreRouter

### TACACS+ Server GUI Settings

**Network Configuration Tab:**

| Field | Value |
|-------|-------|
| Client Name | CoreRouter |
| Client IP | 192.168.5.1 |
| Secret | aaa-secret |
| Server Type | TACACS |

Click **Add**

**User Setup Tab:**

| Field | Value |
|-------|-------|
| Username | admin |
| Password | admin-secret |

Click **Add** — Make sure **Service is ON**

### CoreRouter Commands

```
aaa new-model
tacacs-server host 192.168.5.2
tacacs-server key aaa-secret

aaa authentication login srwe-exam group tacacs+ local

line console 0
login authentication srwe-exam
```

---

## c) HSRP — Rtr1 and Rtr2

### Rtr1

```
interface gig0/2
standby 5 ip 192.168.15.1
standby 5 priority 105
standby 5 preempt
standby 5 track gig0/0
```

### Rtr2

```
interface gig0/2
standby 5 ip 192.168.15.1
```

---

## d) DHCP Server — Rtr1 and Rtr2

> Apply same config on BOTH routers

```
ip dhcp excluded-address 192.168.15.1 192.168.15.10
ip dhcp excluded-address 192.168.15.254

ip dhcp pool DR_DHCP_POOL
network 192.168.15.0 255.255.255.0
default-router 192.168.15.1
```

---

## e) DHCP Clients

> On PC1, PC2, PC3 — set IP Configuration to **DHCP** in Packet Tracer GUI
> Obtain serially: PC1 = 192.168.15.11, PC2 = 192.168.15.12, PC3 = 192.168.15.13

---

## f) Wireless Router Configuration

> Configure via GUI (WRT300N) — Config Tab

**Internet Tab:**
- Connection Type: `Automatic Configuration - DHCP`

**LAN / Network Tab:**

| Field | Value |
|-------|-------|
| Router IP | 192.168.200.1 |
| Subnet Mask | 255.255.255.0 |
| DHCP Server | Enabled |
| Start IP | 192.168.200.10 |
| Max Users | 75 |

**Wireless Tab:**

| Field | Value |
|-------|-------|
| Network Mode | Mixed |
| SSID | cisco_router |
| Channel | 6 |
| SSID Broadcast | Enabled |

**Wireless Security Tab:**

| Field | Value |
|-------|-------|
| Security Mode | WPA2 Personal |
| Encryption | TKIP |
| Passphrase | cisco@123 |

> Connect Laptop1 and Laptop2 via wireless, set to DHCP

---

## g) Named Standard ACL — BLOCK (CoreRouter)

> Permit only PC1 and PC2 to reach TFTP/TACACS+ Server (192.168.5.2)
> Standard ACL — apply near destination (CoreRouter Gig0/2 out)

```
ip access-list standard BLOCK
permit host 192.168.15.11
permit host 192.168.15.12
deny any

interface gig0/2
ip access-group BLOCK out
```

---

## h) Numbered Extended ACL 110 — Rtr1

> Permit telnet(23), web(80), ping(icmp) to Web Server (192.168.25.2) from any
> Deny all other traffic to Web Server
> Apply on Rtr1 Gig0/1 out (toward server LAN)

```
access-list 110 permit tcp any host 192.168.25.2 eq 23
access-list 110 permit tcp any host 192.168.25.2 eq 80
access-list 110 permit icmp any host 192.168.25.2
access-list 110 deny ip any any

interface gig0/1
ip access-group 110 out
```

---

## i) Static NAT — CoreRouter

> PC1 (192.168.15.11) → Inside Global (209.165.44.6)

```
ip nat inside source static 192.168.15.11 209.165.44.6

interface s0/0/0
ip nat outside

interface gig0/0
ip nat inside
```

---

## j) PAT for Server LAN — CoreRouter

> Server LAN: 192.168.25.0/24 (Web, File, Email Servers)
> Interesting traffic: ACL 10
> Outside interface: S0/0/0

```
access-list 10 permit 192.168.25.0 0.0.0.255

ip nat inside source list 10 interface s0/0/0 overload

interface s0/0/0
ip nat outside

interface gig0/0
ip nat inside
```

---

## Quick Reference — Which Router Does What

| Task | Router |
|------|--------|
| Password Recovery | CoreRouter |
| AAA / TACACS+ | CoreRouter |
| HSRP Active (Priority 105) | Rtr1 — Gig0/2 |
| HSRP Standby | Rtr2 — Gig0/2 |
| DHCP Server | Rtr1 + Rtr2 |
| ACL BLOCK (Standard Named) | CoreRouter — Gig0/2 out |
| ACL 110 (Extended Numbered) | Rtr1 — Gig0/1 out |
| Static NAT (PC1) | CoreRouter |
| PAT (Server LAN) | CoreRouter |

---

## ACL Direction Rules

| ACL Type | Place Near | Direction |
|----------|-----------|-----------|
| Standard | Destination | out |
| Extended | Source | in |
