# Understanding and Make Linux Network Namespaces
#### What is Linux Namespaces?
**_Linux namespace_** is an abstraction over resources in the operating system. We can think of a namespace as a box. Inside this box are these system resources, which ones exactly depend on the box’s (namespace’s) type. There are currently 7 types of namespaces `Cgroup`, `IPC`, `Network`, `Mount`, `PID`, `User`, `UTS`.


#### What is Network Namespaces?
Network namespaces, according to `man 7 network_namespaces`:

**_network namespaces provide isolation of the system resources associated with networking: network devices, IPv4 and IPv6 protocol stacks, IP routing tables, firewall rules, the /proc/net directory, the /sys/class/net directory, various files under /proc/sys/net, port numbers (sockets), and so on._**


#### Virtual Interfaces and Bridges:
**_Virtual interfaces_** provide us with virtualized representations of physical network interfaces; and the **_bridge_** gives us the virtual equivalent of a switch.


#### What are we going to cover?
- Create two Namespaces and connect them using veth.
- Create two Namespaces and connect them using Linux bridge.

## Let's start...

## Create two Namespaces and connect them using veth.

**_Step 1:_** Create two namespaces (ns1, ns2)
- sudo ip netns add ns1
- sudo ip netns add ns2

**_Step 2:_** Check created namespaces
- sudo ip netns list

![namespace_create_list](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/86d1be91-25b8-4a67-a94e-f97cc927d33a)


**_Step 3:_** Create virtual ethernet cable with two connector (ns-1-eth0, ns-2-eth0)
- sudo ip link add ns-1-eth0 type veth peer name ns-2-eth0

**_Step 4:_** Check created veth
- sudo ip link

![veth with connector](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/07b2c8b5-3389-45b8-a9f1-872228ba7132)


**_Step 5:_** Connect ns1 with veth ns-1-eth0 (one side of previously created veth) and ns2 with veth ns-2-eth0 (another side of previously created veth)
- sudo ip link set ns-1-eth0 netns ns1
- sudo ip link set ns-2-eth0 netns ns2

**_Step 6:_** Setup ns1
- sudo ip netns exec ns1 bash
- ip link (check interface)
- ip link set ns-1-eth0 name eth0 (rename interface ns-1-eth0 to eth0)
- ip addr add 192.168.0.1/24 dev eth0 (set ip address)
- ip link set lo up (by default loopback down, so up loopback)
- ip link set eth0 up (by default down, so up eth0)
- ip link (check all interface that we are configured)

![ns1](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/034110f1-a825-4ccf-b984-28492563a3b5)


**_Step 7:_** Setup ns2
- sudo ip netns exec ns2 bash
- ip link (check interface)
- ip link set ns-2-eth0 name eth0 (rename interface ns-2-eth0 to eth0)
- ip addr add 192.168.0.2/24 dev eth0 (set ip address)
- ip link set lo up (by default loopback down, so up loopback)
- ip link set eth0 up (by default down, so up eth0)
- ip link (check all interface that we are configured)

![ns2](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/9456b455-b5c0-424a-a7c7-6b8f387e6ed5)

**_Step 7:_** Test NS2 Connection from NS1
- sudo ip netns exec ns1 bash
- ping 192.168.0.2

**_Step 8:_** Test NS1 Connection from NS2
- sudo ip netns exec ns2 bash
- ping 192.168.0.1

![ns1, ns2 test](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/043da053-3470-4fba-b2b6-788c390c61db)


#### Finally completed the two Namespaces and connect them using veth and test both side
