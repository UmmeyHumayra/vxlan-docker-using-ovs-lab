# vxlan-docker-using-ovs-lab
Create communication between containers using OpenVSwitch 
## Introduction:
- We have 2 VMs. 
- 2 docker containers are running on each of the VMs.
## Objective 
Our objective is to: 
- Use VxLAN to establish communication among these containers across VMs.
- Use Open VSwitch as Bridge to provision the VxLAN.
- Use NAT and iptables rules to establish internet connectivity for the docker containers. 
## Block diagram 
![vxlan-vxlan-ovs-block-diagram](https://github.com/UmmeyHumayra/vxlan-docker-using-ovs-lab/blob/assignment/vxlan-vxlan-ovs.jpeg)
## Steps

0. Setting up the lab environments: 
    >1. Make sure VMs can ping each other which confirms the underlay network is working as expected.
    >2. Update packages and install the essential utilities within the VMs to complete this hands-on lab. Packages include net-tools, docker.io, openvswitch.

1. Configure the Bridges:
    >1. Create 2 bridges on each of the VMs using openvswitch utility i.e. `ovs-vsctl` cli.
    >2. Create the internal port/interfaces to the newly created bridges using `ovs-vsctl` cli.
    >3. Set IP addresses to the bridge interfaces and bring them up using `sudo ip link set` command.

2. Next is to create 2 docker containers on each VM. Please note, when we'll do `docker run` we will use `--net=none` as a result containers will not be connected to the internet. Hence, we will build the docker image using a `Dockerfile` that contains the command to install the utilities and then use the image to run the dockers. As a result, when the containers will run, they'll have the utilities already installed in them.
    >1. Copy the docker file into the VMs and run `docker build` to create the docker image.
    >2. Create 4 containers (2 per VM) from the image using `docker run` and setting the network to `--net-none`. There'll be no internet connectivity. 
    >3. Check the containers' status and IP addresses
    >4. Assign the static IP Addresses to the containers using `ovs-docker` utility. Ping the GW IP to confirm the connectivity with OVS-bridge.

3. It's time to establish the ***VxLAN Tunnel*** between the VMs. The tunnel can be provisioned using `ovs-vsctl` cli and would expect arguments like VxLAN ID/VNI, UDP port 4789, and remote IP which is the other VM's IP. UDP port 4789 is a known port that caters vxlan traffic.
    >1. Check current status of the ports using `netstat -ntulp`.
    >2. Create the VxLAN tunnel using `ovs-vsctl add-port` command. 
    >3. At this point check the VM ports again to confirm port udp 4789 is ***listening***.
    >4. Now, test the connectivity. The containers in the same VxLAN should ping each other. But containers with different VNI won't be able to ping each other. 

4. Now that we can ping containers which are under same VxLAN but in different VMs, there is still no connectivity from the containers to ther Internet. We can do that using iptables for NAT. for details about iptables and NAT follow this [link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/4/html/security_guide/s1-firewall-ipt-fwd) 
    >1. Check the value in file /proc/sys/net/ipv4/ip_forward. On a Linux system, IP forwarding is enabled when the file /proc/sys/net/ipv4/ip_forward contains a 1 and disabled when it contains a 0. 
    >2. Enable ip forwarding by changing the value 0 to 1 using `sysctl -w` command that writes/sets a key to all arguments. We are using `sysctl` because this utility is used to modify kernel parameters at runtime.  
    >3. Run `sysctl -p <conf-file-path>` in order to make sure that IP forward enabling takes effect. 
    >4. Check the existing iptables rule list and add necessary rules to establish connection with the Internet.
    >5. Validate the connection by pinging a public IP from the container. 

## Commands following the steps: 
### Step 0. set up lab environment
 