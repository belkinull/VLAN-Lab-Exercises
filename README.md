# Two Complete CCNA-Level VLAN Lab Exercises
## LAB 1 - Router-on-a-Stick (ROAS)
### 1. Scenario Description
Company: AlphaCorp, a growing mid-sized company with 45 employees spread across three departments operating from a single office floor. The IT team has decided to segment the network for security and performance. All traffic leaving the local network must be routed through a single router that also provides internet access. Because budget is limited, there is only one physical router and one switch, the router must handle all inter-VLAN routing.
| VLAN ID | Name |Purpose | Subnet | Gateway | Usable Hosts |
| ------- | ---- | ------ | ------ | ------- | ------------ |
| 10 | HR | Human Resources PCs | 192.168.10.0/24 | 192.168.10.1 | .2–.254 |
| 20 | IT | IT department PCs and servers | 192.168.20.0/24 | 192.168.20.1 | .2–.254 |
| 30 | Sales | Sales team PCs | 192.168.30.0/24 | 192.168.30.1 | .2–.254 |
| 999 | NATIVE | Unused native VLAN (security) | - | - | - |
### 2. Network Topology
Devices:

* R1: Cisco 1941 Router - performs all inter-VLAN routing via sub-interfaces on Gi0/0
* SW1: Cisco 2960 Layer 2 Switch - carries all three VLANs; uplink to R1 is a trunk
* PC-HR1, PC-HR2: Human Resources workstations
* PC-IT1, PC-IT2: IT department workstations
* PC-Sales1, PC-Sales2: Sales workstations

Interface connections:
| From | Interface |To | Interface | Link Type |
| ------- | ---- | ------ | ------ | ------- |
| R1      | Gi0/0 | SW1 | Gi0/1 | Trunk (802.1Q) |
| SW1     | Fa0/1 | PC-HR1 | NIC | Access VLAN 10 |
| SW1     | Fa0/2 | PC-HR2 | NIC | Access VLAN 10 |
| SW1     | Fa0/4 | PC-IT1 | NIC | Access VLAN 20 |
| SW1     | Fa0/5 | PC-IT2 | NIC | Access VLAN 20 |
| SW1     | Fa0/7 | PC-Sales1 | NIC | Access VLAN 30 |
| SW1     | Fa0/8 | PC-Sales2 | NIC | Access VLAN 30 |
### 3. Objectives
By the end of this lab you must achieve:
1. All devices within the same VLAN communicate freely without routing
2. Devices in different VLANs can communicate through R1
3. The trunk link between SW1 and R1 carries only VLANs 10, 20, 30, no others
4. The native VLAN is changed from the default VLAN 1 to VLAN 999 on all trunk links
5. All unused switch ports are disabled and assigned to VLAN 999
6. Each PC can ping its default gateway
7. Each PC can ping any PC in any other VLAN
### 4. Tasks
Task 1 - Create VLANs 10, 20, 30, and 999 on SW1 with correct names

Task 2 - Assign Fa0/1–Fa0/2 to VLAN 10 as access ports

Task 3 - Assign Fa0/4–Fa0/5 to VLAN 20 as access ports

Task 4 - Assign Fa0/7–Fa0/8 to VLAN 30 as access ports

Task 5 - Shut down and assign unused ports (Fa0/3, Fa0/6, Fa0/9–Fa0/24) to VLAN 999

Task 6 - Configure Gi0/1 on SW1 as a trunk port, allowed VLANs 10, 20, 30 and native VLAN 999

Task 7 - On R1, bring up Gi0/0 with no IP address (physical parent interface)

Task 8 - Create sub-interface Gi0/0.10 with encapsulation dot1q 10 and IP 192.168.10.1/24

Task 9 - Create sub-interface Gi0/0.20 with encapsulation dot1q 20 and IP 192.168.20.1/24

Task 10 - Create sub-interface Gi0/0.30 with encapsulation dot1q 30 and IP 192.168.30.1/24

Task 11 - Configure all PCs with their correct IP address, subnet mask, and default gateway

Task 12 - Verify full connectivity with ping tests

### 5. Full Detailed Solution
R1 - Complete Configuration

