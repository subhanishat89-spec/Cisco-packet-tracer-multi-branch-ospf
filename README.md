# ABC Corporation — Multi-Branch Network Design & Implementation

A multi-branch enterprise network infrastructure designed, implemented, and simulated in Cisco Packet Tracer.

---

## 📌 Project Overview
This project models a converged enterprise network for ABC Corporation, connecting three geographically separated branch offices[cite: 6]. The network utilizes redundant serial WAN links configured in a triangular full-mesh topology with dynamic OSPF routing, alongside centralized DHCP address assignment, DNS name resolution, and HTTP web hosting located in Branch 2 (Central Office)[cite: 6].

### Key Technical Features
* **Redundant WAN Topology:** 3 branch routers interconnected in a full-mesh WAN using Class A `/30` point-to-point serial links (`10.0.0.0/8`)[cite: 6].
* **Dynamic Routing:** Single-area OSPF (Area 0) deployed across all routers for dynamic route discovery and failover connectivity[cite: 6].
* **Centralized Services:** Centralized DHCP and DNS services hosted on Server0 (`192.168.2.2`) in Branch 2[cite: 6].
* **DHCP Relay:** Configured `ip helper-address` on remote branch routers (Router1 & Router2) to relay broadcast DHCP requests to the central server across subnets[cite: 6].
* **DNS Resolution & Web Server:** Domain Name System resolution mapping `www.abccorp.com` to the central Web Server (`192.168.2.3`) hosting the corporate homepage[cite: 6].

---

## 📐 Network Architecture & Topology

```text
                     [ Branch 1 ]
                   ( Router1 - LAN1 )
                   (192.168.1.0/24)
                    /            \
          Se3/0=10.0.0.2      Se2/0=10.0.0.9
                  /                \
                 /                  \
   Se2/0=10.0.0.1                    Se3/0=10.0.0.10
         /                                  \
  [ Branch 2 - Central ] ------------ [ Branch 3 ]
   ( Router0 - LAN2 )  Se3/0=10.0.0.5  ( Router2 - LAN3 )
   (192.168.2.0/24)    Se2/0=10.0.0.6  (192.168.3.0/24)
   (DHCP, DNS, Web)
```

### Device Summary Table
| Branch | Router | Switch | Connected Devices / Servers | Subnet | Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Branch 1** | Router1 | Switch1 | PC0, PC1 | `192.168.1.0/24` | `192.168.1.1` |
| **Branch 2 (Central)** | Router0 | Switch2 | PC2, PC3, Server0 (DHCP+DNS), Server1 (Web) | `192.168.2.0/24` | `192.168.2.1` |
| **Branch 3** | Router2 | Switch3 | PC4, PC5 | `192.168.3.0/24` | `192.168.3.1` |

---

## 📊 IP Addressing Scheme

### LAN Segments (Class C - `/24`)
| Branch | Network Address | Default Gateway | Subnet Mask | Usable Host Range |
| :--- | :--- | :--- | :--- | :--- |
| **Branch 1 (LAN1)** | `192.168.1.0/24` | `192.168.1.1` | `255.255.255.0` | `192.168.1.2 – 192.168.1.254` |
| **Branch 2 (LAN2)** | `192.168.2.0/24` | `192.168.2.1` | `255.255.255.0` | `192.168.2.2 – 192.168.2.254` |
| **Branch 3 (LAN3)** | `192.168.3.0/24` | `192.168.3.1` | `255.255.255.0` | `192.168.3.2 – 192.168.3.254` |

### WAN Serial Links (Class A - `10.0.0.0/8` subnetted to `/30`)
| WAN Link | Network Subnet | Interface A | Interface B |
| :--- | :--- | :--- | :--- |
| **Router0 ↔ Router1** | `10.0.0.0/30` | Router0 Se2/0 (`10.0.0.1`) | Router1 Se3/0 (`10.0.0.2`) |
| **Router0 ↔ Router2** | `10.0.0.4/30` | Router0 Se3/0 (`10.0.0.5`) | Router2 Se2/0 (`10.0.0.6`) |
| **Router1 ↔ Router2** | `10.0.0.8/30` | Router1 Se2/0 (`10.0.0.9`) | Router2 Se3/0 (`10.0.0.10`) |

