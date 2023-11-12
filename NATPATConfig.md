[Order of Guide is: ReadMe -> SwitchCLI -> RouterConfig -> NATPATConfig](https://github.com/BYeungCyberSec/CiscoPTProject/blob/main/README.md)

<h1>NAT and PAT Configuration</h1>

The project calls for static NAT configuration in the Branch network and PAT for the Main network.

<h1>Static Nat Configuration</h1>

As NAT/PAT is handled by the router exclusively, we'll only be configuring the router's for this section

A static NAT is very simple to configure, we just need to define the IP address of the host that we want statically NAT'd and the outgoing interface.

Inside of Branch router in configuration mode:
```
ip nat inside source static 172.17.186.2 209.165.200.188
```
We then repeat the same command inside of Main Router:
```
ip nat inside source static 172.17.186.2 209.165.200.188
```

That is all that is needed for a static NAT and this will work for User PC, however since Branch is part of the failsafe route, we need to configure it for PAT such that the network will still function if ISP-Main is down.

<h1>PAT Configuration</h1>

To configure PAT, we need to define an access-list first within the router
We can begin with Main Router:

```
access-list 1 permit 172.17.186.0 0.0.0.255
access-list 1 permit 172.17.180.0 0.0.0.127
access-list 1 permit 172.17.180.192 0.0.0.7
access-list 1 permit 172.17.180.128 0.0.0.63
```

This places all the network addresses with their wildcard mask within an access-list which we can define for NAT and PAT

```
ip nat pool NATPOOL 209.165.200.189 209.165.200.191 netmask 255.255.255.252
ip nat inside source list 1 pool NATPOOL overload
```

However, for NAT/PAT to function, we also have to define the outgoing and ingoing interfaces for NAT on Main and eventually Branch as well
In Main Router:
```
interface Gig0/0/0
ip nat inside
interface Gig0/0/1
ip nat outside
interface Se0/1/0
ip nat inside
interface Se0/1/1
ip nat inside
```

For Branch Router:
```
access-list 1 permit 172.17.186.0 0.0.0.255
access-list 1 permit 172.17.180.0 0.0.0.127
access-list 1 permit 172.17.180.192 0.0.0.7
access-list 1 permit 172.17.180.128 0.0.0.63
ip nat pool NATPOOL 209.165.200.189 209.165.200.191 netmask 255.255.255.252
ip nat inside source list 1 pool NATPOOL overload
```
And we can also assign ISP-Branch as ``` ip nat outside``` and the other interfaces as ```ip nat inside``` (Note that each VLAN has to be configured individually for ip nat inside)

All the end users should be able to ping each other now, and we can test the fail-safe routing when we force a shutdown on ISP-Main link through either router.

We can also configure ```logging synchronous``` and ```secret class``` but since this is a controlled home lab environment, this will only cause hassle when configuring so there is no need.

Note that the 1st-2nd pings between each device may drop, this is perfectly normal as ARP is being obtained and may result in a time out. The sequential pings should be successful if the network is configured correctly.
![Successful pings](https://github.com/BYeungCyberSec/CiscoPTProject/assets/150320582/ba62275c-9845-4b81-bfa0-e04defd5fce0)