```
!============================================================
! R1 - AlphaCorp Router
! Role: Router-on-a-Stick, inter-VLAN routing for VLANs 10/20/30
!============================================================

R1> enable
R1# configure terminal

!--- Set hostname for identification
R1(config)# hostname R1

!--- Disable DNS lookup to prevent delays on typos
R1(config)# no ip domain-lookup

!--- STEP 1: Activate the physical interface
!--- The physical Gi0/0 gets NO IP address in ROAS
!--- It is just the physical carrier for the sub-interfaces
!--- Without 'no shutdown' here, ALL sub-interfaces stay down
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description TRUNK-TO-SW1
R1(config-if)# no ip address
R1(config-if)# no shutdown
R1(config-if)# exit

!--- STEP 2: Create sub-interface for VLAN 10 (HR)
!--- The .10 after the slash is the sub-interface number (matches VLAN ID by convention)
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description VLAN10-HR-GATEWAY
!--- encapsulation dot1q 10: This sub-interface handles frames tagged with VLAN 10
!--- This is the command that BINDS the sub-interface to VLAN 10
R1(config-subif)# encapsulation dot1q 10
!--- This IP becomes the default gateway for ALL devices in VLAN 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit

!--- STEP 3: Create sub-interface for VLAN 20 (IT)
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description VLAN20-IT-GATEWAY
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit

!--- STEP 4: Create sub-interface for VLAN 30 (Sales)
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# description VLAN30-SALES-GATEWAY
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit

!--- STEP 5: Add a default route pointing to ISP (replace x.x.x.x with real ISP gateway)
R1(config)# ip route 0.0.0.0 0.0.0.0 [ISP_GATEWAY_IP]

!--- STEP 6: Save configuration to NVRAM
R1(config)# end
R1# copy running-config startup-config
```
SW1 - Complete Configuration
```
!============================================================
! SW1 - AlphaCorp Access Switch (Cisco 2960, Layer 2 only)
! Role: Access switch for all user VLANs, trunk to R1
!============================================================

SW1> enable
SW1# configure terminal

!--- Set hostname
SW1(config)# hostname SW1

!--- Disable DNS lookup
SW1(config)# no ip domain-lookup

!============================================================
! STEP 1: CREATE ALL VLANs IN THE DATABASE
! IMPORTANT: Always create VLANs before assigning ports to them
! If you assign a port to a non-existent VLAN, the port goes inactive
!============================================================

SW1(config)# vlan 10
SW1(config-vlan)# name HR
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name IT
SW1(config-vlan)# exit

SW1(config)# vlan 30
SW1(config-vlan)# name Sales
SW1(config-vlan)# exit

!--- VLAN 999 is used as the native VLAN (security best practice)
!--- and as the "dead zone" for unused ports
SW1(config)# vlan 999
SW1(config-vlan)# name NATIVE_UNUSED
SW1(config-vlan)# exit

!============================================================
! STEP 2: CONFIGURE ACCESS PORTS FOR VLAN 10 (HR)
! Fa0/1 and Fa0/2 connect to PC-HR1 and PC-HR2
!============================================================

SW1(config)# interface FastEthernet0/1
SW1(config-if)# description PC-HR1-ACCESS
!--- 'switchport mode access' explicitly sets port to access mode
!--- This prevents DTP from negotiating a trunk with the PC (security risk)
SW1(config-if)# switchport mode access
!--- Assigns this port permanently to VLAN 10
!--- Frames arriving here are internally tagged VLAN 10
SW1(config-if)# switchport access vlan 10
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/2
SW1(config-if)# description PC-HR2-ACCESS
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# no shutdown
SW1(config-if)# exit

!============================================================
! STEP 3: CONFIGURE ACCESS PORTS FOR VLAN 20 (IT)
! Fa0/4 and Fa0/5 connect to PC-IT1 and PC-IT2
!============================================================

SW1(config)# interface FastEthernet0/4
SW1(config-if)# description PC-IT1-ACCESS
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/5
SW1(config-if)# description PC-IT2-ACCESS
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# no shutdown
SW1(config-if)# exit

!============================================================
! STEP 4: CONFIGURE ACCESS PORTS FOR VLAN 30 (Sales)
! Fa0/7 and Fa0/8 connect to PC-Sales1 and PC-Sales2
!============================================================

SW1(config)# interface FastEthernet0/7
SW1(config-if)# description PC-SALES1-ACCESS
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/8
SW1(config-if)# description PC-SALES2-ACCESS
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# no shutdown
SW1(config-if)# exit

!============================================================
! STEP 5: SHUT DOWN AND ISOLATE ALL UNUSED PORTS
! Best practice: unused ports go to VLAN 999 (dead zone) and shut
! This prevents rogue device attacks and unauthorized access
!============================================================

SW1(config)# interface FastEthernet0/3
SW1(config-if)# description UNUSED
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 999
SW1(config-if)# shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/6
SW1(config-if)# description UNUSED
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 999
SW1(config-if)# shutdown
SW1(config-if)# exit

!--- Apply to remaining unused ports Fa0/9 through Fa0/24 in one command
SW1(config)# interface range FastEthernet0/9 - 24
SW1(config-if-range)# description UNUSED
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 999
SW1(config-if-range)# shutdown
SW1(config-if-range)# exit

!============================================================
! STEP 6: CONFIGURE THE TRUNK PORT (Gi0/1 → R1)
! This is the most important port on SW1
! It carries tagged traffic for VLANs 10, 20, and 30 to R1
!============================================================

SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description TRUNK-TO-R1
!--- On a 2960, this command may not be needed (dot1q is only option)
!--- but we include it explicitly for clarity and exam purposes
SW1(config-if)# switchport trunk encapsulation dot1q
!--- Force trunk mode. Never leave uplinks in dynamic mode.
SW1(config-if)# switchport mode trunk
!--- Only allow the 3 user VLANs on this trunk
!--- Default is "all VLANs 1-4094" which is insecure and wasteful
SW1(config-if)# switchport trunk allowed vlan 10,20,30
!--- Change native VLAN from default VLAN 1 to VLAN 999
!--- VLAN 1 as native is a well-known attack vector (double-tagging)
SW1(config-if)# switchport trunk native vlan 999
SW1(config-if)# no shutdown
SW1(config-if)# exit

!============================================================
! STEP 7: CONFIGURE MANAGEMENT SVI (optional but recommended)
! Allows SSH/Telnet management of this switch
!============================================================

SW1(config)# interface vlan 10
SW1(config-if)# description MANAGEMENT-SVI
SW1(config-if)# ip address 192.168.10.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

!--- Default gateway so the switch can be reached from other subnets
SW1(config)# ip default-gateway 192.168.10.1

!--- Save
SW1(config)# end
SW1# copy running-config startup-config
```
PC IP Configuration
```
PC-HR1:
  IP Address  : 192.168.10.10
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.10.1

PC-HR2:
  IP Address  : 192.168.10.11
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.10.1

PC-IT1:
  IP Address  : 192.168.20.10
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.20.1

PC-IT2:
  IP Address  : 192.168.20.11
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.20.1

PC-Sales1:
  IP Address  : 192.168.30.10
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.30.1

PC-Sales2:
  IP Address  : 192.168.30.11
  Subnet Mask : 255.255.255.0
  Gateway     : 192.168.30.1
```
### 6. Verification Commands
On SW1:
```
!--- See all VLANs, their status, and which ports belong to each
SW1# show vlan brief

!--- Expected output (excerpt):
! VLAN  Name              Status    Ports
! ----  ----------------  --------  ----------------------
! 10    HR                active    Fa0/1, Fa0/2
! 20    IT                active    Fa0/4, Fa0/5
! 30    Sales             active    Fa0/7, Fa0/8
! 999   NATIVE_UNUSED     active    Fa0/3, Fa0/6, Fa0/9-24

!--- Verify trunk configuration in detail
SW1# show interfaces trunk

!--- Expected output (excerpt):
! Port      Mode    Encapsulation  Status    Native vlan
! Gi0/1     on      802.1q         trunking  999
!
! Port      Vlans allowed on trunk
! Gi0/1     10,20,30
!
! Port      Vlans allowed and active in management domain
! Gi0/1     10,20,30
!
! Port      Vlans in spanning tree forwarding state and not pruned
! Gi0/1     10,20,30

!--- Check specific interface switchport details
SW1# show interfaces FastEthernet0/1 switchport
SW1# show interfaces GigabitEthernet0/1 switchport

!--- Check MAC address table to confirm devices are learned
SW1# show mac address-table
```
On R1:
```
!--- Verify all interfaces and sub-interfaces are up
R1# show ip interface brief

!--- Expected output:
! Interface              IP-Address       OK?  Method  Status                Protocol
! GigabitEthernet0/0     unassigned       YES  unset   up                    up
! GigabitEthernet0/0.10  192.168.10.1     YES  manual  up                    up
! GigabitEthernet0/0.20  192.168.20.1     YES  manual  up                    up
! GigabitEthernet0/0.30  192.168.30.1     YES  manual  up                    up

!--- Verify the routing table has all three connected subnets
R1# show ip route

!--- Expected output (excerpt):
! C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
! C    192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
! C    192.168.30.0/24 is directly connected, GigabitEthernet0/0.30

!--- Inspect encapsulation on a sub-interface
R1# show interfaces GigabitEthernet0/0.10
```
Ping Test Sequence:
```
!--- From PC-HR1 (192.168.10.10):

Step 1 - Ping own gateway (tests VLAN 10 can reach R1):
> ping 192.168.10.1        Expected: SUCCESS

Step 2 - Ping same-VLAN host (tests L2 switching within VLAN 10):
> ping 192.168.10.11       Expected: SUCCESS

Step 3 - Ping IT gateway (tests inter-VLAN routing VLAN10→VLAN20):
> ping 192.168.20.1        Expected: SUCCESS

Step 4 - Ping IT host (tests end-to-end inter-VLAN):
> ping 192.168.20.10       Expected: SUCCESS

Step 5 - Ping Sales host (tests inter-VLAN VLAN10→VLAN30):
> ping 192.168.30.10       Expected: SUCCESS

!--- From PC-IT1 (192.168.20.10):
> ping 192.168.10.10       Expected: SUCCESS (IT to HR)
> ping 192.168.30.11       Expected: SUCCESS (IT to Sales)

!--- From PC-Sales1 (192.168.30.10):
> ping 192.168.20.11       Expected: SUCCESS (Sales to IT)
> ping 192.168.10.10       Expected: SUCCESS (Sales to HR)
```
### 7. Troubleshooting Section
#### Problem 1 - Sub-interfaces are down even though they are configured\
Symptom: `show ip interface brief` shows Gi0/0.10 as `down/down`\
Root cause: The physical parent interface Gi0/0 was never activated. Sub-interfaces inherit the state of their parent. If the parent is `administratively down`, all sub-interfaces are down regardless of their own configuration.
Fix:
```
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no shutdown
```
#### Problem 2 - Same-VLAN pings work, but inter-VLAN pings fail completely
Symptom: PC-HR1 can ping PC-HR2 (VLAN 10 → VLAN 10) but cannot ping PC-IT1 (VLAN 10 → VLAN 20)
Root cause A: The `encapsulation dot1q` command is missing or uses the wrong VLAN ID on a sub-interface. Without it, R1 doesn't know which VLAN belongs to which sub-interface.\
Root cause B: The trunk's allowed VLAN list doesn't include all necessary VLANs.\
Root cause C: PC's default gateway is wrong or not configured at all.\
Fix - check each:
```
R1# show interfaces GigabitEthernet0/0.10
! Look for: "Encapsulation 802.1Q Virtual LAN, Vlan ID 10"
! If encapsulation is missing, reconfigure:
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# encapsulation dot1q 10

SW1# show interfaces trunk
! Check "Vlans allowed on trunk" row - all VLANs must appear here

! On the PC - verify gateway is exactly 192.168.10.1, not .10.0 or .10.10
```
#### Problem 3 - Native VLAN mismatch warning in logs
Symptom: Console shows `%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/0.`\
Root cause: One end of the trunk has native VLAN 999 configured but the other end still has the default native VLAN 1. Both ends of a trunk must agree on the native VLAN.\
Fix - both ends must match:
```
!--- On SW1:
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport trunk native vlan 999

!--- On R1, the native VLAN sub-interface uses 'native' keyword:
R1(config)# interface GigabitEthernet0/0.999
R1(config-subif)# encapsulation dot1q 999 native
!--- This sub-interface typically has no IP - it just declares the native VLAN
```
#### Problem 4 - A VLAN is in the allowed trunk list but traffic still doesn't pass
Symptom: VLAN 30 is in `switchport trunk allowed vlan 10,20,30` but Sales PCs can't communicate\
Root cause: VLAN 30 exists in the allowed list but doesn't exist in the switch's VLAN database. A VLAN must be both created AND allowed on the trunk to function. Check the fourth line of `show interfaces trunk` - "VLANs in spanning tree forwarding state", if VLAN 30 is missing there, it either doesn't exist in the database or STP is blocking it.\
Fix:
```
SW1# show vlan brief
! If VLAN 30 doesn't appear, create it:
SW1(config)# vlan 30
SW1(config-vlan)# name Sales
SW1(config-vlan)# exit
```
#### Problem 5 - Adding a new VLAN later destroys existing allowed VLAN list
Symptom: Everything works. You add VLAN 40 with the command `switchport trunk allowed vlan 40` and now VLANs 10, 20, 30 stop working.\
Root cause: `switchport trunk allowed vlan [id]` REPLACES the entire list, not adds to it. This is one of the most dangerous and common mistakes on the CCNA exam.\
Fix - always use `add` keyword when appending to an existing list:
```
!--- WRONG (replaces the list - VLANs 10, 20, 30 are now removed from trunk):
SW1(config-if)# switchport trunk allowed vlan 40

!--- CORRECT (adds VLAN 40 while keeping 10, 20, 30):
SW1(config-if)# switchport trunk allowed vlan add 40

!--- Verify after:
SW1# show interfaces trunk
! Should now show: 10,20,30,40
```
### 8. Bonus - Exam Trick Questions
Exam Trick Question 1:
A student configures ROAS correctly. `show ip interface brief` shows all sub-interfaces as `up/up`. All PCs have correct IPs and gateways. But PC-HR1 cannot ping PC-IT1. PC-HR1 CAN ping its own gateway (192.168.10.1). What is the most likely cause?\