### Centralized Server Addressing (Branch 2)
| Server Device | Static IP Address | Role & Domain Service |
| :--- | :--- | :--- |
| **Server0** | `192.168.2.2` | Centralized DHCP Server & DNS Server |
| **Server1** | `192.168.2.3` | Corporate Web Server (`www.abccorp.com`) |

---

## ⚙️ Cisco IOS Router Configuration

### 1. Router0 Configuration (Branch 2 - Central Office)
```cisconetwork
Router0# configure terminal
Router0(config)# interface FastEthernet0/0
Router0(config-if)# ip address 192.168.2.1 255.255.255.0
Router0(config-if)# no shutdown
Router0(config-if)# exit

Router0(config)# interface Serial2/0
Router0(config-if)# ip address 10.0.0.1 255.255.255.252
Router0(config-if)# clock rate 64000
Router0(config-if)# no shutdown
Router0(config-if)# exit

Router0(config)# interface Serial3/0
Router0(config-if)# ip address 10.0.0.5 255.255.255.252
Router0(config-if)# clock rate 64000
Router0(config-if)# no shutdown
Router0(config-if)# exit

Router0(config)# router ospf 1
Router0(config-router)# network 10.0.0.0 0.0.0.3 area 0
Router0(config-router)# network 10.0.0.4 0.0.0.3 area 0
Router0(config-router)# network 192.168.2.0 0.0.0.255 area 0
Router0(config-router)# exit
```

### 2. Router1 Configuration (Branch 1 - Remote Office)
```cisconetwork
Router1# configure terminal
Router1(config)# interface FastEthernet0/0
Router1(config-if)# ip address 192.168.1.1 255.255.255.0
Router1(config-if)# ip helper-address 192.168.2.2
Router1(config-if)# no shutdown
Router1(config-if)# exit

Router1(config)# interface Serial3/0
Router1(config-if)# ip address 10.0.0.2 255.255.255.252
Router1(config-if)# clock rate 64000
Router1(config-if)# no shutdown
Router1(config-if)# exit

Router1(config)# interface Serial2/0
Router1(config-if)# ip address 10.0.0.9 255.255.255.252
Router1(config-if)# clock rate 64000
Router1(config-if)# no shutdown
Router1(config-if)# exit

Router1(config)# router ospf 1
Router1(config-router)# network 10.0.0.0 0.0.0.3 area 0
Router1(config-router)# network 10.0.0.8 0.0.0.3 area 0
Router1(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router1(config-router)# exit
```

### 3. Router2 Configuration (Branch 3 - Remote Office)
```cisconetwork
Router2# configure terminal
Router2(config)# interface FastEthernet0/0
Router2(config-if)# ip address 192.168.3.1 255.255.255.0
Router2(config-if)# ip helper-address 192.168.2.2
Router2(config-if)# no shutdown
Router2(config-if)# exit

Router2(config)# interface Serial2/0
Router2(config-if)# ip address 10.0.0.6 255.255.255.252
Router2(config-if)# clock rate 64000
Router2(config-if)# no shutdown
Router2(config-if)# exit

Router2(config)# interface Serial3/0
Router2(config-if)# ip address 10.0.0.10 255.255.255.252
Router2(config-if)# clock rate 64000
Router2(config-if)# no shutdown
Router2(config-if)# exit

Router2(config)# router ospf 1
Router2(config-router)# network 10.0.0.4 0.0.0.3 area 0
Router2(config-router)# network 10.0.0.8 0.0.0.3 area 0
Router2(config-router)# network 192.168.3.0 0.0.0.255 area 0
Router2(config-router)# exit
```

---

