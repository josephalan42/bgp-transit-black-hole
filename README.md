# BGP Transit Black Hole

Shows how a non-BGP router inside a transit AS black-holes traffic, and how a full iBGP mesh fixes it.

**CCNP ENARSI (300-410):** 1.10 Troubleshoot BGP

## Topology

![Topology](topo.png)

| AS | Routers | Notes |
|----|---------|-------|
| 100 | R1, DSW1, SRV1, SRV2 | Originates 3.3.3.0/24 |
| 200 | R2, R3, R4 | Transit AS. OSPF area 0 on R2–R3–R4. R3 runs no BGP. |
| 300 | R5 | Receives 3.3.3.0/24, advertises 2.2.2.0/24 |

**Links:** R1–R2 1.1.1.0/24 · R2–R3 10.10.10.0/24 · R3–R4 11.11.11.0/24 · R4–R5 2.2.2.0/24

## Key config

```
R2 / R4 (AS 200)
 router bgp 200
  neighbor <other-edge> remote-as 200
  neighbor <other-edge> next-hop-self
```

eBGP links stay out of OSPF. `next-hop-self` keeps the next hop reachable.

## Steps

1. **Check BGP.** R5: `show ip bgp` shows 3.3.3.0/24 via AS 200.
2. **Find the gap.** R3: `show ip route 3.3.3.0` returns "not in table".
3. **See the black hole.** R5: `traceroute <SRV1> source 2.2.2.5` stops at R3.
4. **Fix it.** Make R3 an iBGP peer of R2 and R4 (full mesh):
   ```
   R3: router bgp 200
        neighbor 10.10.10.2 remote-as 200
        neighbor 11.11.11.<R4> remote-as 200
   R2/R4: neighbor <R3> remote-as 200
          neighbor <R3> next-hop-self
   ```
5. **Verify.** R3 shows a `B` route to 3.3.3.0/24. Traceroute from R5 reaches SRV1.

## Takeaway

The old **iBGP synchronization rule** stopped R4 from advertising a route the IGP didn't know, preventing this exact black hole. Modern IOS removed the `synchronization` command, so the fix is to run BGP on every transit router (full mesh or route reflectors).