Answer: The trunk port on SW1 is configured with `switchport trunk allowed vlan 10,20,30`, but the student forgot to also configure the native VLAN correctly, OR more commonly, they forgot `encapsulation dot1q 20` on R1's Gi0/0.20 sub-interface. If the encapsulation is missing on the destination sub-interface, R1 receives the frame on Gi0/0 but cannot classify it to any sub-interface, it drops it. The source VLAN works because .10 has encapsulation. The destination fails because .20 does not.\

A second common answer: the student typed `switchport trunk allowed vlan 20` instead of `switchport trunk allowed vlan add 20` on SW1, which replaced the list and removed VLAN 10, making it appear VLAN 10 (HR) can't reach anything outside, when in fact the trunk is only passing VLAN 20 now.

Exam Trick Question 2:
On R1, a student runs show ip route and sees all three connected subnets (192.168.10.0, 192.168.20.0, 192.168.30.0) as directly connected. But every inter-VLAN ping fails. The switch trunk shows all VLANs active. What single command is most likely missing?\
Answer: The physical interface no shutdown. Even though the routing table shows connected routes for sub-interfaces (the routes are built when the sub-interfaces are configured, regardless of physical state on some IOS versions), if GigabitEthernet0/0 is administratively down, no frames actually pass through it. The routing table can be misleading here, it shows the routes as connected even when the interface is shutdown in some Packet Tracer versions. Always verify with show ip interface brief and look for up/up on the physical interface, not just the sub-interfaces.\

