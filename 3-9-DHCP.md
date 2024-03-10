# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## DHCP, Dnsmasq

Dnsmasq is the lightweight DNS forwarder and DHCP Server Software

> In order to test it in VM environment create additional "Host-Only" network interface
> and configure DHCP server to work on it only 


```bash
yum -y install dnsmasq
```

If `firewalld` is running, allow DNS and DHCP services
```bash
firewall-cmd --add-service=dns –permanent
firewall-cmd --add-service=dhcp –permanent
firewall-cmd –reload
```


Configure Dnsmasq as DHCP only Server

```bash
cat  > /etc/dnsmasq.d/dhcp-only.conf  << "EOF1"
port=0 # don't function as a DNS server
dhcp-range=192.168.168.2,192.168.168.253,12h
dhcp-option=1,255.255.255.0 # Subnet Mask
dhcp-option=3,192.168.168.1 # Default route
dhcp-option=6,192.168.168.1 # DNS server
dhcp-option=28,192.168.168.255 # Broadcast address
log-dhcp # Log DHCP transactions
interface=lo
interface=enp0s8
except-interface=enp0s3

EOF1

```

> "1" means “Subnet Mask", 
> "3" means "Default route",
> "6" means "DNS server" 
> "28" means "Broadcast address"

Start & enable dnsmasq service to start on each boot
```bash
systemctl enable --now dnsmasq
```

