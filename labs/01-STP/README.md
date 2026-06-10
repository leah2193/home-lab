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


## Hardware Evidence

![STP Port Blocking](STP-port-blocking.jpg)

This screenshot shows STP placing a redundant link into the blocking state to prevent a Layer 2 loop.

## Root Bridge Election

![STP Root Bridge](STP-changed-rootbridge.jpg)

This screenshot shows SW1 becoming the root bridge after modifying bridge priority.

## STP Recalculates After Root Bridge Change

![STP Blocking Port on SW2](STP-now-blocking-port-SW2.jpg)

After changing the bridge priority and making SW1 the root bridge, STP recalculated the topology. The blocking port moved to SW2, demonstrating how STP dynamically selects the optimal forwarding and blocking paths while preventing Layer 2 loops.
