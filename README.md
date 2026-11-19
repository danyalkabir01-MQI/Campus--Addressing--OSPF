# Campus--Addressing--OSPF
Campus Addressing Scheme and OSPF Area Allocation - Ezitech Task 1
# Campus Network - Bright Future School


## 1. My Base Network
I got one block from my task: 10.20.0.0/22
Total IPs in this block: 1024

I used VLSM method. It means biggest requirement first, then small.

## 2. My IP Table - Simple

| Building | VLAN | Subnet | For How Many People? |
| :--- | :--- | :--- | :--- |
| Main Building | VLAN 10 - Students | 10.20.0.0 /24 | 200 Students |
| Hostel | VLAN 60 - Hostel A | 10.20.1.0 /25 | 100 Students |
| Main Building | VLAN 20 - Teachers | 10.20.1.128 /25 | 100 Teachers |
| Hostel | VLAN 70 - Hostel B | 10.20.2.0 /25 | 80 Students |
| Main Building | VLAN 30 - Lab | 10.20.2.128 /26 | 60 Computers |
| Admin Block | VLAN 40 - Office | 10.20.2.192 /26 | 50 Staff |
| Admin Block | VLAN 50 - Accounts | 10.20.3.0 /27 | 20 Staff |
| Hostel | VLAN 80 - Guest WiFi | 10.20.3.32 /27 | 20 Guests |

## 3. My OSPF Design

I made 3 Areas. All areas connect to Area 0. This is the rule of OSPF.

| Router | Building | Area | Router ID |
| :--- | :--- | :--- | :--- |
| R1 | Main Building | Area 0 | 1.1.1.1 |
| R2 | Admin Block | Area 10 | 2.2.2.2 |
| R3 | Hostel | Area 20 | 3.3.3.3 |

- Area 0 = Main Building, this is the backbone
- Area 10 = Admin Block
- Area 20 = Hostel

I used Loopback for Router ID because Loopback never goes down.

## 4. My Simple Diagram

      Main Building - R1 - Area 0
             /      \
            /        \
   Admin - R2      Hostel - R3
   Area 10         Area 20

## 5. Router Config - Poora

### R1 - Main Building - Area 0
router ospf 1
 router-id 1.1.1.1
 network 10.20.0.0 0.0.0.255 area 0
 network 10.20.1.128 0.0.0.127 area 0

### R2 - Admin Block - Area 10
router ospf 1
 router-id 2.2.2.2
 network 10.20.2.192 0.0.0.63 area 10
 network 10.20.3.0 0.0.0.31 area 10

### R3 - Hostel - Area 20
router ospf 1
 router-id 3.3.3.3
 network 10.20.1.0 0.0.0.127 area 20
 network 10.20.3.32 0.0.0.31 area 20

## 6. How to Check?
- show ip ospf neighbor
- ping 10.20.0.1

## 7. OSPF Area Design + VLSM Calculations

### Area 0 - Main Building (R1)
- Students: Need 200 hosts -> Need 256 addresses -> /24
  Network: 10.20.0.0/24, Usable: 10.20.0.1 - 10.20.0.254, Broadcast: 10.20.0.255
- Teachers: Need 100 hosts -> Need 128 addresses -> /25
  Network: 10.20.1.128/25, Usable: 10.20.1.129 - 10.20.1.254, Broadcast: 10.20.1.255

### Area 10 - Admin Block (R2)
- Office: Need 50 hosts -> Need 64 addresses -> /26
  Network: 10.20.2.192/26, Usable: 10.20.2.193 - 10.20.2.254, Broadcast: 10.20.2.255
- Accounts: Need 20 hosts -> Need 32 addresses -> /27
  Network: 10.20.3.0/27, Usable: 10.20.3.1 - 10.20.3.30, Broadcast: 10.20.3.31

### Area 20 - Hostel (R3)
- Hostel: Need 100 hosts -> Need 128 addresses -> /25
  Network: 10.20.1.0/25, Usable: 10.20.1.1 - 10.20.1.126, Broadcast: 10.20.1.127
- Guest WiFi: Need 20 hosts -> Need 32 addresses -> /27
  Network: 10.20.3.32/27, Usable: 10.20.3.33 - 10.20.3.62, Broadcast: 10.20.3.63

## 8. Point-to-Point Link Subnets - Router to Router

Routers ko jorne ke liye sirf 2 IP chahiye hote hain, isliye /30 use kiya hai.

| Link | Network | R1 / R2 / R3 IP | Mask |
| --- | --- | --- | --- |
| R1 - R2 | 10.20.3.64/30 | R1: 10.20.3.65, R2: 10.20.3.66 | 255.255.255.252 |
| R2 - R3 | 10.20.3.68/30 | R2: 10.20.3.69, R3: 10.20.3.70 | 255.255.255.252 |
| R1 - R3 | 10.20.3.72/30 | R1: 10.20.3.73, R3: 10.20.3.74 | 255.255.255.252 |


## VLSM Working - Area 10 Admin Block (On Paper Method)

Total Hosts Started From: 70 hosts (Office 50 + Accounts 20)

1. Office - Largest First
   Needed: 50 hosts
   Calculation: 50 + 2 (Network + Broadcast) = 52
   Next Power of 2: 64 = Block Size 64
   Prefix: /26 (32 - 6 = 26, because 2^6 = 64)
   Network: 10.20.2.192/26
   Usable: 10.20.2.193 - 10.20.2.254
   Broadcast: 10.20.2.255

2. Accounts - Next
   Needed: 20 hosts
   Calculation: 20 + 2 = 22
   Next Power of 2: 32 = Block Size 32
   Prefix: /27 (32 - 5 = 27, because 2^5 = 32)
   Network: 10.20.3.0/27
   Usable: 10.20.3.1 - 10.20.3.30
   Broadcast: 10.20.3.31