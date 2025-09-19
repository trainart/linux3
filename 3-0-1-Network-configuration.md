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
|     Teacher    |      | Student 1 |      | Student 2 |
+----------------+      +-----------+      +-----------+
</pre>


### Second network interface

* Create second network interface in VM (with same parameters as the first one).
  * Set 
    * **Attached to** - "**_Bridged Adapter_**"
    * **Adapter Type** - "**_Paravirtualized Network (virtio-net)_**"
  * Start VM
    * After booting it will get the interface name **[enp0s8]**

* Teacher will tell each student the number in the list. 
  Use that number for you below instead of `x`

### Set your hostname to "lt0x.am"

```bash
nmcli general hostname lt0x.am
```

> The same can be done with `hostnamectl set-hostname lt0x.am`

### Configure static IP `10.10.x.1/24` on **[enp0s8]** interface.
  Instead on "**nmtui**" we can use "**nmcli**"


#### Show current state:

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

#### Add static route via Teacher IP as gateway
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

> All the above can be done with "**nmtui**" interactive interface


#### TEACHER CONFIG (**Students do not need to do this**)


```bash
nmcli connection modify "Wired connection 1" connection.id enp0s8
nmcli connection modify enp0s8 ipv4.method manual ipv4.addresses 10.10.0.1/24,10.10.1.111/24,10.10.2.111/24,10.10.3.111/24,10.10.4.111/24,10.10.5.111/24,10.10.6.111/24,10.10.7.111/24,10.10.8.111/24,10.10.9.111/24,10.10.10.111/24,10.10.11.111/24,10.12.8.111/24,10.10.13.111/24,10.10.14.111/24,10.10.15.111/24 connection.autoconnect yes
nmcli connection down enp0s8 ; nmcli connection up enp0s8
```

As a result Teacher will have `10.10.0.1` IP address for himself
and at the same time will have `10.10.x.111` address for each student's subnet,
as presented on the above chart.

### Check that everyting works


#### Check your IP address config

```bash
ip a
```

#### Check your route config

```bash
ip r
```

#### Try pinging Teacher's IP in your subnet

```bash
ping 10.10.x.111
```

#### Try pinging IP in Teacher's subnet

```bash
ping 10.10.0.1
```


#### Try pinging IP in some other Student's subnet

```bash
ping 10.10.5.1
```
