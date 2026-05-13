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
