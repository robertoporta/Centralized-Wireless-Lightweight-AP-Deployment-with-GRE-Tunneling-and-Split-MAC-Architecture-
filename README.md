#

<h1>Centralized Wireless Lightweight AP Deployment with GRE Tunneling and Split-MAC Architecture </h1>



<p>
- This project demonstrates the implementation of a scalable, multi-site enterprise network bridging a central headquarters in Sterling with a remote branch in Fairfax. The primary objective is to establish secure, high-performance connectivity using a GRE (Generic Routing Encapsulation) tunnel to overlay private traffic across a simulated public WAN.
  
Throughout this lab, I will configure OSPF for dynamic routing and a Router-on-a-Stick design for VLAN segmentation. Central to the wireless strategy is a Unified WLC Deployment model utilizing a split-MAC architecture. By configuring the Lightweight Access Points (APs) in FlexConnect operating mode, I will implement Local Switching and Local Authentication secured by WPA2-PSK/AES encryption. This distributed data plane ensures that while management and control functions (CAPWAP Control Tunnels) remain centralized at the Sterling WLC, DHCP services and user data traffic are handled locally at the edge. This design provides a resilient wireless environment that preserves network access for branch users even if the CAPWAP data tunnels to the central hub are disrupted.

The image below shows the topology illustrating the interconnection of the two locations (Sterling HQ and Fairfax Office) through the Service Provider Network, including localized subnets for management and wireless infrastructure with their devices.
</p>

<p>
<img width="778" height="339" alt="image" src="https://github.com/user-attachments/assets/550b9750-8fa7-4518-8926-5781b12ae60c" />
</p>
<br>

<p>
- To begin configuration, the first stage of the Sterling HQ deployment involves initializing the physical hardware paths on the edge router. I configured g0/0 with a public-facing IP to establish the upstream link to the Service Provider. For the internal network, g0/1 was brought online to serve as the physical trunk for all local traffic. At this stage, the internal interface is enabled but left without a primary IP address, as it will be subdivided into logical sub-interfaces to handle the segmented VLAN traffic. 
</p>

<p>
<img width="773" height="298" alt="image" src="https://github.com/user-attachments/assets/0d966953-42df-4ea2-a7ce-6e5c7330c484" />
</p>
<br>

<p>
- To support a robust multi-VLAN environment, I implemented a Router-on-a-Stick architecture using 802.1Q encapsulation. I created sub-interface .10 for the Sterling Data segment and sub-interface .30 to handle the WLC Management subnet. By assigning the first usable IP of each respective range to these sub-interfaces, they act as the default gateways for their specific broadcast domains, ensuring clear logical separation and secure inter-VLAN routing. 
</p>

<p>
<img width="764" height="286" alt="image" src="https://github.com/user-attachments/assets/74f28c7a-accd-4766-be84-1c43491347b4" />
</p>
<br>

<p>
- To establish the primary communications hub in Sterling, I configured the edge router to support a virtual overlay. The GRE tunnel is a critical component that encapsulates internal traffic, allowing private 172.16.x.x and 10.x.x.x addresses to traverse the public ISP network. To ensure the Tunnel Line Protocol transitions to an "up" state, a default static route to the ISP gateway is required; this provides the router with the necessary reachability to resolve the tunnel destination (200.0.0.2) and activate the virtual interface. Additionally, I implemented a specific static route for the Fairfax network (10.0.2.0) pointing back through the tunnel. This is essential to ensure that return traffic from the WLC or Sterling clients can find the remote Fairfax branch, as the router must explicitly know to send Fairfax-bound packets into the GRE tunnel rather than out to the public internet. 
</p>

<p>
<img width="776" height="305" alt="image" src="https://github.com/user-attachments/assets/24b85f3a-b346-4f2b-9667-2abd17b6fe37" />
</p>
<br>

<p>
- To automate the exchange of routing information, I deployed OSPF on the Sterling router. I explicitly included the 172.16.1.0/24 subnet in the OSPF process; this is vital because it allows the Fairfax branch to "learn" the route to the management network through the GRE tunnel. I also advertised the 10.0.1.0 network to make Sterling's local resources visible to the remote site. The passive-interface command is used on user-facing LANs to enhance security by preventing OSPF updates from being broadcast to local workstations.
</p>

