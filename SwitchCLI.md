[Order of Guide is: ReadMe -> SwitchCLI -> RouterConfig -> NATPATConfig](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/README.md)

<h1>Switch Configuration</h1>

We can access the switch's GUI by double clicking on them in the topology, and we can configure them through the command-line interface (CLI)

![Switch GUI](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/9f311644-b6fd-4ae4-be8d-215c369648ad)

We want to take note of the ports we have connected from the PC Hosts to the switch (In this case it is Fa0/1 and Fa0/11), we will need to configure off those ports 
We will configure the Main Network Switch first.
To enable the switch to configuration mode, we use the following commands

```
en (short for enable)
conf t (short for configure terminal)
hostname MainNetworkSwitch
no ip domain-lookup
```

Taking note of those ports earlier is important, as we need to setup trunking for those ports as we can see from the topology, it is a router-on-a-stick topology (PC connected to switch connected to router)

We can use the following commands to trunk those interfaces. Our initial documentation requires us to have VLANS 10,20,67 to be active for the Users, Staff and Management respectively and also requires us to use something other than VLAN1 as the default VLAN (in this case we'll be assigning VLAN555)
The commands needed are as followed:

```
interface fa0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,67,555
switchport trunk native vlan 555
exit
interface fa0/11
switchport mode trunk
switchport trunk allowed vlan 10,20,67,555
switchport trunk native vlan 555
exit
```

These series of commands tells the switch that these interfaces are for trunking, which allows them to communicate to another network (in this case a series of routers that they are connected to)

We also want to configure the switchport's SVI as we can see in the addressing table. To achieve this, we need to configure VLAN67 directly by setting the default gateway and assigning IP and subnet mask.

```
interface vlan67
ip add 172.17.180.195 255.255.255.248
ip default-gateway 172.17.180.193 
```

We can check if our switch's ports are configured properly by checking their status in the switch's enabled mode (when the CLI shows MainNetworkSwitch#)
 ```
show interface trunk
```
Here we can see our assigned VLAN's on the port's interface and that their status is in trunking mode.
![Sh Int trunk](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/22110a6f-d17c-42b0-8d71-35e2850ff359)

We can configure using similiar steps and commands for the Branch's Network Switch

```
en
conf t
Hostname BranchNetworkSwitch
no ip domain-lookup
Interface Fa0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,67,555
switchport trunk native vlan 555
exit
interface vlan 67
ip address 172.17.187.2 255.255.255.252
ip default-gateway 172.17.187.1
```

Double-check the switch is configured properly using ```sh int trunk``` in enabled mode and we can proceed to configuration of [Configuration of Routers](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/RouterConfiguration.md)