## LAB 2 - Inter-VLAN Routing via Layer 3 Switch (SVI)
### 1. Scenario Description
Company: BetaTech Solutions - a larger company with 80+ employees across two floors. The IT team has upgraded the network infrastructure with a Layer 3 core switch. The goal is to eliminate the external router as the single point of failure for inter-VLAN routing and move that function inside the switch fabric itself using SVIs. A dedicated management VLAN keeps switch management traffic completely separate from user traffic.\
Note: This lab uses the `10.10.x.x` addressing scheme deliberately different from Lab 1, so you practice reading and not confusing subnet assignments.
| VLAN ID | Name |Purpose | Subnet | Gateway | Notes |
| ------- | ---- | ------ | ------ | ------- | ------------ |
| 10 | HR | Human Resources PCs | 10.10.10.0/24 | 10.10.10.1 | SVI on CORE-SW |
| 20 | IT | IT department PCs and servers | 10.10.20.0/2 | 10.10.20.1 | SVI on CORE-SW |
| 30 | Sales | Sales team PCs | 10.10.30.0/24 | 10.10.30.1 | SVI on CORE-SW |
| 99 | Management | Switch management | 10.10.99.0/24 | 10.10.99.1 | Out-of-band mgmt |
| 999 | NATIVE | Unused native VLAN | - | - | Security |
### 2. Network Topology
Devices:
* CORE-SW: Cisco Catalyst 3560 - Layer 3 switch, routing engine enabled, SVIs for all VLANs, WAN uplink to ISP
* SW-A: Cisco 2960 - Layer 2 access switch, Floor 1, carries VLAN 10 (HR) and VLAN 20 (IT)
* SW-B: Cisco 2960 - Layer 2 access switch, Floor 2, carries VLAN 30 (Sales) only
* 7 PCs across three departments

Interface connections:

| From Device | Interface | To Device | Interface | Link Type |
|------------|----------|----------|----------|-----------|
| CORE-SW | Gi0/1 | ISP Router | - | WAN / Routed |
| CORE-SW | Gi0/2 | SW-A | Gi0/1 | Trunk (VLANs 10,20,99) |
| CORE-SW | Gi0/3 | SW-B | Gi0/1 | Trunk (VLANs 30,99) |
| SW-A | Fa0/1 | PC-HR1 | NIC | Access VLAN 10 |
| SW-A | Fa0/2 | PC-HR2 | NIC | Access VLAN 10 |
| SW-A | Fa0/5 | PC-IT1 | NIC | Access VLAN 20 |
| SW-A | Fa0/6 | PC-IT2 | NIC | Access VLAN 20 |
| SW-B | Fa0/1 | PC-Sales1 | NIC | Access VLAN 30 |
| SW-B | Fa0/2 | PC-Sales2 | NIC | Access VLAN 30 |
| SW-B | Fa0/3 | PC-Sales3 | NIC | Access VLAN 30 |
### 3. Objectives
1. CORE-SW performs all inter-VLAN routing internally using SVIs - no external router is needed for local traffic
2. Each VLAN has a corresponding SVI on CORE-SW that acts as the default gateway
3. ip routing is enabled on CORE-SW so the switch can route between VLANs
4. SW-A and SW-B are pure Layer 2 - no routing, no SVIs except for management
5. Trunk links carry only the VLANs they need - no unnecessary VLAN flooding
6. All switches are manageable via VLAN 99 with unique management IPs
7. All PCs across all VLANs can communicate with each other through CORE-SW
8. All unused ports on all switches are disabled and in VLAN 999
### 4. Tasks
Task 1 - On CORE-SW, enable ip routing to activate the Layer 3 routing engine

Task 2 - Create VLANs 10, 20, 30, 99, 999 on ALL three switches (CORE-SW, SW-A, SW-B)

Task 3 - On CORE-SW, create SVIs for VLAN 10, 20, 30, and 99 with correct IP addresses

Task 4 - On CORE-SW, configure Gi0/2 as a trunk to SW-A (allow VLANs 10, 20, 99)

Task 5 - On CORE-SW, configure Gi0/3 as a trunk to SW-B (allow VLANs 30, 99)

Task 6 - On SW-A, configure access ports Fa0/1–Fa0/2 for VLAN 10 and Fa0/5–Fa0/6 for VLAN 20

Task 7 - On SW-A, configure Gi0/1 as a trunk to CORE-SW (allow VLANs 10, 20, 99)

Task 8 - On SW-B, configure access ports Fa0/1–Fa0/3 for VLAN 30

Task 9 - On SW-B, configure Gi0/1 as a trunk to CORE-SW (allow VLANs 30, 99)

Task 10 - Configure management SVIs on SW-A and SW-B in VLAN 99, with default gateways

Task 11 - Disable all unused ports on all three switches, assign to VLAN 999

Task 12 - Configure PC IP addresses, masks, and gateways

Task 13 - Verify full inter-VLAN connectivity and management reachability

