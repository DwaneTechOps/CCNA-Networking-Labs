# Lab 05 — Extended ACL Network Security

## Project Overview

This lab was executed using Cisco Packet Tracer as part of my hands-on CCNA networking and cybersecurity training.

The objective was to configure an extended Access Control List (ACL) to control communication between departments while maintaining authorized network connectivity.

The lab simulates a network security scenario at a fictional company, Skyfall Technologies.

## Business Scenario

Skyfall Technologies operates three departments: Sales, Human Resources (HR), and Information Technology (IT).

Management introduced the following network security requirements:

- Prevent Sales from accessing the HR network.
- Allow Sales to communicate with the IT network.
- Allow HR to communicate with the IT network.
- Ensure security restrictions do not disrupt normal network operations or authorized communication.

My responsibility as the Network Administrator was to implement these requirements using an extended ACL on the company's router.

## Network Topology

The network uses a Router-On-A-Stick configuration to enable inter-VLAN routing.

### VLAN Configuration

| Department | VLAN | Network | Default Gateway |
|---|---|---|---|
| Sales | 10 | 192.168.10.0/24 | 192.168.10.1 |
| HR | 20 | 192.168.20.0/24 | 192.168.20.1 |
| IT | 30 | 192.168.30.0/24 | 192.168.30.1 |

### Network Devices

- Cisco 2911 Router (R1)
- Cisco 2960 Switch (SW1)
- Three PCs representing Sales, HR, and IT

### Router Interfaces

| Interface | Department | IP Address |
|---|---|---|
| G0/1.10   | Sales      | 192.168.10.1 |
| G0/1.20   | HR         | 192.168.20.1 |
| G0/1.30   | IT         | 192.168.30.1 |

## Network Prerequisites

This lab builds upon the network infrastructure configured in Lab 4.

Before implementing the extended ACL, the following configurations were already completed:

- VLAN 10 — Sales (`192.168.10.0/24`)
- VLAN 20 — HR (`192.168.20.0/24`)
- VLAN 30 — IT (`192.168.30.0/24`)
- Access ports assigned to the appropriate VLANs.
- An 802.1Q trunk configured between SW1 and R1.
- Router-on-a-stick inter-VLAN routing configured on R1.
- Default gateways configured for all three departments.
- Successful connectivity tests between VLANs before applying the ACL.

### Existing Network Interfaces

| Device | Interface | Configuration |
|--------|-----------|---------------|
| SW1    | Fa0/1    | VLAN 10 — Sales|
| SW1    | Fa0/2    | VLAN 20 — HR |
| SW1    | Fa0/3    | VLAN 30 — IT |
| SW1    | Gi0/1    | Trunk connection to R1 |
| R1     | Gi0/1.10 | Sales gateway — 192.168.10.1 |
| R1     | Gi0/1.20 | HR gateway — 192.168.20.1 |
| R1     | Gi0/1.30 | IT gateway — 192.168.30.1 |

The purpose of Lab 5 was to introduce network security controls without rebuilding the existing network infrastructure.

## Extended ACL Configuration

### Step 1 — Access R1 and Enter Configuration Mode

```cisco
enable
configure terminal
```

### Step 2 — Create the Named Extended ACL

```cisco
ip access-list extended SALES_SECURITY
10 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
20 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
30 permit ip 192.168.10.0 0.0.0.255 any
exit
```

The ACL contains three rules:

| Rule | Action | Explanation |
|------|--------|-------------|
| 10   | Deny   | Blocks all IP traffic from Sales to HR |
| 20   | Permit | Allows Sales to communicate with IT |
| 30   | Permit | Allows Sales to communicate with other destinations not previously denied |

### Step 3 — Apply the ACL to the Sales Subinterface

```cisco
interface GigabitEthernet0/1.10
ip access-group SALES_SECURITY in
exit
```

The ACL is applied inbound on `G0/1.10`, the router subinterface serving VLAN 10 (Sales).

Extended ACLs are generally placed close to the source of traffic to prevent unauthorized packets from travelling unnecessarily through the network.

### Step 4 — Verify the ACL Rules

Return to privileged EXEC mode:

```cisco
end
```

Display the ACL:

```cisco
show ip access-lists SALES_SECURITY
```

This command confirms that the three ACL rules have been configured correctly.

### Step 5 — Verify the ACL Interface Assignment

```cisco
show ip interface GigabitEthernet0/1.10
```

The output should contain:

```text
Inbound access list is SALES_SECURITY
```

This confirms that the ACL is applied inbound on the correct router subinterface.

