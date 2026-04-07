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

## LAB 2 — Inter-VLAN Routing via Layer 3 Switch (SVI)
### 1. Scenario Description
Company: BetaTech Solutions — a larger company with 80+ employees across two floors. The IT team has upgraded the network infrastructure with a Layer 3 core switch. The goal is to eliminate the external router as the single point of failure for inter-VLAN routing and move that function inside the switch fabric itself using SVIs. A dedicated management VLAN keeps switch management traffic completely separate from user traffic.\
Note: This lab uses the `10.10.x.x` addressing scheme deliberately different from Lab 1, so you practice reading and not confusing subnet assignments.
| VLAN ID | Name |Purpose | Subnet | Gateway | Notes |
| ------- | ---- | ------ | ------ | ------- | ------------ |
| 10 | HR | Human Resources PCs | 10.10.10.0/24 | 10.10.10.1 | SVI on CORE-SW |
| 20 | IT | IT department PCs and servers | 10.10.20.0/2 | 10.10.20.1 | SVI on CORE-SW |
| 30 | Sales | Sales team PCs | 10.10.30.0/24 | 10.10.30.1 | SVI on CORE-SW |
| 99 | Management | Switch management | 10.10.99.0/24 | 10.10.99.1 | Out-of-band mgmt |
| 999 | NATIVE | Unused native VLAN | - | - | Security |



