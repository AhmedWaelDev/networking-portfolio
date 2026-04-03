<div align="center">
🔒 Extended ACL Configuration
Show Image
Show Image
Show Image
Show Image

Configure Extended Access Control Lists to filter traffic by source IP, destination IP, protocol, and port number.

</div>

📋 Table of Contents

Objective
Network Topology
Technologies Used
Lab Steps
Verification
Key Concepts
Screenshots
Skills Demonstrated


🎯 Objective
This lab demonstrates how to configure and apply Extended ACLs on Cisco routers to filter network traffic with granular control over source IP, destination IP, protocol type, and port number — unlike standard ACLs which only filter on source IP.

🗺️ Network Topology
[PC-A] ──── [SW1] ──── [R1] ──────── [R2] ──── [SW2] ──── [Server]
              │                        │
           VLAN 10                  VLAN 20
        192.168.1.0/24           192.168.2.0/24

📸 See screenshots/topology.png for the full Packet Tracer topology.


🛠️ Technologies Used
TechnologyPurposeCisco IOS CLIDevice configurationExtended Named ACLsTraffic filtering rulesIPv4 RoutingStatic or dynamic routing between subnetsTCP / UDP / ICMPProtocols being filteredip access-groupApplying ACL to an interface

📝 Lab Steps
Step 1 — Configure IP Addressing
Assign IP addresses to all router interfaces and end devices.
ciscoRouter(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Step 2 — Verify Connectivity Before ACL
Confirm full connectivity before applying any restrictions.
ciscoPC> ping 192.168.2.10
Step 3 — Create the Extended ACL
ciscoRouter(config)# ip access-list extended RESTRICT_TRAFFIC
Router(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 any eq 23
Router(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 any eq 21
Router(config-ext-nacl)# permit ip any any

⚠️ Important: Every ACL ends with an implicit deny any any. Always add permit ip any any at the end if you only want to block specific traffic.

Step 4 — Apply ACL to Interface
ciscoRouter(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group RESTRICT_TRAFFIC in

💡 Placement Rule: Extended ACLs should be placed closest to the source to avoid unnecessary traffic traversing the network.

Step 5 — Verify the ACL
ciscoRouter# show access-lists
Router# show ip interface GigabitEthernet0/0
Step 6 — Test Traffic Filtering
TestExpected ResultTelnet to restricted host❌ DeniedFTP to restricted host❌ DeniedHTTP traffic✅ PermittedICMP Ping✅ Permitted

✅ Verification
CommandPurposeshow access-listsView ACL rules and match countersshow ip interface Gi0/0Confirm ACL is bound to interfaceshow running-configView full ACL in config contextdebug ip packetLive packet filtering trace (use carefully)

💡 Key Concepts
<details>
<summary><b>Extended vs Standard ACL</b></summary>
FeatureStandard ACLExtended ACLFilters by Source IP✅✅Filters by Destination IP❌✅Filters by Protocol❌✅Filters by Port Number❌✅PlacementNear destinationNear sourceACL Number Range1–99100–199
</details>
<details>
<summary><b>ACL Direction — Inbound vs Outbound</b></summary>

in → Filters packets entering the interface (before routing)
out → Filters packets leaving the interface (after routing)

Most commonly, Extended ACLs are applied inbound on the interface closest to the traffic source.
</details>
<details>
<summary><b>Wildcard Masks Quick Reference</b></summary>
Subnet MaskWildcard Mask/32 (255.255.255.255)0.0.0.0/24 (255.255.255.0)0.0.0.255/25 (255.255.255.128)0.0.0.127/30 (255.255.255.252)0.0.0.3Any hosthost x.x.x.xAll trafficany
</details>

📸 Screenshots
ScreenshotDescriptiontopology.pngFull network topologyacl-config.pngACL configuration outputdenied-traffic.pngTelnet blocked by ACLpermitted-traffic.pngHTTP/Ping allowed through

🏆 Skills Demonstrated

✅ Configuring extended named ACLs on Cisco IOS
✅ Filtering traffic by protocol (TCP, UDP, ICMP) and port number
✅ Applying and verifying ACLs on router interfaces
✅ Testing permit/deny behavior using Packet Tracer simulation mode
✅ Understanding ACL placement best practices


<div align="center">
⬆ Back to Top  |  🏠 Back to Main Portfolio
</div>