### Step 6 — Save the Router Configuration

```cisco
copy running-config startup-config
```

This saves the running configuration to the startup configuration so the settings persist after a router restart.

### ACL Commands

```cisco
enable
configure terminal

ip access-list extended SALES_SECURITY
10 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
20 permit ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
30 permit ip 192.168.10.0 0.0.0.255 any
exit
```

### Applying the ACL

The ACL was applied inbound on the router subinterface connected to the Sales VLAN.

```cisco
interface GigabitEthernet0/1.10
ip access-group SALES_SECURITY in
exit
```

### Why This Interface?

Extended ACLs are generally placed close to the source of the traffic.

Applying the ACL inbound on `G0/1.10` allows the router to filter unauthorized Sales department traffic before routing it toward other departments.

## Understanding the ACL Rules

| Rule | Action | Purpose |
|------|--------|---------|
| 10   | Deny   | Blocks all IP traffic from Sales to HR |
| 20   | Permit | Allows Sales to access IT |
| 30   | Permit | Allows Sales to access other destinations not previously denied |

Cisco evaluates ACL rules from top to bottom.

Once a packet matches a rule, the router performs the specified action without evaluating subsequent rules.

The ACL also has an implicit deny at the end, which blocks traffic that does not match a permit statement.

## Testing and Verification

After configuring the ACL, I tested network connectivity using ICMP ping commands.

### Connectivity Test Results

| Test               | Expected Result | Actual Result                | Status |
|--------------------|-----------------|------------------------------|--------|
| Sales → HR gateway | Blocked         | Destination host unreachable | PASS |
| Sales → IT gateway | Allowed         | Successful replies           | PASS |
| Sales → Sales gateway | Allowed      | Successful replies           | PASS |
| HR → IT PC            | Allowed      | Successful replies           | PASS |
| HR → Sales PC         | Ping fails   | Request timed out            | PASS |

### ACL Verification

I used the following Cisco command to verify the configured ACL and inspect packet matches:

```cisco
show ip access-lists SALES_SECURITY
```

The router recorded:

- Rule 10: Four denied packet matches.
- Rule 20: Four permitted packet matches.
- Rule 30: Four permitted packet matches.

These results confirmed that the ACL was processing traffic according to the configured rules.

## Security Analysis

### Why Could HR Not Successfully Ping Sales?

Although the ACL was applied to traffic entering the router from Sales, it also affected ICMP replies sent from Sales to HR.

When HR initiated a ping to Sales:

1. The ICMP Echo Request travelled from HR toward Sales.
2. The request entered R1 through the HR subinterface, `G0/1.20`.
3. The Sales PC received the request and generated an ICMP Echo Reply.
4. The reply entered R1 through the Sales subinterface, `G0/1.10`.
5. Rule 10 matched the reply because its source was Sales and its destination was HR.
6. R1 denied the reply, causing the ping from HR to time out.

This demonstrated an important limitation of traditional extended ACLs: they are stateless and do not automatically track established communication sessions.

### Security Lessons

- Extended ACLs can filter traffic using source and destination IP addresses.
- ACL placement determines which traffic is inspected.
- ACL rules are processed in sequence.
- Wildcard masks define which IP addresses match a rule.
- The `any` keyword represents all possible IP addresses.
- Stateless ACLs can block return traffic even when the original request was permitted.

## How to Open the Lab

### Requirements

- Cisco Packet Tracer

### Instructions

1. Download `Skyfall_Network_Security_Extended_ACL_Lab.pkt` from this folder.
2. Open Cisco Packet Tracer.
3. Select **File → Open**.
4. Locate and open the downloaded `.pkt` file.
5. Inspect the network topology and device configurations.
6. Open the PCs to repeat the connectivity tests.
7. Open R1's CLI to examine the ACL configuration and interface assignment.

### Useful Verification Commands

```cisco
show ip access-lists
show ip interface GigabitEthernet0/1.10
show running-config
show ip route
```

## Skills Developed

- Cisco Packet Tracer network simulation
- VLAN configuration and inter-VLAN routing
- Router-on-a-stick networking
- Extended ACL configuration
- Wildcard mask calculations
- Network traffic filtering
- ICMP connectivity testing
- ACL troubleshooting and verification
- Network security documentation

## Future Improvements

- Implement protocol-specific ACL rules for HTTP, HTTPS, SSH, and ICMP.
- Explore more granular access controls between departments.
- Compare stateless ACL filtering with stateful firewall policies.
- Expand the lab to include additional VLANs and security requirements.
