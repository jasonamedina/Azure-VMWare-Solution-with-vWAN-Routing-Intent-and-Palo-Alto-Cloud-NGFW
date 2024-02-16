# vWAN-Routing-Intent-with-Palo-Alto-SaaS

In this article, you will learn how Virtual WAN with Routing Intent works with the Palo Alto SaaS firewall to connect and route traffic between different networks. You will see how this applies to an Azure VMware Solution private cloud, on-premises sites, and Azure native networks. This article does not cover how to set up or configure Virtual WAN with Routing Intent and Palo Alto SaaS.

## Virtual WAN network scenario  
Virtual WAN with Routing Intent is only supported with Virtual WAN Standard SKU. Virtual WAN with Routing Intent provides the capability to send all Internet traffic and Private network traffic (RFC 1918 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) to a security solution like Azure Firewall, a third-party Network Virtual Appliance (NVA), or Palo Alto SaaS solution. In the scenario, we have a network topology that spans only a single region. There is one Virtual WAN with a single hub(Hub1) and the Hub has the Palo Alto SaaS Firewall deployed. Having a firewall deployed in the Hub is a technical prerequisite to Routing Intent. Virtual WAN Hub1 has Routing Intent enabled.    

The diagram shows the Virtual WAN design, which has three components: an Azure VMware Solution Private Cloud, an Azure Virtual Network, and an on-premises site. The Azure VMware Solution and the on-premises site use ExpressRoute to connect to the hub, while the Azure Virtual Network uses a VNet Peering. We will go into more detail about each component later in this document.
![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/03a0c0df-da7c-4394-9afc-4b7bf96434c4)


>[!NOTE]
>  When configuring Azure VMware Solution with Virtual WAN Hubs, ensure optimal routing results on the hub by setting the Hub Routing Preference option to "AS Path." - see [Virtual hub routing preference](https://learn.microsoft.com/azure/virtual-wan/about-virtual-hub-routing-preference)

### ExpressRoute Global Reach deployment options 

Global Reach establishes a direct logical link via the Microsoft backbone, connecting Azure VMware Solution to On-Premises. 

 **With Global Reach**  
A benefit of using Global Reach is that it makes the design simpler with a direct logical connection between Azure VMware Solution and On-Premises. It also helps troubleshoot traffic between Global Reach sites and removes the worry of throughput limitations at the Virtual WAN Hub level. 

When Global Reach is deployed, traffic between the Global Reach sites bypasses Virtual WAN Hub Firewall. This means the Virtual WAN Hub firewall will not inspect any Global Reach traffic that goes between the Azure VMware Solution and the On-Premises datacenter. 

> [!NOTE]
> When utilizing Global Reach, traffic between these locations bypasses the Virtual WAN and the Hub Firewall. To ensure optimal security, we recommend inspecting traffic within the Azure VMware Solution environment's NSX-T or using an on-premises firewall between these locations.
>
![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/a7add202-9c9d-48eb-a798-144f9655f421)

**Without Global Reach**  
When using Virtual WAN Routing-Intent without Global Reach, from an on-premises network, you cannot advertise the exact default RFC 1918 address prefixes (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) back to Azure. Instead, you must always advertise more specific routes.
In regions without Global Reach support or with a security requirement to inspect traffic between Azure VMware Solution and on-premises at the hub firewall, a support ticket must be opened to enable ExpressRoute to ExpressRoute transitivity. Once the support ticket has been fulfilled, the Virtual WAN Hub will advertise the default RFC 1918 addresses to Azure VMware Solution and to on-premises (as shown below in the diagram).  ExpressRoute to ExpressRoute transitivity isn't supported by default with Virtual WAN with Routing Intent. [Transit connectivity between ExpressRoute circuits with routing intent](https://learn.microsoft.com/en-us/azure/virtual-wan/how-to-routing-policies#expressroute)  

To demonstrate how the hub firewall can inspect traffic, this design does not use Global Reach
![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/29c3ed30-a692-47d4-a687-e516e5921ae0)



### Azure VMware Solution connectivity 

This section focuses on only the Azure VMware Solution private cloud. The Azure VMware Solution private cloud has an ExpressRoute connection to the hub (connections labeled as "E").


The diagram illustrates the Route Table as seen from the perspective of Azure VMware Solution.

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/96d13428-bb4f-4b10-b437-064d09607791)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/b9e59f9d-1117-4d5d-9c6c-da3abfbb9af1)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/104f7036-a10e-4971-8b9b-34a8c51358d6)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/064eb6c0-928d-47f5-9149-93fd8a6bb656)








