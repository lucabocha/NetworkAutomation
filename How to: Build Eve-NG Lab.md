# How to Build my Eve NG Lab in GGC (Gooogle Cloud)



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
