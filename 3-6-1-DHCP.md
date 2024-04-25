# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## DHCP, Dnsmasq

Dnsmasq is the lightweight DNS forwarder and DHCP Server Software

> In order to test it in VM environment create additional "Internal Network" network interface
> and configure DHCP server to work on it only 

#### Create new separate interface for DHCP testing

Since we can't have multiple DHCP servers in a single network we need separate environment for that.

In VirtualBox config add new network device and assign "Internal Network"
After booting that interface will get `enp0s9` name.

> You will also need second VM with same additional interface as Client.


```bash
yum -y install dnsmasq
```

If `firewalld` is running, allow DNS and DHCP services
```bash
firewall-cmd --add-service=dns –permanent
firewall-cmd --add-service=dhcp –permanent
firewall-cmd –reload
```


#### Add **Rsyslog** configuration to log Dnsmasq logs separately.

```bash
cat > /etc/rsyslog.d/dnsmasq.conf << "ENDTEXT"
if $programname == "dnsmasq" then /var/log/dnsmasq.log
ENDTEXT

```

Restart rsyslog:
```bash
systemctl restart rsyslog
```

Restart dnsmasq:
```bash
systemctl restart dnsmasq 
```

Check

```bash
tail /var/log/dnsmasq.log
```

### Assign static address to `ens0s9` interface 

Use `nmtui` to assign `ens0s9` interface static address `192.168.168.1/24`


### Configure Dnsmasq as DHCP only Server

```bash
cat  > /etc/dnsmasq.d/dhcp-only.conf  << "EOF1"
port=0 # don't function as a DNS server
dhcp-range=192.168.168.2,192.168.168.253,12h
dhcp-option=1,255.255.255.0 # Subnet Mask
dhcp-option=3 # Default route
dhcp-option=6 # DNS server
#dhcp-option=3,192.168.168.1 # Default route
#dhcp-option=6,192.168.168.1 # DNS server
dhcp-option=28,192.168.168.255 # Broadcast address
default-lease-time=600
log-dhcp # Log DHCP transactions
interface=lo
interface=enp0s9
except-interface=lo
except-interface=enp0s3
except-interface=enp0s8
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


Now on another server configure `enp0s9` to get interface settings automatically via DHCP.

And check here:

```bash
cat /var/lib/dnsmasq/dnsmasq.leases
```

