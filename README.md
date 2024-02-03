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


## Create two Namespaces and connect them using Linux bridge.

**_Step 1:_** Create two namespaces (ns1, ns2)
- sudo ip netns add ns1
- sudo ip netns add ns2

**_Step 2:_** Check created namespaces
- sudo ip netns list

![namespace_create_list](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/86d1be91-25b8-4a67-a94e-f97cc927d33a)

**_Step 3:_** Create Bridge and configure
- sudo ip link add br0 type bridge
- sudo ip link set br0 up
- sudo ip addr add 192.168.1.1/24 dev br0
- ip addr
  
![bridge 1](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/8d63c9ff-31fb-4a7f-b4df-c140f8c2a95d)

**_Step 4:_** Test Bridge connection
  - ping -c 2 192.168.1.1

 ![2024-02-03-12-38-47](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/aa15f6e1-b096-473a-819d-641848a09964)

**_Step 5:_** For ns1

  #### Create virtual ethernet cable and configure
  - sudo ip link add veth0 type veth peer name ceth0
  - sudo ip link set veth0 master br0 (connect to the bridge br0)
  - sudo ip link set veth0 up (up veth0)
  - sudo ip link set ceth0 netns ns1 (connect to the ns1)
    
  #### Configure interface
  - sudo ip netns exec ns1 bash
  - ip link set lo up
  - ip link set ceth0 up
  - ip addr add 192.168.1.10/24 dev ceth0
  - ip add

![2024-02-03-13-16-19](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/fdac28b4-f5ee-4f99-8988-ab345ffca893)


**_Step 6:_** For ns2

  #### Create virtual ethernet cable and configure
  - sudo ip link add veth1 type veth peer name ceth1
  - sudo ip link set veth1 master br0 (connect to the bridge br0)
  - sudo ip link set veth1 up (up veth1)
  - sudo ip link set ceth1 netns ns2 (connect to the ns2)
    
  #### Configure interface
  - sudo ip netns exec ns2 bash
  - ip link set lo up
  - ip link set ceth1 up
  - ip addr add 192.168.1.11/24 dev ceth1
  - ip add
    
  ![2024-02-03-13-18-50](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/f0ce2dd9-b8f7-4b1a-827f-acf1b5aeca9c)

  
**_Step 7:_** Test NS2, Bridge Connection from NS1

  - ping -c 2 192.168.1.1 (test br0)

 ![2024-02-03-13-24-57](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/05c4a735-d73a-438b-9b95-cd012a888c1d)

    
  - ping -c 2 192.168.1.11 (test ns2)
 
 ![2024-02-03-13-25-11](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/bf851e74-4879-4718-8555-4bc8b5867a8d)

   
    
**_Step 8:_** Test NS1, Bridge Connection from NS2

  - ping -c 2 192.168.1.1 (test br0)

 ![2024-02-03-13-21-18](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/7d50ad9c-600e-41f4-bf19-7cf14aca438a)

    
  - ping -c 2 192.168.1.10 (test ns1)
    
 ![2024-02-03-13-21-39](https://github.com/Tareq-Islam/linux-network-namespace/assets/19193021/c5122ba2-cb7a-4be4-a533-174cf3e0a11f)

    

#### Finally completed the two Namespaces and connect them using linux bridge.
