# Configs (black hole state)

Relevant lines only. Starting state before the fix in the [README](README.md).

## R1 (AS 100)
```
interface GigabitEthernet0/0
 ip address 1.1.1.1 255.255.255.0
interface GigabitEthernet0/1
 ip address 3.3.3.1 255.255.255.0
router bgp 100
 network 3.3.3.0 mask 255.255.255.0
 neighbor 1.1.1.2 remote-as 200
```

## R2 (AS 200, edge)
```
interface GigabitEthernet0/0
 ip address 1.1.1.2 255.255.255.0
interface GigabitEthernet0/1
 ip address 10.10.10.2 255.255.255.0
router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
router bgp 200
 neighbor 1.1.1.1 remote-as 100
 neighbor 11.11.11.4 remote-as 200
 neighbor 11.11.11.4 next-hop-self
```

## R3 (AS 200, transit, no BGP)
```
interface GigabitEthernet0/0
 ip address 10.10.10.3 255.255.255.0
interface GigabitEthernet0/1
 ip address 11.11.11.3 255.255.255.0
router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
 network 11.11.11.0 0.0.0.255 area 0
```

## R4 (AS 200, edge)
```
interface GigabitEthernet0/0
 ip address 11.11.11.4 255.255.255.0
interface GigabitEthernet0/1
 ip address 2.2.2.4 255.255.255.0
router ospf 1
 network 11.11.11.0 0.0.0.255 area 0
router bgp 200
 neighbor 2.2.2.5 remote-as 300
 neighbor 10.10.10.2 remote-as 200
 neighbor 10.10.10.2 next-hop-self
```

## R5 (AS 300)
```
interface GigabitEthernet0/0
 ip address 2.2.2.5 255.255.255.0
router bgp 300
 neighbor 2.2.2.4 remote-as 200
```

## Hosts
| Host | IP | Gateway |
|------|----|---------|
| SRV1 | 3.3.3.3/24 | 3.3.3.1 |
| SRV2 | 3.3.3.2/24 | none |

DSW1 is a plain L2 switch (default config).
