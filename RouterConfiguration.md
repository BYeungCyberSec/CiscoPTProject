[Order of Guide is: ReadMe -> SwitchCLI -> RouterConfig -> NATPATConfig](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/README.md)

<h1>Router Configuration</h1>

The routers will have slightly more complex configurations, but they are largely similar in concept as all we're are doing is encapsulating those VLAN's we just assigned between the switch and PC's, encapsulating them and giving them an IP address.

Setup the routers similiar like with the switches
```
en
conf t
Hostname West
no ip domain-lookup
interface Gig0/0/0.10 (the router's outgoing interface to the switch and .10 indicates the VLAN we've assigned to the switch)
encapsulation dot1Q 10 (press tab after dot to auto-complete to dot1Q)
ip add 172.17.176.1 255.255.252.0
```

After that series of commands, we have encapulsated the outgoing interface of the router to the switch, allowing for traffic through that specific VLAN10 and assigned them the IP address according to the addressing table.
We will need to repeat those steps for VLAN 20, 67 and 555 but keep in mind VLAN 555 will not have any IP address since it is our native VLAN

```
interface Gig0/0/0.20
encapsulation dot1Q 20
ip add 172.17.180.1 255.255.255.128
interface Gig0/0/0.67
encapsulation dot1Q 67
ip add 172.17.180.193 255.255.255.248
interface Gig0/0/0.555
encapsulation1 dot1Q 555 native
```
If the outgoing interface of the router has red arrows signalling the interface is down like in the image
![Interface down](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/d6b57c9b-b99b-4b62-aab2-7dfb38e52676)

We can force them into the upstate in the router's configuration mode using 
```
interface gig0/0/0
no sh
```
There will be green arrows outgoing from the switch if this is enabled properly
![Interface up](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/1cf0524f-eb97-4190-b782-ddbe6328d1cc)

We also need to configure our router's serial connection (West in this case) connecting to Main
```
interface Se0/1/1
ip add 172.17.180.202 255.255.255.252
no sh
```
Keep in mind, the serial port interface may be different on how you have connected it, check the connection and change the interface if needed.
Make sure all the VLAN connections are configured properly by entering the router's enabled mode, using ```sh ip int``` and checking each Gig0/0/0.(VLANNumber) is configured properly with an IP address.

Repeat the same steps for all the routers, configuring them in the same manner and taking note of the outgoing interfaces as per the addressing table

![Addressing Table](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/8ad81733-235c-4110-8736-2d130a5d00f6)

As you can see in the addressing table, we need to also enable the Loopback for our ISP router (this will act as Internet Access within this home lab)
To achieve this, make sure the ISP router is in configuration mode already 
```
interface Loopback1
ip add 1.1.1.1 255.255.255.255
no sh
```
<h1>Static Routing and Open Shortest Path First (OSPF)</h1>

The original documentation requires the main network to have OSPF active whilst East-Branch is statically connected such that traffic can reach 172.17.187.0 network.

When OSPF and static routing overlap, we have to be very careful how we advertise the static routing and/or implement our static routing. If done incorrectly, the traffic can be routed through an infinite loop of static routes and lead to a packet drop and no network connectivity.
The method I have used is I static route from East ``` 0.0.0.0 0.0.0.0``` through to Branch but I specify the traffic coming from Branch to East

Inside of East's router configuration mode
``` 
ip route 0.0.0.0 0.0.0.0 172.17.187.6
```

For Branch's router
```
ip route 172.17.180.0 255.255.255.252 172.17.185.5
ip route 172.17.180.192 255.255.255.248 172.17.187.5
ip route 172.17.180.128 255.255.255.192 172.17.187.5
ip route 0.0.0.0 0.0.0.0 209.165.200.181
```
To breakdown the static route's for Branch:
Each VLAN (10,20,67) is static routed individually to avoid an infinite loop when OSPF is implemented.
The static route of "0.0.0.0 0.0.0.0 209.165.200.181" is implemented as a fail-safe in the event that ISP-Main is down.

Now we need to configure the static routes for ISP and Main

Inside of ISP:
```
ip route 209.165.200.188 255.255.255.252 209.165.200.182 10
ip route 209.165.200.188 255.255.255.252 209.165.200.178 5
```
Here we purposely set the adminstrative distance (AD) of Main lower than Branch so that it will always direct traffic towards Main unless Main-ISP is unavailable

Inside of Main:

```
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/1 2
ip route 172.17.186.0 255.255.255.0 Serial0/1/1 
ip route 0.0.0.0 0.0.0.0 Serial0/1/1 3
```
We need to set the AD here for Main-ISP lower than Main-East otherwise the OSPF advertised will direct outgoing traffic from Main-East and cause traffic to exit through Branch-ISP. We also exclusively set traffic for 172.17.186.0 network to go through Main-East as this will cause less strain on the rest of the network directing this traffic.

To setup OSPF, we must advertise that router's outgoing interface (each VLAN is counted as its indivudal interface, so in this case, all 3 VLAN's need to be advertised seperately) with its wildcard mask (inverse of subnet mask)
As an example, inside of Main Router:
```
router ospf 1
 log-adjacency-changes
 network 172.17.180.0 0.0.0.63 area 0
 network 172.17.180.204 0.0.0.3 area 0
 network 172.17.180.200 0.0.0.3 area 0
 network 172.17.180.128 0.0.0.63 area 0
 default-information originate
```
We only want ```default-information originate``` on this router otherwise it will advertise static routes from the other routers back to Main, causing an infinite loop and causing traffic to be dropped.

In West:
```
router ospf 1
 network 172.17.180.200 0.0.0.3 area 0
 network 172.17.180.0 0.0.0.127 area 0
 network 172.17.180.192 0.0.0.7 area 0
 network 172.17.180.128 0.0.0.63 area 0
```
In East:
```
network 172.17.187.0 0.0.0.3 area 0
network 172.17.180.192 0.0.0.7 area 0
network 172.17.180.204 0.0.0.3 area 0
network 172.17.180.128 0.0.0.63 area 0
network 172.17.180.0 0.0.0.127 area 0
```

![Sh IP Route West](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/bf8eb15c-34b3-4f0b-91fe-19a8fc45ddcb)

When OSPF is configured properly on the Main network Routers (Main+West+East), you can use ```sh ip route``` to identify the new advertised routes through OSPF with an "O" next to the route.

Our last part of this project is NAT and PAT assignment, which is a very short and quick configuration [in this section](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/NATPATConfig.md)


