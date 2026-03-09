In this lab, I focused on troubleshooting OSPF. The main issue was that routes were not being advertised between the core router (E1) and the area routers (R1, R2, R3). To make it realistic, I created an IT ticket for myself to troubleshoot the problem. I will share the steps I took to identify and resolve the issue.

# Troubleshooting OSPF
Find the github repository for the lab files [here](https://github.com/dannebrob/CML---Enterprice-lab)

## Ticket ID: OSPF‑Troubleshooting‑INC0000456

OSPF Troubleshooting Ticket – E1 and R1
1. Summary
OSPF adjacency is not forming between E1 and router R1. Layer‑2 communication is functioning, but OSPF does not reach FULL state. There is suspicion of mismatched interface parameters, subnet masks, or area configuration.

2. Environment
Affected Devices
- E1 (router‑ID 1.1.1.1)
- R1 (router‑ID 10.10.10.10)
- R2 (router‑ID 20.20.20.20)
- R3 (router‑ID 30.30.30.30)
<br><br>
Links
- E1 ↔ R1: 10.20.0.0/30
- E1 ↔ R2: 10.30.0.0/30
- E1 ↔ R3: 10.40.0.0/30
<br><br>
 OSPF Areas
- E1–R1: Area 10
- E1–R2: Area 20
- E1–R3: Area 30
- E1 Loopback0: Area 0

3. Symptoms
- show ip ospf neighbor on E1 shows no neighbors.
- show ip ospf interface confirms OSPF is enabled on the correct interfaces on E1.
- R1 reports no OSPF neighbor on Ethernet0/0.
- Ping between E1 and R1 works, indicating Layer‑2 connectivity.
- The routing table contains no OSPF‑learned routes.

4. Expected Behavior
- E1 should establish adjacency with R1, R2, and R3 in their respective areas.
- show ip ospf neighbor should display neighbors in FULL state.
- OSPF routes should appear in the routing tables of all routers.


## The Troubleshooting Process

My first step was to verify the interface parameters on both E1 and R1. Luckily theres a lot of ways to troubleshoot OSPF issues. I ran the following commands: 
- `show ip interface Ethernet0/1` on E1 and `show ip interface Ethernet0/0` on R1 to check the IP address, subnet mask, and OSPF area configuration. 
- `show ip ospf neighbor` to check the neighbor status, since I got no neighbors, this was the first indication that there was an issue with the OSPF configuration. 
- `show ip ospf interface` to verify OSPF is enabled on the correct interfaces and to check the OSPF parameters.
- `show ip ospf database` to see if there were any OSPF LSAs being exchanged.
- `show ip interface brief` to check the status of the interfaces.

With this information, I found that the IP address on E1 was configured with a /30 subnet mask, while R1 was configured with a /24 subnet mask. This mismatch was likely the cause of the adjacency failure, as OSPF includes the subnet mask in its Hello packets, and a mismatch would prevent the routers from recognizing each other as neighbors.

After identifying the subnet mask mismatch, I updated the IP address configuration on R1 to match the /30 subnet mask used on E1. After making this change, I issued the `show ip ospf neighbor` command again, and this time I saw that E1 and R1 had successfully formed an OSPF adjacency and were in FULL state. The routing tables on both routers also started showing OSPF‑learned routes, confirming that the issue was resolved. 

Next, I confirmed Layer‑2 connectivity by pinging the neighbor IP addresses and checking the ARP tables, by running the `show arp` command. And I got the expected results. These two commands confirmed that there were no issues with physical connectivity or basic IP reachability between E1 and R1. This helped me narrow down the issue to the OSPF configuration rather than a physical or Layer‑2 problem.

Finally, I confirmed that OSPF was running on the correct interfaces by using the commands: `show ip protocols` and `show run | section ospf`. This helped me ensure that OSPF was properly configured on both E1 and R1. Once this was confirmed, I confirmed that ospf also worked as expected on R2 and R3.

## Conclusion
In conclusion, the OSPF adjacency issue between E1 and R1 was caused by a subnet mask mismatch. By verifying the interface parameters, OSPF configuration, and Layer‑2 connectivity, I was able to identify and resolve the issue. This troubleshooting process really shows the importance of attention to detail when configuring OSPF, as even a small mismatch in subnet masks can prevent routers from forming adjacencies and exchanging routing information. 

It also shows the value of using a structured approach to troubleshooting, starting with basic connectivity checks and then moving on to more specific OSPF configuration verification. 
By following these steps, I was able to successfully resolve the OSPF adjacency issue and restore proper routing between E1 and R1, as well as R2 and R3.
