[SRWE_COMMANDS_ONLY.md](https://github.com/user-attachments/files/28180143/SRWE_COMMANDS_ONLY.md)
## CoreRouter — Password Recovery
```
rommon 1> confreg 0x2142
rommon 2> reset
copy tftp running-config
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

## CoreRouter — AAA
```
aaa new-model
tacacs-server host 192.168.5.2
tacacs-server key aaa-secret
aaa authentication login srwe-exam group tacacs+ local
line console 0
login authentication srwe-exam
```

## Rtr1 — HSRP
```
interface gig0/2
standby 5 ip 192.168.15.1
standby 5 priority 105
standby 5 preempt
standby 5 track gig0/0
```

## Rtr2 — HSRP
```
interface gig0/2
standby 5 ip 192.168.15.1
```

## Rtr1 + Rtr2 — DHCP
```
ip dhcp excluded-address 192.168.15.1 192.168.15.10
ip dhcp excluded-address 192.168.15.254
ip dhcp pool DR_DHCP_POOL
network 192.168.15.0 255.255.255.0
default-router 192.168.15.1
```

## CoreRouter — ACL BLOCK (Standard Named)
```
ip access-list standard BLOCK
permit host 192.168.15.11
permit host 192.168.15.12
deny any
interface gig0/2
ip access-group BLOCK out
```

## Rtr1 — ACL 110 (Extended Numbered)
```
access-list 110 permit tcp any host 192.168.25.2 eq 23
access-list 110 permit tcp any host 192.168.25.2 eq 80
access-list 110 permit icmp any host 192.168.25.2
access-list 110 deny ip any any
interface gig0/1
ip access-group 110 out
```

## CoreRouter — Static NAT
```
ip nat inside source static 192.168.15.11 209.165.44.6
interface s0/0/0
ip nat outside
interface gig0/0
ip nat inside
```

## CoreRouter — PAT
```
access-list 10 permit 192.168.25.0 0.0.0.255
ip nat inside source list 10 interface s0/0/0 overload
interface s0/0/0
ip nat outside
interface gig0/0
ip nat inside
```
