# Enterprise HSRP Network Redundancy

![Cisco Packet Tracer](https://img.shields.io/badge/Network-Cisco%20Packet%20Tracer-blue)
![HSRP](https://img.shields.io/badge/Protocol-HSRP-green)
![Networking](https://img.shields.io/badge/Domain-Network%20Engineering-orange)
![CCNA](https://img.shields.io/badge/CCNA-Practice-red)
![Status](https://img.shields.io/badge/Project-Lab%20Implementation-brightgreen)

## 📌 Project Overview

**Enterprise HSRP Network Redundancy** is a Cisco Packet Tracer networking project that demonstrates gateway redundancy using the Hot Standby Router Protocol (HSRP).

In an enterprise network, the failure of a default gateway can interrupt connectivity for connected users. HSRP addresses this problem by allowing multiple routers to share a virtual IP address that hosts use as their default gateway.

One router operates as the active gateway, while another remains on standby. If the active router becomes unavailable, the standby router can take over the gateway role.

This project demonstrates router configuration, HSRP election, connectivity verification, failure simulation, and gateway recovery.

## 🎯 Project Objectives

- Understand the purpose and operation of HSRP.
- Configure two routers for first-hop redundancy.
- Create a shared virtual gateway.
- Configure router interface IP addresses.
- Set HSRP priorities and preemption.
- Verify active and standby router roles.
- Simulate router failure and observe failover.
- Restore the failed router and verify recovery.
- Develop practical Cisco IOS troubleshooting skills.

## 🏗️ Network Topology

The lab uses two Cisco 2911 routers, two Cisco 2960 switches, and four PCs.

```text
                  +------------------+
                  |       R1         |
                  | Primary Router   |
                  | 192.168.10.2     |
                  | Priority: 110    |
                  +---------+--------+
                            |
                            |
                  +---------+--------+
                  |   Shared LAN      |
                  | 192.168.10.0/24   |
                  | Virtual IP:       |
                  | 192.168.10.1      |
                  +---------+--------+
                            |
                +-----------+-----------+
                |                       |
           +----+----+             +----+----+
           |   SW1   |-------------|   SW2   |
           +----+----+             +----+----+
                |                       |
             PC1, PC2                 PC3, PC4
                |
                  +------------------+
                  |       R2         |
                  | Standby Router   |
                  | 192.168.10.3     |
                  | Priority: 100    |
                  +------------------+
```

The diagram is a logical representation. In the actual Packet Tracer topology, both routers must connect to the same Layer 2 LAN or VLAN, and all PCs must be able to reach that shared network.

## 🧰 Technologies Used

| Technology | Purpose |
|---|---|
| Cisco Packet Tracer | Network simulation |
| Cisco 2911 | Router configuration |
| Cisco 2960 | Layer 2 switching |
| HSRP | Default gateway redundancy |
| IPv4 | Network addressing |
| Cisco IOS CLI | Device configuration |
| ICMP / Ping | Connectivity testing |
| Ethernet | Device connectivity |

## 🌐 IP Addressing Plan

All devices use the `192.168.10.0/24` network.

| Device | Interface | IPv4 Address | Subnet Mask |
|---|---|---|---|
| R1 | G0/0 | `192.168.10.2` | `255.255.255.0` |
| R2 | G0/0 | `192.168.10.3` | `255.255.255.0` |
| HSRP virtual gateway | Virtual | `192.168.10.1` | `255.255.255.0` |
| PC1 | FastEthernet0 | `192.168.10.10` | `255.255.255.0` |
| PC2 | FastEthernet0 | `192.168.10.11` | `255.255.255.0` |
| PC3 | FastEthernet0 | `192.168.10.12` | `255.255.255.0` |
| PC4 | FastEthernet0 | `192.168.10.13` | `255.255.255.0` |

**Default gateway for all PCs:** `192.168.10.1`

## ⚙️ Network Configuration

### 1. Configure R1 — Primary Router

Open the R1 CLI and enter:

```cisco
enable
configure terminal
hostname R1

interface gigabitEthernet 0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown
exit

end
```

### 2. Configure R2 — Standby Router

Open the R2 CLI and enter:

```cisco
enable
configure terminal
hostname R2

interface gigabitEthernet 0/0
 ip address 192.168.10.3 255.255.255.0
 no shutdown
exit

end
```

### 3. Configure HSRP on R1

R1 is configured with priority 110, making it preferred over R2, which has priority 100.

```cisco
enable
configure terminal

interface gigabitEthernet 0/0
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

end
```

**Configuration details:**

- HSRP group: `1`
- Virtual IP: `192.168.10.1`
- Priority: `110`
- Preemption: Enabled

The `standby 1 preempt` command allows R1 to reclaim the active role after recovering, provided it has the highest eligible priority and the HSRP conditions are satisfied.

### 4. Configure HSRP on R2

```cisco
enable
configure terminal

interface gigabitEthernet 0/0
 standby 1 ip 192.168.10.1
 standby 1 priority 100
 no shutdown
exit

end
```

R2 shares the same HSRP group and virtual IP but has a lower priority.

### 5. Configure PC IP addresses

Open each PC and select **Desktop → IP Configuration**.

| PC | IP Address | Default Gateway |
|---|---|---|
| PC1 | `192.168.10.10` | `192.168.10.1` |
| PC2 | `192.168.10.11` | `192.168.10.1` |
| PC3 | `192.168.10.12` | `192.168.10.1` |
| PC4 | `192.168.10.13` | `192.168.10.1` |

Set the subnet mask to `255.255.255.0` for every PC.

## 🔍 Verification Commands

Use these commands to inspect the router interfaces and HSRP state.

### Verify router interfaces

```cisco
show ip interface brief
```

The LAN interfaces should show `up/up` when the physical links and connected switch ports are operational.

### Verify HSRP status

```cisco
show standby brief
```

Expected initial roles:

| Router | Priority | Expected Role |
|---|---:|---|
| R1 | 110 | Active |
| R2 | 100 | Standby |

The exact result depends on interface state, HSRP configuration, and successful communication between routers.

## 🧪 Failover Testing

### Test 1: Verify normal connectivity

From PC1, open **Desktop → Command Prompt** and run:

```text
ping 192.168.10.1
ping 192.168.10.12
```

The virtual gateway and PC3 should respond when the network is configured correctly.

### Test 2: Simulate primary router failure

On R1, shut down the LAN interface:

```cisco
enable
configure terminal
interface gigabitEthernet 0/0
 shutdown
end
```

Wait for HSRP to detect the failure and converge.

On R2, run:

```cisco
show standby brief
```

Expected result: R2 becomes the active router.

### Test 3: Verify connectivity after failover

From PC1:

```text
ping 192.168.10.1
ping 192.168.10.12
```

Some packet loss may occur during convergence. Subsequent connectivity should succeed if the standby router and shared LAN are operational.

### Test 4: Restore R1

On R1:

```cisco
enable
configure terminal
interface gigabitEthernet 0/0
 no shutdown
end
```

Wait for HSRP to converge, then run:

```cisco
show standby brief
```

Expected result: R1 resumes the active role because it has priority 110 and preemption enabled.

### Test 5: Save device configurations

On each router and switch:

```cisco
copy running-config startup-config
```

Accept the default destination filename when prompted.

## 📊 Expected Results

| Test | Expected Outcome |
|---|---|
| R1 and R2 operational | R1 Active, R2 Standby |
| PC pings virtual gateway | Successful |
| PC-to-PC communication | Successful on the shared LAN |
| R1 LAN interface shut down | R2 becomes Active |
| R1 restored | R1 can resume Active |
| HSRP verification | Correct roles displayed |

These are expected results for a correctly configured lab. Record your actual output and screenshots when testing.

## 🛠️ Troubleshooting

### HSRP does not form

Check:

```cisco
show standby brief
show ip interface brief
show running-config
```

Verify that both routers use the same HSRP group and virtual IP, are connected to the same Layer 2 network, and have operational interfaces.

### R2 does not become Active

- Confirm R1's LAN interface is actually down.
- Check that R2's LAN interface is operational.
- Verify HSRP group configuration.
- Allow time for HSRP convergence.
- Check for interface or VLAN connectivity problems.

### PCs cannot reach the virtual gateway

- Verify the PC IP addresses and subnet masks.
- Verify the default gateway is `192.168.10.1`.
- Confirm the routers and PCs share the same Layer 2 network.
- Check switch cabling and port status.
- Run `show standby brief` on both routers.

### R1 does not reclaim the active role

Confirm that R1 has priority 110, `standby 1 preempt` is configured, its interface is operational, and the HSRP group has converged.

## 📚 Concepts Learned

- First-hop redundancy.
- HSRP active and standby router roles.
- Virtual IP addresses and default gateways.
- HSRP priority and preemption.
- Router interface configuration.
- IPv4 addressing and subnetting.
- Network failure simulation.
- Gateway failover and recovery.
- Cisco IOS verification and troubleshooting.

## 🚀 Future Improvements

- Add a second VLAN and redundant gateways for additional subnets.
- Introduce a Layer 3 routing protocol for upstream connectivity.
- Explore HSRP interface tracking.
- Add redundant uplinks and evaluate spanning-tree behavior.
- Document convergence time and packet loss during failover.
- Add screenshots and a test-results table based on actual observations.

## 📂 Project Structure

```text
Enterprise-HSRP-Network-Redundancy/
├── README.md
├── Enterprise-HSRP-Network-Redundancy.pkt
└── screenshots/
    ├── topology.png
    ├── hsrp-active.png
    ├── hsrp-standby.png
    └── failover-test.png
```

This is a suggested structure. Add only files and screenshots that you have created.

## 👨‍💻 Author

**Andrews GnanaSelvin D**

- GitHub: [Andrewsdavid2005](https://github.com/Andrewsdavid2005)
- LinkedIn: [Andrews GnanaSelvin D](https://www.linkedin.com/in/andrews-gnana-selvin-d)

## 📄 License

This project is intended for educational and portfolio purposes. If you choose the MIT License, include a `LICENSE` file with the official MIT License text.

---

⭐ If this project helped you understand HSRP and network redundancy, feel free to explore the configuration and suggest improvements.
