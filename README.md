# OCI IP Forwarding Gateway Setup (IPv4 + IPv6)

This guide sets up a public VM in Oracle Cloud Infrastructure (OCI) to act as a NAT gateway for private VMs, enabling internet access via IP forwarding. It supports both IPv4 and IPv6 routing.

---

## 🧱 Architecture Overview

- **Public VM**: Acts as NAT router with IP forwarding enabled.
- **Private VM**: Routes outbound traffic through the public VM.
- **No NAT Gateway used** — cost-effective and customizable.

---

## 🔧 Public VM Configuration (NAT Gateway)

### 1. Enable IP Forwarding Run in public VM

```bash

sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# IPv4 NAT
sudo iptables -t nat -A POSTROUTING -s 172.16.2.0/24 -o ens3 -j MASQUERADE

# IPv6 NAT (if supported) reolace ens3 as per your condition
sudo ip6tables -t nat -A POSTROUTING -o ens3 -j MASQUERADE

# IPv4 Replace CIDR as Per requirement
sudo iptables -I FORWARD 1 -s 172.16.2.0/24 -j ACCEPT
sudo iptables -I FORWARD 2 -m state --state RELATED,ESTABLISHED -j ACCEPT

# IPv6
sudo ip6tables -A FORWARD -i ens3 -j ACCEPT
sudo ip6tables -A FORWARD -o ens3 -j ACCEPT

#Run this below commands in both VM
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 2 -p tcp --dport 443 -j ACCEPT
sudo iptables -I INPUT 3 -p tcp --dport 5678 -j ACCEPT
sudo iptables -I INPUT 4 -p tcp --dport 8080 -j ACCEPT


#save rules
sudo netfilter-persistent save


```
### 2. Disable Source/Destination Check

- Go to OCI Console → Networking → VNICs
- Select the public VM’s VNIC
- Click Edit
- Enable Skip Source/Destination Check
- create rule in private route table with 0.0.0.0/0 and put private ip of public vm
- save