<p>
<img width="735" height="167" alt="image" src="https://github.com/user-attachments/assets/ff828f05-29c1-4442-ba90-6d8a7ab9a0e3" />
</p>
<br>

<p>
- To support a scalable environment, I configured multiple DHCP pools on R1 to act as the primary addressing engine for the Sterling site. I excluded the first 10 addresses from all subnets to reserve them for critical management hardware. Because we are using FlexConnect Local Switching, wireless clients at Sterling will communicate directly with R1 for their addresses. I implemented a dedicated pool for the Management segment and the 10.0.1.0/24 subnet for Sterling users. I have included a DNS server address in the configuration for completeness, though it will not be actively used for name resolution in this specific lab scope. To adhere to the management policy, I statically assigned the Management PC the IP 172.16.1.2/24, making the router ready to serve both static management terminals and dynamic wireless endpoints.
</p>

<p>
<img width="775" height="397" alt="image" src="https://github.com/user-attachments/assets/6b8dd9c1-4a34-4218-a79d-a74d34ba2f14" />
</p>
<p>
<img width="778" height="188" alt="image" src="https://github.com/user-attachments/assets/568f4ffa-eb27-46ba-8503-ebb406b16a57" />
</p>
<br>

<p>
- Before moving to R2, I configured SW1. I created and named VLAN 10 for Sterling_Wireless and VLAN 30 for Management. These VLANs must be created because without a local VLAN entry, the switch will not know how to forward frames that carry those specific 802.1Q tags. This step makes the switch be able to accept and route traffic for those segments. To support the FlexConnect deployment, I utilized the interface range command to simultaneously configure G0/1 (to the router), G0/2 (to the WLC), and F0/1 (to the AP) as 802.1Q trunks, enabling the transport of multiple VLANs over a single physical link. I also set VLAN 30 as the native VLAN to match the router's sub-interface configuration, allowing untagged management traffic to flow through the trunk without overhead. Lastly, Port F0/2 was assigned to VLAN 30 to support the Management PC using an access port, which strips VLAN tags for end-device compatibility.
</p>

<p>
<img width="771" height="461" alt="image" src="https://github.com/user-attachments/assets/8fb6bf85-a993-497f-8b21-00fef07ee5c7" />
</p>
<br>

<p>
- Now onto R2’s configuration. I initialized the physical connectivity for the Fairfax branch. I assigned a unique public IP to G0/0 and enabled the G0/1 interface. It is important to note that the interface G0/1 requires no primary IP address; instead, it serves as the physical carrier for a sub-interface. This leads into the configuration of G0/1.30, where I assigned the gateway for the 10.0.2.0/24 network. By using this sub-interface to handle VLAN 30 tagged traffic, I maintained a consistent logical structure across both sites. Note that unlike R1, R2 only requires a single sub-interface and pool, as the WLC and Management PC are hosted centrally at Sterling.
</p>

<p>
<img width="773" height="425" alt="image" src="https://github.com/user-attachments/assets/04c079aa-f6f5-4cfa-bcfe-9997d9187760" />
</p>
<br>

<p>
- With the Sterling hub active, I initialized the Fairfax branch router to complete the virtual circuit. This configuration mirrors the Sterling side, effectively bridging the two geographic locations over the simulated WAN. By mapping the tunnel destination back to Sterling’s public IP and applying the default static route, the Tunnel Line Protocol successfully activates. This setup is essential for maintaining a unified corporate network where resources in Sterling appear locally accessible to users in Fairfax.
</p>

<p>
<img width="776" height="302" alt="image" src="https://github.com/user-attachments/assets/61304c2c-34b9-497e-817e-0644790022a4" />
</p>
<br>

<p>
- The final step in establishing the routing fabric was to enable OSPF at the Fairfax branch. Once activated, the Fairfax router successfully formed a neighbor adjacency with Sterling across the GRE tunnel. This dynamic link ensures that Fairfax traffic destined for the 172.16.1.10 WLC is correctly routed through the tunnel.  Also, I made the G0/1.30 sub-interface a passive OSPF interface.
</p>

<p>
<img width="775" height="115" alt="image" src="https://github.com/user-attachments/assets/664e7d75-0239-4761-b458-a9bb1ce72503" />
</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>
