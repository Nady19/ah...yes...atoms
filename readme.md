# Step By Step Guide

--> Log in to Kloudlab

--> go to AWS console (on the left menu bar)

--> `request credentials`

--> copy paste the url in another tab and use the given username and password to log in

--> change region to `Asia Pacific (Mumbai)ap-south-1`

--> go to `All Services`

--> open `EC2` in another tab

--> open `VPC` in another tab

Now VPC has CIDR notations to define the network range, route table etc etc...
But we need to create a subnet under a VPC

--> go to the left menu and open `subnet` in another tab

--> `create subnet` (top right corner)

--> using `prod-kube` as `vpc ID`

--> using `worker-subnetn` as the name of the subnet

--> zone is `ap-south-1`

--> since the VPC prod-kube 's CIDR was 10.1.0.0/16, the subnet's ip must be under it so we can go for `10.1.7.0/24` which means we can launch 251 VMs under this subnet. ({2^(32-24)}-5)=251

--> now we go back to the EC2 tab

--> `launch instance`

--> `t2 micro`

--> select `prod-kube` as network

--> select `worker-subnetn` as subnet (the one i just created)

-->`auto assign public IP` ---`enable`

-->`Add storage`

-->`Add tags`---`Add tag`----`Name`----`Ndevops`

--> `Configure security group`---changing name to `Ndevops-sg`

--> change source to `My IP`

-->`review and launch`

-->`launch`

--> ` create new key pair` 

Name : `Ndevops19`

--> `Download key pair`

-->`launch instances`

--> from terminal, go to where u downloaded the key pair `cd`

--> `sudo chmod 400 Ndevops19.pem`

--> `ssh -i Ndevops19.pem ubuntu@13.232.32.104`(go to the instance u just created and copy the public ip address from there)

-->`yes`

Basically we are at the terminal of the vm we just created moments ago.


-->now go to browser and go to the instance you just created , go to `subnet`, then go to `route table` under that subnet 

--> go to the `edit route table` 

--> if we delete the gateway `igw.....` then from the terminal we can no more ssh it or access it

-->so we try `telnet ubuntu@13.232.32.104`

-->we can see that packet is going to the address and into the server but when the packet is going out as directed from the server it doesnt go anywhere because we just delete the only `gateway`. As a result, there is no default gateway from which the packet can go out .

-->Even changing the security group's `inbound rules` to `anywhere from the world` doesn't change anything

-->after changing inbound rules we can ping the address but it seems like the packet is going there but then showing no response.

-->it is simply because there is no route table or to be more precise no gateway in the route table from where the packet would get out.

--> so now we edit the route table and `add route` 

-->then add my ip as a route with internet gateway `igw....prodkube`

-->go to the terminal again and you can see that your packet is responding now and we can ping

-->but no other ip can ping it though.

-->lets now change the route back to `0.0.0.0/0` and turn everything to normal.

# installing docker in the vm

`sudo apt update`

`sudo apt install net-tools`

`sudo apt install docker`

to see route table :`route`

to see other pcs under same subnet :`arp -a`

to create a new network namespace :`sudo ip netns add red`

to see if it was added: `sudo ip netns list`

adding another : `sudo ip netns add green`

to go inside "red" namespace and open shell: `sudo ip netns exec red sh`

to see interface : `ip link`

host has two interfaces though, host and loopback

Now we want to make a wire between the green and the red namespaces and give them ip sothat we can ping them from each other

--> `sudo ip link add veth-red type veth peer name veth-green`

to see `ip link` whether or not the interfaces were created .

basically we are creating a wire to connect red and green namespaces.

the veth-red is the socket that will go into red and veth-green is the socket that will go into the green namespace. So , lets get to it :

`sudo ip link set veth-red netns red`

we can check `ip link` and see that there is 1 interface less as the veth-red interface has submerged into red namespace. So lets do it for green too and make the wire functional.

`sudo ip link veth-green netns green`

Now it is time for assign IP

`sudo ip netns exec red sh`

`ip addr add 10.1.1.1/24 dev veth-red`

It is not up yet so:

`ip link set dev veth-red up`

We will do the same for green name space:

`sudo ip netns exec green sh`

`ip addr add 10.1.1.2/24 dev veth-green`

`ip link set dev veth-green up`

Now if we ping the red namespace from here (green) we can see the packet transferring successfully

`ping 10.1.1.1`

Doing with instructions one might not feel what he is doing BUT ,it is really a significant task that u can virtually create namespaces, then wire to connect those namespaces and ping from each other....its like taking a matter and going into its atoms and doing works there .
SO YEAH it is really significant work.