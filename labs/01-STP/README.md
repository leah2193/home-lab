## Topology

![Lab Topology](topology.jpeg)

## IP Addressing

| Device | Interface | IP Address | Role |
|--------|-----------|------------|------|
| SW1 | VLAN 1 | 10.0.1.1/24 | Distribution switch |
| SW2 | VLAN 1 | 10.0.1.2/24 | Access switch |
| R1 | Gi0/0 | 10.0.1.254/24 | Default gateway |
| R1 | Gi0/1 | 10.0.2.1/30 | R1-R2 link |
| R2 | Gi0/0 | 10.0.1.253/24 | Secondary router |
| R2 | Gi0/1 | 10.0.2.2/30 | R1-R2 link |
| Laptop | ETH | 10.0.1.100/24 | Management workstation |

## What Was Observed

### Before Priority Manipulation
- SW2 elected root bridge — lowest MAC address won
- SW1 Fa1/0/4 placed in blocking state (amber port light)
- SW1 Fa1/0/3 elected root port — best path to SW2

### After Priority Manipulation
- SW1 priority set to 4096 via:
  spanning-tree vlan 1 priority 4096
- SW1 became root bridge
- STP reconverged — different port now blocking
- Confirmed with show spanning-tree

## Hardware Evidence

![STP Port Blocking](STP-port-blocking.jpg)

This screenshot shows STP placing a redundant link into the blocking state to prevent a Layer 2 loop.

## Root Bridge Election

![STP Root Bridge](STP-changed-rootbridge.jpg)

This screenshot shows SW1 becoming the root bridge after modifying bridge priority.

## STP Recalculates After Root Bridge Change

![STP Blocking Port on SW2](STP-now-blocking-port-SW2.jpg)

After changing the bridge priority and making SW1 the root bridge, STP recalculated the topology. The blocking port moved to SW2, demonstrating how STP dynamically selects the optimal forwarding and blocking paths while preventing Layer 2 loops.

### Issue 1 — Cannot Ping 10.0.2.0/30 From Laptop
**Symptom:** Pings to the R1-R2 link subnet `10.0.2.0/30` 
timed out from the management laptop despite correct 
device configurations.

**Not straightfoward** The directly connected 
management subnet `10.0.1.0/24` was fully reachable — 
all four devices (SW1, SW2, R1, R2) responded to pings 
normally. This initially suggested the network was 
healthy and pointed suspicion toward routing or 
firewall issues on the devices rather than the 
laptop itself.

**Initial Investigation:** Windows Firewall was the first 
suspect since the symptom resembled inbound ICMP being 
blocked. A firewall rule was added to allow inbound 
ICMPv4 using the following command run as Administrator:

    netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow

Pings still failed after the rule was added, ruling out 
the firewall as the cause and shifting investigation 
toward the routing table.

**Tools Used:** `ping`, `tracert`, `route print`, `ipconfig`

**Diagnosis:** `tracert 10.0.2.1` revealed traffic was 
exiting via the home WiFi interface to Spectrum rather 
than the lab ethernet adapter. `route print` confirmed 
two competing default routes — WiFi adapter metric (35) 
was lower than the lab ethernet adapter metric (291), 
so Windows preferred WiFi for all traffic including 
lab destinations.

**Root Cause:** Competing default routes on Windows 
laptop. The `10.0.1.0/24` subnet worked because it 
was directly connected on the lab adapter — Windows 
didn't need the default route for that traffic. 
The moment traffic needed to cross to `10.0.2.0/30` 
via a gateway, Windows chose the WiFi default route 
instead of the lab adapter, sending packets out to 
the internet instead of to R1.

**Fix:** Added a specific static route for the lab subnet: 
route add -p 10.0.2.0 mask 255.255.255.252 10.0.1.254

This directed traffic destined for `10.0.2.0/30` 
specifically through the lab gateway while leaving 
internet traffic unaffected.

**Lesson:** Always verify the routing table on the 
management workstation before troubleshooting network 
devices. A directly connected subnet bypasses the 
default route entirely — which is why `10.0.1.0/24` 
worked fine while `10.0.2.0/30` failed. The problem 
was never on the network — it was on the host.