## 🛠 Server Services Configuration

### Centralized DHCP Pools (Server0: `192.168.2.2`)
| Pool Name | Default Gateway | DNS Server | Start IP Address | Subnet Mask | Max Users |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LAN1_POOL** | `192.168.1.1` | `192.168.2.2` | `192.168.1.10` | `255.255.255.0` | 50 |
| **LAN2_POOL** | `192.168.2.1` | `192.168.2.2` | `192.168.2.10` | `255.255.255.0` | 50 |
| **LAN3_POOL** | `192.168.3.1` | `192.168.2.2` | `192.168.3.10` | `255.255.255.0` | 50 |

### DNS Configuration (Server0: `192.168.2.2`)
* **Service:** DNS ON
* **Resource Record:** A Record
* **Name:** `www.abccorp.com`
* **Address:** `192.168.2.3`

### Web Hosting Configuration (Server1: `192.168.2.3`)
* **Service:** HTTP ON
* **File (`index.html`):**
```html
<html>
  <body>
    <h1>Welcome to ABC Corp</h1>
  </body>
</html>
```

---

## 🧪 Verification & Testing

1. **OSPF Neighbor Adjacency Verification:**
   ```text
   Router0# show ip ospf neighbor
   Neighbor ID     Pri   State           Dead Time   Address         Interface
   192.168.1.1       0   FULL/  -        00:00:31    10.0.0.2        Serial2/0
   192.168.3.1       0   FULL/  -        00:00:31    10.0.0.6        Serial3/0
   ```
2. **DHCP Verification:** End devices across all three branches received dynamic IP configurations via `ip helper-address` (e.g., PC0 received `192.168.1.11`, PC2 received `192.168.2.4`, and PC4 received `192.168.3.10`).
3. **Inter-Branch Connectivity (Ping Test):**
   ```text
   PC> ping 192.168.3.11
   Pinging 192.168.3.11 with 32 bytes of data:
   Reply from 192.168.3.11: bytes=32 time=1ms TTL=126
   Reply from 192.168.3.11: bytes=32 time=2ms TTL=126
   Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
   ```
4. **DNS Resolution Test:**
   ```text
   PC> nslookup [www.abccorp.com](https://www.abccorp.com)
   Server:  192.168.2.2
   Address: 192.168.2.2

   Non-authoritative answer:
   Name:    [www.abccorp.com](https://www.abccorp.com)
   Address: 192.168.2.3
   ```
5. **Web Browser Access:** Browsing `http://www.abccorp.com` from workstations on any branch successfully loads the home webpage displaying `"Welcome to ABC Corp"`.

---

## ⚠️ Challenges & Solutions

1. **Serial Cabling Interface Mismatch:**
   * *Issue:* Router0 to Router2 link failed to come up due to a cable plugged into `Se3/0` instead of `Se2/0`.
   * *Resolution:* Diagnosed using `show cdp neighbors detail` and re-cabled to the matching serial interface.
2. **Missing DHCP Relay Configuration:**
   * *Issue:* Workstations on remote branches (Branch 1 and Branch 3) received APIPA addresses (`169.254.x.x`).
   * *Resolution:* Configured `ip helper-address 192.168.2.2` on remote router LAN interfaces (`Fa0/0`), allowing broadcast DHCP request packets to be unicast-relayed across routers to the central DHCP server.

---

## 🚀 How to Run in Packet Tracer

1. Download or clone this repository:
   ```bash
   git clone [https://github.com/subhanishat89-spec/ABC-Corporation-Multi-Branch-Network.git](https://github.com/subhanishat89-spec/ABC-Corporation-Multi-Branch-Network.git)
   ```
2. Open **Cisco Packet Tracer** (v8.0+ recommended).
3. Load the topology file (`.pkt`).
4. Perform ping or web browsing tests directly from any PC workstation.

---

## 📝 Author 
* **Name:** Nishat Subha Mithela (ID: 2023-1-60-248)
