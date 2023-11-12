# CiscoPTProject
Based off my University Assignment, some host names have been changed to keep confidentiality.

As per the task assigned, the following topology is configured within Packet Tracer
<h1> Basic Topology Setup</h1>

![Base Topology](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/e5fa48dc-69b9-4327-b2c2-3dc3344f9707)

The following addressing table shows all IP's/ subnet masks and default gateways needed to configure this. Note that 172.17.16.0 was given and VLSM was conducted in order to calculate the network's usable subnets. The two public IP's given by our "ISP" are 209.165.200.177 and 209.165.200.181 are used for both ISP connections.

![Addressing Table](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/96d25b24-ae47-44de-b241-0d60d04f3308)

In order to setup up the serial port's for the router's, we need to install a Network Interface Module (NIM) and in this case, NIM-2T through the router's virtual GUI. 

![DCE Port S1](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/b8f16db0-ebe9-4bd6-806f-01959d606668)

We then drag the NIM-2T to the corresponding NIM slot of the router

![DCE Port 2](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/b778565c-d193-4fa5-9424-398050c0cbf7)

This is repeated on all the router's that require a DCE connection to their serial ports.

<h2>IP Configuration of Routers, PC's and Switch SVI's</h2>

All the IP addresses needed are calculated as per the addressing table, and it will be constantly referred back to.

To perform the simple IP configurations first, we can configure the end point users (PC's) first.

![PC IP Configuration](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/48db7c54-bd01-4e1f-b0a7-700457a38306)

After accessing the desktop on the PC's GUI, we can click on IP Configuration to change the PC's IP configs.

![Management PC Config 1](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/eb4ef4c3-ab10-4ef5-a523-626f33e25d12)

As per the addressing table, we can configure our PC (in this case Admin) to:
```
IPv4 Address: 172.17.180.197
Subnet Mask: 255.255.255.248
Gateway: 172.17.180.193
```

![Management PC Config 2](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/73788de2-a04e-4b29-b6c7-796d2815a8c2)

We can repeat the same steps for the remaining PC and Servers.

![Labelled PC](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/53820514-132b-4600-8d72-b8e9c54318e2)

```
Staff PC:
IPv4: 172.17.180.3  
Subnet Mask: 255.255.255.128
Gateway: 172.17.180.1

Server:
IPv4: 172.17.180.130
Subnet Mask: 255.255.255.192
Gateway: 172.17.180.129

User Host:
IPv4: 172.17.186.2
Subnet Mask: 255.255.255.0
Gateway: 172.17.186.1
```
[Now we can move onto configuring the Switches](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/SwitchCLI)
