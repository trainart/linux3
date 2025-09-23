# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)

## Network configuration

For Level 3 we will need to have the environment like follows:

<pre>
--------+---------------------+------------------+------------
        | [enp0s8]            | [enp0s8]         | [enp0s8]
        | 10.10.0.1           | 10.10.1.1        | 10.10.2.1
        | 10.10.1.111         |                  |
        | 10.10.2.111         |                  |
        | 10.10.x.111         |                  |
+-------+--------+      +-----+-----+      +-----+-----+
|     lt00.am    |      |  lt01.am  |      |  lt02.am  |
|     Trainer    |      | Student 1 |      | Student 2 |
+----------------+      +-----------+      +-----------+
</pre>


## Second network interface

Both trainer and students should do this section.

* Create second network interface in VM (with same parameters as the first one).
  * Set 
    * **Attached to** - "**_Bridged Adapter_**"
    * **Adapter Type** - "**_Paravirtualized Network (virtio-net)_**"
  * Start VM
    * After booting it will get the interface name **[enp0s8]**


  
## Trainer's Config (**Students do not need to do this section **, it is provided just for information)

### Show current state

```bash
nmcli connection show
```

### Configure necessary parameters

```bash
nmcli connection modify "Wired connection 1" connection.id enp0s8 ;\
nmcli connection modify enp0s8 ipv4.method manual ipv4.addresses 10.10.0.1/24,10.10.1.111/24,10.10.2.111/24,10.10.3.111/24,10.10.4.111/24,10.10.5.111/24,10.10.6.111/24,10.10.7.111/24,10.10.8.111/24,10.10.9.111/24,10.10.10.111/24,10.10.11.111/24,10.12.8.111/24,10.10.13.111/24,10.10.14.111/24,10.10.15.111/24 connection.autoconnect yes ;\
nmcli connection down enp0s8 ;\
nmcli connection up enp0s8 ;\
nmcli general hostname lt00.am
```

As a result trainer will have `10.10.0.1` IP address for himself
and at the same time will have `10.10.x.111` address for each student's subnet,
as presented on the above chart.

### Check result

Check IP address config

```bash
ip a
```

Check route config

```bash
ip r
```

### Permanently enable IP forwarding

Because according to our network configuration only trainer's Linux system will act as router, 
we need to permanently enable IP forwarding there.
(**Students do not need to do this**, because their Linuxes act as an end host, not a router).

Add config parameter:

```bash
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf 
```

Activate the change:

```bash
sysctl -p 
```

Check it's now enabled:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

> For security reasons on production Linux systems IP forwarding is to be enabled only it is actually required.
> Keeping it disabled (0) is the default and more secure for most Linux use cases.
> We do this here on trainer's Linux system only for this training purposes !


## STUDENT's Config

Trainer will tell each student the number in the list. 
Use that number for you below instead of `x`.


### Configure static IP `10.10.x.1/24` on **[enp0s8]** interface.
Instead on "**nmtui**" we can use "**nmcli**"


#### Show current state

```bash
nmcli connection show
```

#### Rename the Connection Profile

```bash
nmcli connection modify "Wired connection 1" connection.id enp0s8
```

#### Assign static IP 

```bash
nmcli connection modify enp0s8 ipv4.method manual ipv4.addresses 10.10.x.1/24 connection.autoconnect yes
```

#### Add static route via trainer's IP as gateway
nmcli connection modify enp0s8 +ipv4.routes "10.10.0.0/16 10.10.x.111"

#### Apply the changes

```bash
nmcli connection down enp0s8 ; nmcli connection up enp0s8
```

#### Check that config is persistent (will remain after reboot)

Type <Tab><Tab> to see files in that directory

```bash
cat /etc/NetworkManager/system-connections/
```

### Set your hostname to "lt0x.am"

```bash
nmcli general hostname lt0x.am
```

> The same can be done with `hostnamectl set-hostname lt0x.am`


> All the above can also be done with "**nmtui**" interactive tool.


### Disable ICMP redirects for `enp0s8` interface

In TCP/IP (inside Linux Kernel) ICMP redirects are used to inform hosts of a "better" next-hop route.
Below we configure Linux not to accept it to have clear model of our routing.
(This is required only for our training, not for production, because here we put many subnets in same network).

```bash
echo "net.ipv4.conf.enp0s8.accept_redirects=0" >> /etc/sysctl.conf
```

```bash
echo "net.ipv4.conf.enp0s8.send_redirects=0" | tee -a /etc/sysctl.conf
```

> NOTE! 
> In the above 2 commands the same thing is done in 2 ways.
> On seconde line instead of `>>` we used `| tee -a`
> Difference is that `tee` command will also show the same in the terminal.
> You should not see first line output, but should see the second one.
> So this way is more visual, than `>>`

Now let's permanently apply changes.

```bash
sysctl -p
```


## Test whole config 

Students can check connection to trainer's Linux by pinging trainer's IP in student's subnet.

```bash
ping 10.10.x.111
```

Or pinging IP in trainer's subnet

```bash
ping 10.10.0.1
```

Both should work for all students.


Trainer can check all student's IP's

```bash
yum -y install fping
```

```bash
fping -c3 -aq 10.10.{1..15}.1
```


Now student's can try pinging each other's IPs 
(pinging IP in some other student's subnet)

```bash
ping 10.10.1.1
```

It should also work.
Keep that ping till the trainer will run following:

```bash
echo 0 > /proc/sys/net/ipv4/ip_forward
```

Ping should stop. Why ?

After trainer will run following, ping should work again:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

You can also try tracing the path

```bash
yum -y install mtr 
```


```bash
mtr 10.10.1.1
```

Now our Network Setup should be complete !

