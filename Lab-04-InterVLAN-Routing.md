# Lab 4: Inter-VLAN Routing

## Objective
Configure VLANs and a trunk, then use router-on-a-stick so devices in Sales, HR, and IT can communicate across subnets.

## VLANs

| Department | VLAN | Network | Gateway |
|---|---:|---|---|
| Sales | 10 | 192.168.10.0/24 | 192.168.10.1 |
| HR | 20 | 192.168.20.0/24 | 192.168.20.1 |
| IT | 30 | 192.168.30.0/24 | 192.168.30.1 |

## Configuration summary
- Created VLANs 10, 20, and 30 on the switch and assigned the department ports.
- Configured the switch-to-router link as an 802.1Q trunk.
- Configured router subinterfaces on G0/1 as the default gateways for each VLAN.

## Verification
Connectivity between devices in the Sales, HR, and IT VLANs was tested successfully.
