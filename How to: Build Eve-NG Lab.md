# How to Build my Eve NG Lab in GGC (Gooogle Cloud)

The initial set up of the Eve NG virtual machine in google cloud was done following the instructions on this [link](https://www.eve-ng.net/index.php/documentation/community-cookbook/). More specficailly starting from page 40

Either way, Im going to outline the instructions specifically related to the deployment of the VM and not the creation of the google account or anything of the sort

## Create the Virtual Machine 

1. Open the Google Cloud Shell :

  <div align="center" dir="auto">

![GCC_Shell](https://github.com/lucabocha/NetworkAutomation/assets/44237986/df380329-ce0a-4b6b-a4ac-a536af1981b3)

  </div>

2. Execute the following command in the Google Cloud Shell:

        gcloud compute images create nested-ubuntu-focal --source-image-family=ubuntu-2004-lts --source-image-project=ubuntu-os-cloud --licenses https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx

This will create the image we are going to use when deploying the Virtual Machine in Google Cloud. When the command is completed you should get the following output:  

    username@cloudshell:~ (my-eve-ng-network-lab)$ gcloud compute images create nested-ubuntu-focal --source-image-family=ubuntu-2004-lts --source-image-project=ubuntu-os-cloud --licenses https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx
    Created [https://www.googleapis.com/compute/v1/projects/my-eve-ng-network-lab/global/images/nested-ubuntu-focal].
    NAME: nested-ubuntu-focal
    PROJECT: my-eve-ng-network-lab
    FAMILY: 
    DEPRECATED: 
    STATUS: READY

3. Create the VM using the Google Cloud Console
  - Go to "Compute Engine" > "VM Instances" > "Create Instance"

  <div align="center" dir="auto">
    
![GCC VM Instances](https://github.com/lucabocha/NetworkAutomation/assets/44237986/c2cb7b5b-8cea-49d2-b563-a7ba7d4691c0)

![Create Instance GCC](https://github.com/lucabocha/NetworkAutomation/assets/44237986/1042da87-a80b-441c-a8d4-e869b2c03595)

  </div>

  - Assign the name for the VM
  - Set your Region and zone
  - Edit your machine configuraion. General Purpose. Choose the series of CPU platform to be Intel CPUs Skylake or Cascade. (In my case I picke the following machine type: **n2-standard-8** )
  - Chose your CPU and RAM settings and make sure that the option "Deploy a container image is unchecked or simply is not enabled"
  - Select the image previously created
  - Allow HTTP traffic and create the VM 

For this third point you can reference the guide on this [link](https://www.eve-ng.net/index.php/documentation/community-cookbook/) on page 43 & 44 which contain an example of tha VM machine deployment

## EVE-NG Installation 

1. For this installation you can follow the instructions on this [link](https://www.eve-ng.net/index.php/documentation/community-cookbook/) on pages 45 & 46. Im not explaining this since it is very well explained on this guide so it will be repeatetive to outline all the steps on this guide too 

## How to Connect to Even NG Lab devices to the Internet

First, lets clarify some basics about the eve ng machine network interfaces

- The pnet0 interface maps to the main interface for this VM. This is the interface that interacts directly to the google cloud network
- THe other pnet interfaces from 1 to 9 will each of them be related to the cloud networks available in the eve NG Lab. For example:

      root@myevenglab:~# ifconfig | grep pnet
      pnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
      pnet1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      pnet9: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

<div align="center" dir="auto">


![EvenNGClouds](https://github.com/lucabocha/NetworkAutomation/assets/44237986/c611df66-6e0c-4113-b0d4-75930eab0fd4)

</div>

With this much information we will be able to understand the below steps better. So, lets start setting up the internet access for the lab devices 

1. Edit the pnet1 interface configuration to give it an IP address. This IP address should be an IP that we will not use on any of the lab we will be setting up 

        sudo su 
        vim /etc/network/interfaces

   This will open the VIM editor so you configure the interface like below. In my case I know that I wont be using 10.255.255.0/24 IP range anywhere else on the labs 
   
        iface pnet1 inet static
            bridge_ports eth1
            bridge_stp off
            address 10.255.255.1
            network 255.255.255.0

2. Enabled the IP forwarding in the EVE NG VM so it can start routing traffic from the internal labs to the internet

        sudo su
        vim /etc/sysctl.conf 

  - Find the following line in this file and uncomment it (Remove the "#" at the beginning)

        net.ipv4.ip_forward=1
  
  - Reload this configuration so the changes made can be applied

        sysctl -p /etc/sysctl.conf 

3. Install the iptables-persistent package, so we can then later save the ip table configuration if everything works as expected:

        sudo su 
        apt install iptables-persistent

4. Execute the following command in order to start natting the internal IPs (pnet1) to the internet. Have in mind that you will have to change the IP addresses on the command in order to match the ip address range previusly configured for pnet1. Here, Im pasting the command I used on my set up since I configured the ip 10.255.255.1/24 on pnet1 

        iptables -t nat -A POSTROUTING -s 10.255.255.0/24 -o pnet0 -j MASQUERADE 

This command will basically receive traffic from 10.255.255.0/24 and then nat it out to pnet0, which is the interface that has connectivity to the internet

5. Confirm that the internet connectivity works. You can do it by setting up something like this and pinging from the device to the internet. Make sure that the IP addresses is configured correctly on the device (It needs to be in the same range as previously applied on pnet1) and also the routing is correctly set up. 

  <div align="center" dir="auto">
    
![Eve NG Lab Set up ](https://github.com/lucabocha/NetworkAutomation/assets/44237986/b3c81e56-05d0-43e8-969c-4e0603bae9ed)

  </div>

A simple ping to 8.8.8.8 from the test device should confirm or not if the internet connectivity is working 

5. Finally (If everything is working correctly), save the IP tables configuration so it can survive reboots 

        iptables-save > /etc/iptables/rules.v4 

If you need to give internet connectivity to any other device simply connect it to the cloud1, with an IP in the range of 10.255.255.0/24 that is not repeated anywhere else and the internet connectivity should work. 

## How to connect from your home network or from the internet to the Lab Device in Eve NG

The goal here is to initiate an ssh session from your local computer and that it connects to the lab device in Eve NG. 

For this we are going to make use two destination nats: One is going to be configured on the Eve NG virtual machine using ip tables and the other is configured on a firewall device (In my case it is an vSRX) so the connectivity should work this way: 

- You computer is going to initiate an ssh session to the public IP of the Eve NG machine on tcp port 8001
- This arrives at the Eve NG machine which is going to translate the destination IP to 10.255.255.10 and going to keep the same destination port 8001
- This is going to arrive to the lab firewall (vSRX) and this is going to translate the destination IP and destination port to 10.255.254.11 destination port 22 and forward this packet to 10.255.254.11
- The packet will be received on this host which is listening on port 22 (ssh) and then the connection is going to be established

This is the diagram for the test setup: 

  <div align="center" dir="auto">

![EveNGSetup2](https://github.com/lucabocha/NetworkAutomation/assets/44237986/89dceeea-20a5-41f6-9b1f-6ee0330ab623)

  </div>

1. Set up the 1st destination nat on the Eve NG Virtual Machine:
   - Issue the following command to create the destination nat on the iptables. Make sure to change the IP address inforamtion as needed 

          iptables -t nat -A PREROUTING -p tcp --dport 8000:9000 -i pnet0 -j DNAT --to-destination 10.255.255.10

This command is going to match traffic arriving on pnet0 with a destination port between 8000 and 9000 and then change the destination IP address to 10.255.255.10 

2. Configure the 2nd destination nat on the firewall (vsrx):
   
        set security nat destination pool mgmt-ip-1 address 10.255.254.11/32
        set security nat destination pool mgmt-ip-1 address port 22
        set security nat destination pool mgmt-ip-2 address 10.255.254.12/32
        set security nat destination pool mgmt-ip-2 address port 22
        set security nat destination rule-set mgmt-devices from interface ge-0/0/0.0
        set security nat destination rule-set mgmt-devices rule device-1 match destination-address 10.255.255.10/32
        set security nat destination rule-set mgmt-devices rule device-1 match destination-port 8001
        set security nat destination rule-set mgmt-devices rule device-1 match protocol tcp
        set security nat destination rule-set mgmt-devices rule device-1 then destination-nat pool mgmt-ip-1
        set security nat destination rule-set mgmt-devices rule device-2 match destination-address 10.255.255.10/32
        set security nat destination rule-set mgmt-devices rule device-2 match destination-port 8002
        set security nat destination rule-set mgmt-devices rule device-2 match protocol tcp
        set security nat destination rule-set mgmt-devices rule device-2 then destination-nat pool mgmt-ip-2

Here you can see the mapping that the destination nat creates: 

10.255.255.10:8001 - > Translate to - > 10.255.254.11:22 
10.255.255.10:8002 - > Translate to - > 10.255.254.12:22 

Just like this you can create multiple other rule to map to other devices if needed 

You can refer to the following [link](https://www.juniper.net/documentation/us/en/software/junos/nat/topics/topic-map/security-nat-destination.html) for more details about destination nat for Juniper SRX devices

With this you should be able to ssh to the Eve NG public IP address on port 8001 and connect to the Test_Device1 management via SSH from your computer 

Please see below the whole configuration of the Mgmt-Firewall device: 

      set interfaces ge-0/0/0 unit 0 family inet filter input test
      set interfaces ge-0/0/0 unit 0 family inet address 10.255.255.10/24
      set interfaces ge-0/0/1 unit 0 family inet address 10.255.254.1/24
      set routing-options static route 0.0.0.0/0 next-hop 10.255.255.1
      set security screen ids-option untrust-screen icmp ping-death
      set security screen ids-option untrust-screen ip source-route-option
      set security screen ids-option untrust-screen ip tear-drop
      set security screen ids-option untrust-screen tcp syn-flood alarm-threshold 1024
      set security screen ids-option untrust-screen tcp syn-flood attack-threshold 200
      set security screen ids-option untrust-screen tcp syn-flood source-threshold 1024
      set security screen ids-option untrust-screen tcp syn-flood destination-threshold 2048
      set security screen ids-option untrust-screen tcp syn-flood queue-size 2000
      set security screen ids-option untrust-screen tcp syn-flood timeout 20
      set security screen ids-option untrust-screen tcp land
      set security nat destination pool mgmt-ip-1 address 10.255.254.11/32
      set security nat destination pool mgmt-ip-1 address port 22
      set security nat destination pool mgmt-ip-2 address 10.255.254.12/32
      set security nat destination pool mgmt-ip-2 address port 22
      set security nat destination rule-set mgmt-devices from interface ge-0/0/0.0
      set security nat destination rule-set mgmt-devices rule device-1 match destination-address 10.255.255.10/32
      set security nat destination rule-set mgmt-devices rule device-1 match destination-port 8001
      set security nat destination rule-set mgmt-devices rule device-1 match protocol tcp
      set security nat destination rule-set mgmt-devices rule device-1 then destination-nat pool mgmt-ip-1
      set security nat destination rule-set mgmt-devices rule device-2 match destination-address 10.255.255.10/32
      set security nat destination rule-set mgmt-devices rule device-2 match destination-port 8002
      set security nat destination rule-set mgmt-devices rule device-2 match protocol tcp
      set security nat destination rule-set mgmt-devices rule device-2 then destination-nat pool mgmt-ip-2
      set security policies from-zone OUTSIDE to-zone DMZ policy default-permit match source-address any
      set security policies from-zone OUTSIDE to-zone DMZ policy default-permit match destination-address any
      set security policies from-zone OUTSIDE to-zone DMZ policy default-permit match application any
      set security policies from-zone OUTSIDE to-zone DMZ policy default-permit then permit
      set security policies from-zone OUTSIDE to-zone OUTSIDE policy default-permit match source-address any
      set security policies from-zone OUTSIDE to-zone OUTSIDE policy default-permit match destination-address any
      set security policies from-zone OUTSIDE to-zone OUTSIDE policy default-permit match application any
      set security policies from-zone OUTSIDE to-zone OUTSIDE policy default-permit then permit
      set security policies from-zone DMZ to-zone OUTSIDE policy default-permit match source-address any
      set security policies from-zone DMZ to-zone OUTSIDE policy default-permit match destination-address any
      set security policies from-zone DMZ to-zone OUTSIDE policy default-permit match application any
      set security policies from-zone DMZ to-zone OUTSIDE policy default-permit then permit
      set security zones security-zone OUTSIDE interfaces ge-0/0/0.0
      set security zones security-zone DMZ interfaces ge-0/0/1.0
      set firewall family inet filter test term 1 from protocol tcp
      set firewall family inet filter test term 1 from destination-port 8001
      set firewall family inet filter test term 1 then log
      set firewall family inet filter test term 1 then accept
      set firewall family inet filter test term 2 then accept

