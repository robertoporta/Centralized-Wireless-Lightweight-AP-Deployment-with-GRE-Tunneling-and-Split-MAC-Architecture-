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
- Address automation at the Fairfax branch is handled with a local DHCP pool on R2. I implemented DHCP Option 43 using the direct IP of the Sterling WLC (172.16.1.10) to allow remote registration across the GRE tunnel. Because the Fairfax WLAN is configured for FlexConnect Local Switching, R2 handles DHCP requests locally for wireless clients. As with the Sterling configuration, a DNS server was included at 20.20.20.20 for architectural completeness.
</p>

<p>
<img width="750" height="163" alt="image" src="https://github.com/user-attachments/assets/97180ca5-3746-41e1-92e6-0673ae73987f" />
</p>
<br>

<p>
- To complete the Fairfax infrastructure, I configured SW2. I initialized the VLAN database by creating and naming VLAN 30 for Fairfax Wireless. Unlike SW1, SW2 only requires a single VLAN because the Fairfax site does not host a centralized WLC or local server segment, relying instead on the management VLAN for AP connectivity and client bridging.  Creating this VLAN is necessary for the switch to correctly identify and process traffic coming from AP2; without this, the switch would not understand the VLAN 30 tag and would fail to pass the traffic. I used the “interface range” command for G0/1 and F0/1 to configure them as 802.1Q trunks. Then specifically assigned VLAN 30 as the native VLAN on F0/1 to ensure management traffic to the Access Point remains untagged for easier discovery and communication, while still allowing the port to carry tagged wireless client traffic.
</p>

<p>
<img width="777" height="302" alt="image" src="https://github.com/user-attachments/assets/3cdd946d-39d4-4902-b35f-b081f799d3ce" />
</p>
<br>

<p>
- Before beginning the wireless configuration, I connected the Lightweight APs to DC power, causing their link lights to activate. Now that both routers are now fully set up, the tunnels are operational, and the APs are powered, this is an updated view of the topology, with added subnet labels and all links functional. While the physical traffic still flows through the ISP, we have created a GRE Tunnel which functions as a direct virtual link between the two sites. This tunnel consists of two virtual interfaces (Tunnel0 on both ends) that operate within their own private subnet: 192.168.1.0/30. By assigning 192.168.1.1 to the Sterling end and 192.168.1.2 to the Fairfax end, we have effectively placed both routers in the same room logically. This "virtual wire" provides a safe, encapsulated path for our internal OSPF routing and WLC management traffic to travel, bypassing the complexities of the public WAN.
  
Additionally, the topology now illustrates the CAPWAP Tunnels established between the WLC and the Access Points. CAPWAP (Control and Provisioning of Wireless Access Points) is a dual-purpose protocol that creates two distinct logical channels. The Control Tunnel is used for the WLC to manage AP configurations, push firmware updates, and monitor AP health. The Data Tunnel remains essential for initial authentication and certain management traffic, even though we are using FlexConnect for local data switching. The orange lines in the topology represent these logical overlay connections, illustrating how the two branches and their wireless infrastructure are now a single unified network despite the geographic distance.
</p>

<p>
<img width="778" height="337" alt="image" src="https://github.com/user-attachments/assets/8cdd9843-5f2e-4003-beaa-bc46c9e8236c" />
</p>
<br>

<p>
- To begin the wireless configuration, from the Management PC's web browser I use HTTPS to connect to the WLC's GUI at https://172.16.1.10. I use the username of “admin” and a password of “123C!sco” to log in.  This step is critical for accessing the WLC's split-MAC management functions. 
</p>

<p>
<img width="776" height="601" alt="image" src="https://github.com/user-attachments/assets/e104cff0-e182-4918-a5bb-b0eb9f0c7e09" />
</p>
<p>
<img width="775" height="509" alt="image" src="https://github.com/user-attachments/assets/e86c2d7d-7d76-497f-ad09-6dacd99f8f73" />
</p>
<br>

<p>
- Once logged in, I am presented with the WLC Monitor Summary dashboard. Here, I can see a comprehensive overview of the system, including the software version, memory usage, fan status, and can see at the top that this particular WLC can support up to 150 Access Points. The graphical representation of the controller ports confirms that port 1 is active (indicated by the green port light), showing it is correctly connected to the Sterling network fabric.
  
I also confirmed that the controller is active and connected to the Sterling fabric. In the Management interface settings, I verified the gateway is set to 172.16.1.1. I ensured the VLAN Identifier was set to 0, utilizing the switch's native VLAN 30 mapping. This allows the WLC to reach the Fairfax branch via the GRE tunnel to manage AP2. 

</p>

<p>
<img width="776" height="504" alt="image" src="https://github.com/user-attachments/assets/50aaa4f1-d89a-4bbb-9177-a0a70b3f5b66" />
</p>
<p>
<img width="777" height="520" alt="image" src="https://github.com/user-attachments/assets/fc371738-9a78-4aa0-bebf-c6d2581252e5" />
</p>
<br>

<p>
- To bridge wireless users to the wired network, I created a Dynamic Interface named "Sterling-Interface." I assigned it VLAN Identifier 10 to match the router's sub-interface. I configured the interface with an IP of 10.0.1.11 and pointed the gateway and Primary DHCP Server to 10.0.1.1. This ensures that the WLC is logically present within the user subnet to manage traffic flow and proxy DHCP requests to the R1 router gateway. This configuration, combined with the trunking on SW1 g0/2, allows the WLC to successfully bridge the physical and virtual, effectively eliminating APIPA issues for Sterling users by providing a clear path to the local DHCP server.
</p>

<p>
<img width="777" height="147" alt="image" src="https://github.com/user-attachments/assets/74015747-dc1e-4625-8fbc-2c84e036131b" />
</p>
<p>
<img width="752" height="617" alt="image" src="https://github.com/user-attachments/assets/d31cbbc0-7e59-4650-afe2-3657d9deff96" />
</p>
<br>

<p>
- I then created the specific WLAN profile for the Sterling location. On the General Tab, I defined the Profile Name and SSID as “Sterling-HQ”. After I applied the initial settings, I ensure that I check the “Enabled” status box to bring it online and selected the Sterling-Interface created in the previous step as the interface gateway. Using the Sterling-Interface binds this WLAN specifically to VLAN 10, ensuring that users land in the correct broadcast domain and use the local Sterling gateway rather than saturating the management channel.
</p>

<p>
<img width="780" height="138" alt="image" src="https://github.com/user-attachments/assets/9cbbfe8a-2763-4556-981c-3476af453fe4" />
</p>
<p>
<img width="782" height="374" alt="image" src="https://github.com/user-attachments/assets/ac68564f-f62c-4472-af92-286c491893b0" />
</p>
<br>

<p>
- 
</p>

<p>

</p>
<br>
