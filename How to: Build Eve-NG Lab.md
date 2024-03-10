# How to Build my Eve NG Lab in GGC (Gooogle Cloud)

The initial set up of the Eve NG virtual machine in google cloud was done following the instructions on this [link](https://www.eve-ng.net/index.php/documentation/community-cookbook/). More specficailly starting from page 40

Either way, Im going to outline the instructions specifically related to the deployment of the VM and not the creation of the google account or anything of the sort

1. Open the Google Cloud Shell :
  <div align="center" dir="auto">

![GCC_Shell](https://github.com/lucabocha/NetworkAutomation/assets/44237986/df380329-ce0a-4b6b-a4ac-a536af1981b3)

  </div>

this jouijnarvoinarwovnoauindw  
gcloud compute images create nested-ubuntu-focal --source-image-family=ubuntu-2004-lts --source-image-project=ubuntu-os-cloud --licenses https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx

lucabocha@cloudshell:~ (my-eve-ng-network-lab)$ gcloud compute images create nested-ubuntu-focal --source-image-family=ubuntu-2004-lts --source-image-project=ubuntu-os-cloud --licenses https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx
Created [https://www.googleapis.com/compute/v1/projects/my-eve-ng-network-lab/global/images/nested-ubuntu-focal].
NAME: nested-ubuntu-focal
PROJECT: my-eve-ng-network-lab
FAMILY: 
DEPRECATED: 
STATUS: READY



/etc/network/interfaces
pnet1: 

iface pnet1 inet static
    bridge_ports eth1
    bridge_stp off
    address 10.255.255.1
    network 255.255.255.0

/etc/sysctl.conf 

    net.ipv4.ip_forward=1

sysctl -p /etc/sysctl.conf 

apt install iptables-persistent

iptables-save > /etc/iptables/rules.v4 


VSRX ip to Cloud 9 10.255.255.10/24 


iptables -t nat -A PREROUTING -p tcp --dport 8000:9000 -i pnet0 -j DNAT --to-destination 10.255.255.10


set version 12.1X46-D30.2
set system host-name vSRX-MGMT-Firewall
set system root-authentication encrypted-password "$1$Kvb73fFf$pvZBxJ5Qxftf8yHvaQ0Lh."
set system services ssh
set system services web-management http interface ge-0/0/0.0
set system syslog file messages any any
set system license autoupdate url https://ae1.juniper.net/junos/key_retrieval
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
set security nat destination pool mgmt-ip-2 address 10.255.254.2/32
set security nat destination pool mgmt-ip-2 address port 22
set security nat destination rule-set mgmt-devices from interface ge-0/0/0.0
set security nat destination rule-set mgmt-devices rule device-1 match destination-address 10.255.255.10/32
set security nat destination rule-set mgmt-devices rule device-1 match destination-port 8001
set security nat destination rule-set mgmt-devices rule device-1 match protocol tcp
set security nat destination rule-set mgmt-devices rule device-1 then destination-nat pool mgmt-ip-1
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