### 5. Full Detailed Solution
CORE-SW - Complete Configuration
```
!============================================================
! CORE-SW - BetaTech Solutions Layer 3 Core Switch
! Model: Cisco Catalyst 3560
! Role: Inter-VLAN routing via SVIs, trunk hub for access switches
!============================================================

CORE-SW> enable
CORE-SW# configure terminal

!--- Device identification
CORE-SW(config)# hostname CORE-SW
CORE-SW(config)# no ip domain-lookup

!============================================================
! STEP 1: ENABLE LAYER 3 ROUTING
! THIS IS THE SINGLE MOST IMPORTANT COMMAND IN THIS LAB
! Without it, the switch operates as a pure L2 device
! SVIs will have IPs but traffic will NOT be routed between VLANs
! You will see all SVIs up but zero inter-VLAN pings succeed
!============================================================

CORE-SW(config)# ip routing

!============================================================
! STEP 2: CREATE ALL VLANs IN THE DATABASE
! Must be done before any port assignments or SVI creation
!============================================================

CORE-SW(config)# vlan 10
CORE-SW(config-vlan)# name HR
CORE-SW(config-vlan)# exit

CORE-SW(config)# vlan 20
CORE-SW(config-vlan)# name IT
CORE-SW(config-vlan)# exit

CORE-SW(config)# vlan 30
CORE-SW(config-vlan)# name Sales
CORE-SW(config-vlan)# exit

CORE-SW(config)# vlan 99
CORE-SW(config-vlan)# name Management
CORE-SW(config-vlan)# exit

CORE-SW(config)# vlan 999
CORE-SW(config-vlan)# name NATIVE_UNUSED
CORE-SW(config-vlan)# exit

!============================================================
! STEP 3: CREATE SVIs (Switched Virtual Interfaces)
! Each SVI is the Layer 3 gateway for its VLAN
! When a PC sends a packet to another VLAN, it goes to its SVI
! The routing engine on CORE-SW then forwards it to the destination SVI
!============================================================

!--- SVI for VLAN 10 (HR) - becomes default gateway for all HR PCs
CORE-SW(config)# interface vlan 10
CORE-SW(config-if)# description SVI-VLAN10-HR-GATEWAY
!--- This IP address is what PC-HR1 and PC-HR2 will use as their gateway
CORE-SW(config-if)# ip address 10.10.10.1 255.255.255.0
!--- SVIs are UP only if at least one port in that VLAN is active
!--- We still explicitly configure 'no shutdown' for clarity
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!--- SVI for VLAN 20 (IT)
CORE-SW(config)# interface vlan 20
CORE-SW(config-if)# description SVI-VLAN20-IT-GATEWAY
CORE-SW(config-if)# ip address 10.10.20.1 255.255.255.0
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!--- SVI for VLAN 30 (Sales)
CORE-SW(config)# interface vlan 30
CORE-SW(config-if)# description SVI-VLAN30-SALES-GATEWAY
CORE-SW(config-if)# ip address 10.10.30.1 255.255.255.0
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!--- SVI for VLAN 99 (Management)
!--- This SVI is the management gateway for the switches themselves
!--- Also allows CORE-SW to be managed via VLAN 99
CORE-SW(config)# interface vlan 99
CORE-SW(config-if)# description SVI-VLAN99-MANAGEMENT
CORE-SW(config-if)# ip address 10.10.99.1 255.255.255.0
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!============================================================
! STEP 4: CONFIGURE TRUNK PORT TO SW-A (Gi0/2)
! This trunk carries VLAN 10 (HR) and VLAN 20 (IT) and VLAN 99 (Mgmt)
! SW-B doesn't need VLAN 10 or 20 - so we don't allow them there
!============================================================

CORE-SW(config)# interface GigabitEthernet0/2
CORE-SW(config-if)# description TRUNK-TO-SW-A
!--- On a 3560, this command IS required (unlike 2960 which is dot1q-only)
CORE-SW(config-if)# switchport trunk encapsulation dot1q
!--- Force trunk - never rely on auto-negotiation for uplinks
CORE-SW(config-if)# switchport mode trunk
!--- Allow only the VLANs Floor 1 needs - deny everything else
CORE-SW(config-if)# switchport trunk allowed vlan 10,20,99
!--- Native VLAN to unused VLAN 999 - security hardening
CORE-SW(config-if)# switchport trunk native vlan 999
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!============================================================
! STEP 5: CONFIGURE TRUNK PORT TO SW-B (Gi0/3)
! This trunk carries only VLAN 30 (Sales) and VLAN 99 (Mgmt)
! Floor 2 only has Sales devices - HR and IT traffic stays on Floor 1
!============================================================

CORE-SW(config)# interface GigabitEthernet0/3
CORE-SW(config-if)# description TRUNK-TO-SW-B
CORE-SW(config-if)# switchport trunk encapsulation dot1q
CORE-SW(config-if)# switchport mode trunk
CORE-SW(config-if)# switchport trunk allowed vlan 30,99
CORE-SW(config-if)# switchport trunk native vlan 999
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!============================================================
! STEP 6: CONFIGURE WAN UPLINK TO ISP (Gi0/1)
! On a Layer 3 switch, routed ports use 'no switchport' to
! turn them into a pure Layer 3 interface (like a router port)
! This is called a "routed port" - it has an IP but no VLAN
!============================================================

CORE-SW(config)# interface GigabitEthernet0/1
CORE-SW(config-if)# description WAN-TO-ISP
!--- 'no switchport' converts this from a switchport to a routed port
!--- After this command the port behaves exactly like a router interface
CORE-SW(config-if)# no switchport
CORE-SW(config-if)# ip address 203.0.113.2 255.255.255.252
CORE-SW(config-if)# no shutdown
CORE-SW(config-if)# exit

!--- Default route to internet via ISP gateway
CORE-SW(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

!============================================================
! STEP 7: DISABLE AND ISOLATE UNUSED PORTS ON CORE-SW
! Core switches typically have fewer access ports, but harden all unused
!============================================================

CORE-SW(config)# interface range GigabitEthernet0/4 - 24
CORE-SW(config-if-range)# description UNUSED
CORE-SW(config-if-range)# switchport trunk encapsulation dot1q
CORE-SW(config-if-range)# switchport mode access
CORE-SW(config-if-range)# switchport access vlan 999
CORE-SW(config-if-range)# shutdown
CORE-SW(config-if-range)# exit

!--- Save
CORE-SW(config)# end
CORE-SW# copy running-config startup-config
```
SW-A - Complete Configuration
```
!============================================================
! SW-A - BetaTech Floor 1 Access Switch (Cisco 2960, L2 only)
! Role: Access layer for HR (VLAN 10) and IT (VLAN 20)
!============================================================

SW-A> enable
SW-A# configure terminal

SW-A(config)# hostname SW-A
SW-A(config)# no ip domain-lookup

!============================================================
! STEP 1: CREATE VLANs
! SW-A only needs the VLANs relevant to Floor 1
! We still create VLAN 999 for unused port assignment
!============================================================

SW-A(config)# vlan 10
SW-A(config-vlan)# name HR
SW-A(config-vlan)# exit

SW-A(config)# vlan 20
SW-A(config-vlan)# name IT
SW-A(config-vlan)# exit

SW-A(config)# vlan 99
SW-A(config-vlan)# name Management
SW-A(config-vlan)# exit

SW-A(config)# vlan 999
SW-A(config-vlan)# name NATIVE_UNUSED
SW-A(config-vlan)# exit

!============================================================
! STEP 2: CONFIGURE HR ACCESS PORTS (VLAN 10)
! Fa0/1 → PC-HR1, Fa0/2 → PC-HR2
!============================================================

SW-A(config)# interface FastEthernet0/1
SW-A(config-if)# description PC-HR1-VLAN10
SW-A(config-if)# switchport mode access
SW-A(config-if)# switchport access vlan 10
!--- 'no shutdown' ensures the port is administratively up
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

SW-A(config)# interface FastEthernet0/2
SW-A(config-if)# description PC-HR2-VLAN10
SW-A(config-if)# switchport mode access
SW-A(config-if)# switchport access vlan 10
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

!============================================================
! STEP 3: CONFIGURE IT ACCESS PORTS (VLAN 20)
! Fa0/5 → PC-IT1, Fa0/6 → PC-IT2
! (Fa0/3 and Fa0/4 are intentionally skipped - will be unused)
!============================================================

SW-A(config)# interface FastEthernet0/5
SW-A(config-if)# description PC-IT1-VLAN20
SW-A(config-if)# switchport mode access
SW-A(config-if)# switchport access vlan 20
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

SW-A(config)# interface FastEthernet0/6
SW-A(config-if)# description PC-IT2-VLAN20
SW-A(config-if)# switchport mode access
SW-A(config-if)# switchport access vlan 20
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

!============================================================
! STEP 4: DISABLE ALL UNUSED PORTS
! Fa0/3, Fa0/4, Fa0/7 through Fa0/24 are unused on SW-A
!============================================================

SW-A(config)# interface range FastEthernet0/3 - 4
SW-A(config-if-range)# description UNUSED
SW-A(config-if-range)# switchport mode access
SW-A(config-if-range)# switchport access vlan 999
SW-A(config-if-range)# shutdown
SW-A(config-if-range)# exit

SW-A(config)# interface range FastEthernet0/7 - 24
SW-A(config-if-range)# description UNUSED
SW-A(config-if-range)# switchport mode access
SW-A(config-if-range)# switchport access vlan 999
SW-A(config-if-range)# shutdown
SW-A(config-if-range)# exit

!============================================================
! STEP 5: CONFIGURE TRUNK UPLINK TO CORE-SW (Gi0/1)
!============================================================

SW-A(config)# interface GigabitEthernet0/1
SW-A(config-if)# description TRUNK-TO-CORE-SW
!--- On 2960, encapsulation is dot1q only - but we include it for exam clarity
SW-A(config-if)# switchport trunk encapsulation dot1q
SW-A(config-if)# switchport mode trunk
!--- Allow VLANs 10, 20, and 99 - must match what CORE-SW allows on its Gi0/2
SW-A(config-if)# switchport trunk allowed vlan 10,20,99
SW-A(config-if)# switchport trunk native vlan 999
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

!============================================================
! STEP 6: CONFIGURE MANAGEMENT SVI
! This gives SW-A an IP address for remote management
! VLAN 99 is used - kept completely separate from user VLANs
!============================================================

SW-A(config)# interface vlan 99
SW-A(config-if)# description MANAGEMENT
!--- Each switch gets a unique IP in the management subnet
SW-A(config-if)# ip address 10.10.99.11 255.255.255.0
SW-A(config-if)# no shutdown
SW-A(config-if)# exit

!--- Default gateway for the SWITCH ITSELF (not the PCs)
!--- Points to CORE-SW's VLAN 99 SVI so the switch can be reached remotely
!--- 'ip default-gateway' is used on L2 switches (not 'ip route')
!--- because ip routing is NOT enabled on SW-A
SW-A(config)# ip default-gateway 10.10.99.1

SW-A(config)# end
SW-A# copy running-config startup-config
```
SW-B - Complete Configuration
```
!============================================================
! SW-B - BetaTech Floor 2 Access Switch (Cisco 2960, L2 only)
! Role: Access layer for Sales department (VLAN 30 only)
!============================================================

SW-B> enable
SW-B# configure terminal

SW-B(config)# hostname SW-B
SW-B(config)# no ip domain-lookup

!============================================================
! STEP 1: CREATE VLANs
! SW-B only needs VLAN 30 (Sales), VLAN 99 (Mgmt), VLAN 999 (unused)
! Do NOT create VLAN 10 or 20 here - they don't belong on Floor 2
!============================================================

SW-B(config)# vlan 30
SW-B(config-vlan)# name Sales
SW-B(config-vlan)# exit

SW-B(config)# vlan 99
SW-B(config-vlan)# name Management
SW-B(config-vlan)# exit

SW-B(config)# vlan 999
SW-B(config-vlan)# name NATIVE_UNUSED
SW-B(config-vlan)# exit

!============================================================
! STEP 2: CONFIGURE SALES ACCESS PORTS (VLAN 30)
! Fa0/1 → PC-Sales1, Fa0/2 → PC-Sales2, Fa0/3 → PC-Sales3
!============================================================

SW-B(config)# interface FastEthernet0/1
SW-B(config-if)# description PC-SALES1-VLAN30
SW-B(config-if)# switchport mode access
SW-B(config-if)# switchport access vlan 30
SW-B(config-if)# no shutdown
SW-B(config-if)# exit

SW-B(config)# interface FastEthernet0/2
SW-B(config-if)# description PC-SALES2-VLAN30
SW-B(config-if)# switchport mode access
SW-B(config-if)# switchport access vlan 30
SW-B(config-if)# no shutdown
SW-B(config-if)# exit

SW-B(config)# interface FastEthernet0/3
SW-B(config-if)# description PC-SALES3-VLAN30
SW-B(config-if)# switchport mode access
SW-B(config-if)# switchport access vlan 30
SW-B(config-if)# no shutdown
SW-B(config-if)# exit

!============================================================
! STEP 3: DISABLE ALL UNUSED PORTS
!============================================================

SW-B(config)# interface range FastEthernet0/4 - 24
SW-B(config-if-range)# description UNUSED
SW-B(config-if-range)# switchport mode access
SW-B(config-if-range)# switchport access vlan 999
SW-B(config-if-range)# shutdown
SW-B(config-if-range)# exit

!============================================================
! STEP 4: CONFIGURE TRUNK UPLINK TO CORE-SW (Gi0/1)
! Must match CORE-SW's allowed VLAN list on its Gi0/3
!============================================================

SW-B(config)# interface GigabitEthernet0/1
SW-B(config-if)# description TRUNK-TO-CORE-SW
SW-B(config-if)# switchport trunk encapsulation dot1q
SW-B(config-if)# switchport mode trunk
!--- Only VLAN 30 and management VLAN 99 are needed on this trunk
SW-B(config-if)# switchport trunk allowed vlan 30,99
SW-B(config-if)# switchport trunk native vlan 999
SW-B(config-if)# no shutdown
SW-B(config-if)# exit

!============================================================
! STEP 5: CONFIGURE MANAGEMENT SVI
!============================================================

SW-B(config)# interface vlan 99
SW-B(config-if)# description MANAGEMENT
!--- Different IP from SW-A (.12 vs .11)
SW-B(config-if)# ip address 10.10.99.12 255.255.255.0
SW-B(config-if)# no shutdown
SW-B(config-if)# exit

SW-B(config)# ip default-gateway 10.10.99.1

SW-B(config)# end
SW-B# copy running-config startup-config
```
PC IP Configuration
```
PC-HR1:
  IP Address  : 10.10.10.10
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.10.1       ← CORE-SW SVI vlan 10

PC-HR2:
  IP Address  : 10.10.10.11
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.10.1

PC-IT1:
  IP Address  : 10.10.20.10
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.20.1       ← CORE-SW SVI vlan 20

PC-IT2:
  IP Address  : 10.10.20.11
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.20.1

PC-Sales1:
  IP Address  : 10.10.30.10
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.30.1       ← CORE-SW SVI vlan 30

PC-Sales2:
  IP Address  : 10.10.30.11
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.30.1

PC-Sales3:
  IP Address  : 10.10.30.12
  Subnet Mask : 255.255.255.0
  Gateway     : 10.10.30.1
```
### 6. Verification Commands
On CORE-SW:
```
!--- Confirm ip routing is active and all SVIs have correct IPs
CORE-SW# show ip interface brief

!--- Expected output:
! Interface         IP-Address    OK? Method  Status    Protocol
! Vlan10            10.10.10.1    YES manual  up        up
! Vlan20            10.10.20.1    YES manual  up        up
! Vlan30            10.10.30.1    YES manual  up        up
! Vlan99            10.10.99.1    YES manual  up        up
! GigabitEthernet0/1 203.0.113.2  YES manual  up        up
! GigabitEthernet0/2 unassigned   YES unset   up        up
! GigabitEthernet0/3 unassigned   YES unset   up        up

!--- Confirm routing table has all four connected subnets
CORE-SW# show ip route

!--- Expected output (excerpt):
! Gateway of last resort is 203.0.113.1 to network 0.0.0.0
! S*   0.0.0.0/0 [1/0] via 203.0.113.1
! C    10.10.10.0/24 is directly connected, Vlan10
! C    10.10.20.0/24 is directly connected, Vlan20
! C    10.10.30.0/24 is directly connected, Vlan30
! C    10.10.99.0/24 is directly connected, Vlan99

!--- If ip routing is NOT enabled, show ip route shows nothing useful
!--- and there are no 'C' entries for VLAN subnets

!--- Confirm trunk status on both uplinks
CORE-SW# show interfaces trunk

!--- Expected output:
! Port      Mode  Encapsulation  Status    Native vlan
! Gi0/2     on    802.1q         trunking  999
! Gi0/3     on    802.1q         trunking  999
!
! Port      Vlans allowed on trunk
! Gi0/2     10,20,99
! Gi0/3     30,99
!
! Port      Vlans allowed and active in management domain
! Gi0/2     10,20,99
! Gi0/3     30,99

!--- Verify the VLAN database
CORE-SW# show vlan brief

!--- Inspect a specific SVI in detail
CORE-SW# show interfaces vlan 10
CORE-SW# show interfaces vlan 30
```
Ping Test Sequence:
```
!--- From PC-HR1 (10.10.10.10):

Step 1 - Ping VLAN 10 SVI (own gateway):
> ping 10.10.10.1         Expected: SUCCESS  (L2 within VLAN 10 to CORE-SW SVI)

Step 2 - Ping same VLAN host:
> ping 10.10.10.11        Expected: SUCCESS  (pure L2 switching, no routing)

Step 3 - Ping IT gateway (VLAN 20 SVI):
> ping 10.10.20.1         Expected: SUCCESS  (HR to IT - L3 routing on CORE-SW)

Step 4 - Ping IT host:
> ping 10.10.20.10        Expected: SUCCESS  (end-to-end VLAN 10 → VLAN 20)

Step 5 - Ping Sales host (different floor, different switch):
> ping 10.10.30.10        Expected: SUCCESS  (VLAN 10 → VLAN 30 via CORE-SW)

Step 6 - Ping Sales3 (third Sales PC, different IP):
> ping 10.10.30.12        Expected: SUCCESS

!--- From PC-Sales1 (10.10.30.10):
> ping 10.10.10.10        Expected: SUCCESS  (Sales Floor 2 → HR Floor 1)
> ping 10.10.20.11        Expected: SUCCESS  (Sales → IT)
> ping 10.10.99.1         Expected: SUCCESS  (Sales → Management SVI)
> ping 10.10.99.11        Expected: SUCCESS  (Sales → SW-A management IP)
> ping 10.10.99.12        Expected: SUCCESS  (Sales → SW-B management IP)

!--- Management reachability test:
> ping 10.10.99.11        (reach SW-A from any subnet)  Expected: SUCCESS
> ping 10.10.99.12        (reach SW-B from any subnet)  Expected: SUCCESS
```
### 7. Troubleshooting Section
#### Problem 1 - SVIs are configured with IPs but inter-VLAN routing still fails completely
Symptom: show ip interface brief shows Vlan10, Vlan20, Vlan30 all up/up with correct IPs. But no PC in any VLAN can reach any PC in another VLAN. Same-VLAN pings work fine.\
Root cause: ip routing was never entered on CORE-SW. This is the #1 mistake in SVI labs. Without it, CORE-SW is just a Layer 2 switch that happens to have IP addresses on its VLAN interfaces - it will not route between them.\
Fix:
```
CORE-SW# show ip route
! If output shows nothing or just "Default gateway is not set" - ip routing is off

CORE-SW(config)# ip routing

! Immediately re-test:
CORE-SW# show ip route
! Should now show C (connected) routes for all VLAN subnets
```
#### Problem 2 - SVI shows down/down despite correct configuration
Symptom: show ip interface brief shows Vlan30   10.10.30.1   YES manual  down  down\
Root cause: An SVI comes up only when at least one physical port in that VLAN is active (up/up). If every port assigned to VLAN 30 is either shutdown, disconnected, or not yet assigned to VLAN 30, the SVI stays down. This is different from a router interface which can be up independently.\
Fix - diagnose which ports are in VLAN 30:
```
CORE-SW# show vlan id 30
! Check which ports are listed - if none, no access ports are in this VLAN

SW-B# show interfaces FastEthernet0/1 switchport
! Check: Access Mode VLAN = 30
! Check: Administrative Mode = static access

! If ports are assigned but still down, check physical layer:
SW-B# show interfaces FastEthernet0/1
! Look for: FastEthernet0/1 is administratively down
! If so:
SW-B(config)# interface FastEthernet0/1
SW-B(config-if)# no shutdown
```
#### Problem 3 - PC-Sales1 can reach CORE-SW but cannot reach HR or IT hosts
Symptom: ping 10.10.30.1 succeeds (Sales SVI reachable). ping 10.10.10.10 fails (HR host unreachable).\
Root cause A: VLAN 10 is not in the allowed VLAN list on the trunk between CORE-SW and SW-A, so CORE-SW can route the packet to VLAN 10 but cannot forward it to SW-A.\
Root cause B: VLAN 10 does not exist in SW-A's VLAN database. CORE-SW forwards the frame out Gi0/2 tagged VLAN 10, but SW-A has no record of VLAN 10 and drops it.\
Fix:
```
!--- Check trunk allowed VLANs:
CORE-SW# show interfaces GigabitEthernet0/2 trunk
! If VLAN 10 is missing from "Vlans allowed on trunk":
CORE-SW(config)# interface GigabitEthernet0/2
CORE-SW(config-if)# switchport trunk allowed vlan add 10

!--- Check VLAN database on SW-A:
SW-A# show vlan brief
! If VLAN 10 is not listed:
SW-A(config)# vlan 10
SW-A(config-vlan)# name HR
SW-A(config-vlan)# exit
```
#### Problem 4 - SW-A management SVI (VLAN 99) is up but the switch cannot be pinged from another subnet
Symptom: From CORE-SW, ping 10.10.99.11 fails. SW-A's SVI is up with correct IP. VLAN 99 is allowed on the trunk.\
Root cause: ip default-gateway was not configured on SW-A. SW-A has an IP (10.10.99.11) and can receive pings, but when it tries to reply, it has no idea where to send packets destined for 10.10.10.x or any other subnet outside its own. It drops the reply packet. The ping appears as request-timeout even though SW-A received it.\
Fix:
```
SW-A# show ip route
! On an L2 switch with ip routing disabled, this shows:
! "Default gateway is not set"  ← this is the problem

SW-A(config)# ip default-gateway 10.10.99.1
! Verify:
SW-A# show ip route
! Should now show: "Default gateway is 10.10.99.1"
```
#### Problem 5 - Trunk shows correct allowed VLANs but "VLANs in spanning tree forwarding state" is missing some VLANs
Symptom: show interfaces trunk on CORE-SW shows:
```
Vlans allowed on trunk: 10,20,99
Vlans allowed and active in management domain: 10,20,99
Vlans in spanning tree forwarding state and not pruned: 10,99
```
VLAN 20 is missing from the last line - traffic for VLAN 20 is blocked.
Root cause: Spanning Tree Protocol (STP) has put the port in a blocking state for VLAN 20. This can happen if there's a loop, or if the port recently came up and STP hasn't converged yet (takes up to 50 seconds in classic STP). On access switches in a single-switch environment, you can enable PortFast on the trunk to speed convergence - though this is only appropriate on edge ports in production.\
Fix:
```
!--- Wait 30-50 seconds and re-check (STP convergence delay)
CORE-SW# show spanning-tree vlan 20

!--- If the port is in BLK state, investigate why STP blocked it
!--- In a lab with no loops, you can speed up STP with:
SW-A(config)# spanning-tree mode rapid-pvst
CORE-SW(config)# spanning-tree mode rapid-pvst
!--- RSTP (Rapid PVST+) converges in ~1-2 seconds vs 30-50 for classic STP
```
### 8. Bonus - Exam Trick Questions
#### Exam Trick Question 1:

A student configures CORE-SW with SVIs for VLAN 10, 20, and 30. They run show ip interface brief and all three SVIs show up/up. They run show ip route and see all three connected routes. But every inter-VLAN ping still fails. The student has already verified PCs have correct IPs and gateways. What is the exact command that is missing and why does the routing table sometimes show connected routes even without it?
Answer: The missing command is ip routing. On some IOS versions and in Packet Tracer, SVIs that are up/up will create connected route entries in the routing table even without ip routing enabled, this is a misleading quirk that tricks many students. The routing table entry exists for the interface's own subnet, but the actual L3 forwarding engine (the routing process that decides to forward packets between different VLANs) is disabled. The switch will ARP and respond on each SVI independently but will not forward packets between VLANs. The giveaway: show ip route without ip routing shows routes but there is no "Gateway of last resort" line and no static or dynamic routes — only the local interface entries. After ip routing, the full routing engine activates.

#### Exam Trick Question 2:

In this lab, VLAN 30 exists on SW-B but NOT on CORE-SW's VLAN database — however the trunk is configured and the SVI for VLAN 30 is configured on CORE-SW. Will VLAN 30 traffic pass between SW-B and CORE-SW? Will the SVI be up?
Answer: This is a subtle and frequently tested scenario. The SVI (interface vlan 30) can be configured with an IP address regardless of whether VLAN 30 exists in the VLAN database. However, the SVI will be down/down if VLAN 30 is not in the VLAN database, because a VLAN that doesn't exist in the database has no ports and no logical existence on the switch. Additionally, while the trunk between CORE-SW and SW-B is 802.1Q which is standards-based, a Cisco switch will only pass VLAN tagged frames for VLANs that exist in its local VLAN database. If VLAN 30 doesn't exist in CORE-SW's VLAN database, it is pruned from the trunk automatically regardless of the allowed list. The fix is always: vlan 30 → name Sales on CORE-SW. The lesson: the allowed VLAN list on a trunk and the VLAN database are two independent requirements — both must be satisfied for a VLAN to pass traffic.