### on-premises connectivity & traffic flow

This section focuses only on the on-premises site. As shown in the diagram, the on-premises site has an ExpressRoute connection to the hub (connection labeled as "F").

On-premises systems can communicate to Azure VMware Solution via connection "Global Reach (A)".

The diagram illustrates the Route Table as seen from the perspective of on-premises and Azure VMware Solution.

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/9a8a63f4-4ebb-422d-bf45-20b685f24862)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/dc186f01-6792-4fa8-9ead-6fb4bfc3d2ff)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/0a6b25a5-143e-4250-81e2-ec5d88ecd014)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/69ddbd71-dd64-4aa7-8add-17b5e2d8d7f2)








### Azure Virtual Network connectivity & traffic flow

This section focuses only on connectivity from an Azure Virtual Network perspective. As depicted in the diagram, the Virtual Network has a Virtual Network peering directly to the hub.

The diagram illustrates how all Azure native resources in the Virtual Network learn routes under their "Effective Routes". A Secure Hub with enabled Routing Intent always sends the default RFC 1918 addresses (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) to peered Virtual Networks, plus any other prefixes that have been added as "Private Traffic Prefixes" - see [Routing Intent Private Address Prefixes](/azure/virtual-wan/how-to-routing-policies#azurefirewall). In our scenario, with Routing Intent enabled, all resources in the Virtual Network currently possess the default RFC 1918 address and use the hub firewall as the next hop. All traffic ingressing and egressing the Virtual Networks will always transit the Hub Firewall. For more information, see the traffic flow section for more detailed information.

The diagram illustrates the Route Table as seen from the perspective of the Azure Virtual Network and Azure VMware Solution. 
![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/d019ef47-a378-44df-99d2-46c7ebf2dcd0)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/65c5cd43-709b-41d5-834b-a34dbf00c5fc)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/e60d01e7-9203-45b8-bf68-46b99b1824ab)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/8cc8b0f3-0d98-4b56-95b9-a23f7b6b156d)



### Internet connectivity

This section focuses only on how internet connectivity is provided for Azure native resources in Virtual Networks and Azure VMware Solution Private Clouds in a single region. There are several options to provide internet connectivity to Azure VMware Solution. - see [Internet Access Concepts for Azure VMware Solution](/azure/azure-VMware/concepts-design-public-internet-access)

Option 1: Internet Service hosted in Azure  
Option 2: VMware Solution Managed SNAT  
Option 3: Azure Public IPv4 address to NSX-T Data Center Edge  

Although you can use all three options with Single Region Secure Virtual WAN with Routing Intent,  "Option 1: Internet Service hosted in Azure" is the best option when using Secure Virtual WAN with Routing Intent and is the option that is used to provide internet connectivity in the scenario.  

As mentioned earlier, when you enable Routing Intent on the Secure Hub, it advertises RFC 1918 to all peered Virtual Networks. However, you can also advertise a default route 0.0.0.0/0 for internet connectivity to downstream resources. The default route is advertised via connection "E".

Virtual networks peered to the Hub will use the hub firewall to access the internet.  
 
Another important point is that with Routing Intent, you can choose to not advertise the default route over specific ExpressRoute connections. We recommend not to advertise the default route to your on-premises ExpressRoute connections. 

See traffic flow section more information.

The diagram illustrates the Route Table as seen from the perspective of Azure VMware Solution and the Azure Virtual Network.

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/1a1b9aba-ae1a-4653-bc86-64700d8b834a)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/eb69f1c8-70d2-42c9-bfb5-a23c262b40d2)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/7b46532a-256f-480b-b84e-4e3f5162dca7)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/7daddcf3-752f-4b8e-8ae5-54dd68aad879)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/cba68635-e783-4adc-8156-6c71324735a1)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/7fefba55-e4ff-4cdd-8098-a1923be6fa5e)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/395ca706-70d8-4571-aa64-2a8aa00a6537)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/d41113c3-e65a-460c-bffb-0d2063d1ae2e)

![image](https://github.com/jasonamedina/vWAN-Routing-Intent-with-Palo-Alto-SaaS/assets/97964083/de0e1555-0b24-4e26-98ce-38650df3b9ee)





