# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)

## Linux firewall, packet filtering, iptables 

"Firewall" term is generally used for some mechanism to do **network security** and **traffic management** tasks.

**Linux firewall** mechanism is implemented in Linux kernel in form of network firewall capabilities to filter packets (called **Netfilter**).

Initially, Linux firewall was designed as **packet filter**. 
The goal was to define rules to permit, deny, or modify network traffic 
based on various criteria such as **source/destination IP addresses**, **ports**, and **protocols**.

So **Netfilter** is a framework within the Linux kernel that provides various functionalities for packet filtering.

But **Linux firewall** term also involves additional components on top of **Netfilter**. 
These are various tools to configure and manage it.
Different Linux distributions and versions use different tools.
Also some tools function on top of others.

We can call "**backend tools**" - those that directly interact with Netfilter (part of Linux kernel) to define firewall rules and manipulate network packets.

And "frontend tools" - higher-level tools that work on top of "**backend tools**".

Current "**backend tools**" in Linux firewall system are:

* **IPtables**
* **Nftables**

Among "**frontend tools**" in Linux firewall system are:

* **FirewallD**
* **UFW (Uncomplicated Firewall)**

(there are also others like Shorewall, Ferm, ...)

Above tools offer various levels of complexity and features for configuring firewall. 
Some provide simple interface for basic firewall configuration, 
while others offer more advanced capabilities for managing complex firewall scenarios.

### IPtables
IPtables was basic tool to manage packet filtering, but newer Linux versions
provide other "front-end" tools for `iptables`.

Nftables is new tool meant to replace the aging iptables. 
Some distributions today are already moved to `nftables` and enable it by default. 
Others still keep enabled `iptables`, but have also already `nftables` present, which you need to enable manually.
We will shortly talk about 'nftables' later below.

As we said above in modern Linux versions there exist distribution-dependent 
higher-level "front-end" tools on top of iptables/nftables. 
For example **RH/CentOS** from version 7 and above come with an alternative service called `firewalld`
which fulfills this same purpose & **Ubuntu** versions now use `ufw` (Uncomplicated Firewall).

Also CentOS versions may have `iptables` as special package/service, 
with some predefined chains/rules.

> WHAT YOU NEED TO UNDERSTAND AND REMEMBER IS THAT PACKET **FILTERING ITSELF** IS KERNEL-BASED FUNCTION - **NETFILTER**
> ALL THE ABOVE TOOLS ARE JUST MEANS OF MANAGING THAT INSTRUMENT INSIDE LINUX KERNEL.

Now let's ensure we have clear config for learning we can check and stop/disable such services if any:

Check if we have have `iptables` service running/enabled:
```bash
systemctl is-active iptables ; systemctl is-enabled iptables
```
Disable/Stop it if needed:
```bash
systemctl --now disable iptables ; systemctl is-active iptables ; systemctl is-enabled iptables
```

Also check if we have have `firewalld` service running/enabled:
```bash
systemctl is-active firewalld ; systemctl is-enabled firewalld
```
Disable/Stop it if needed:
```bash
systemctl --now disable firewalld ; systemctl is-active firewalld ; systemctl is-enabled firewalld
```

Now we should have a clean initial configuration to start learning.

First we will get understanding of `iptables`, since it's anyway remaining
at the bottom of any modern netfilter-based Linux firewall.

When a packet passes through `iptables`, it passes a set of **chains**. 
Each chain contains set of **rules**.
Decisions what to do with packet is made by those **rules**.

Below command will give the list of all current active **chains** & **rules**. 
```bash
iptables -nL
```

More informative command is with `-v` added, which will also give information about traffic flows.
```bash
iptables -vnL
```


![iptables_chains.png](../linux_training/level3-2025-Additional/iptables_chains.png)


_(taken from: https://jensd.be/343/linux/forward-a-tcp-port-to-another-ip-or-port-using-nat-with-iptables)_

Basic chains are (plus PREROUTING, POSTROUTING):

* INPUT - for packets coming **into** the network interface from outside.


* FORWARD - for packets **transiting** between two network interfaces.


* OUTPUT - for packets going **out** from network interface to outside.

For each chain a sequence of **rules** with appropriate **actions** can be defined.

Each rule can specify filtering parameters (like Source/Destination IP address)
and if packet fits the **action** for that rule will be taken.

Basic actions are: 
* ACCEPT
* REJECT
* DROP

For each chain there is **default action** - final decision what to do with packets that did not fit any rule in that chain. 

Standard default action is **ACCEPT**.

<br>
<br>

#### PRACTICE

Add rule to INPUT chain:<br>
```bash
iptables -A INPUT -d 127.0.0.2 --jump REJECT
```

Check: `iptables -vnL`

Try if it works:
```bash
ping -c 3 127.0.0.2
```

Check again: `iptables -vnL` <br>
You should see 3 more packet (in **pkts** column) filtered for that rule.


Now remove that rule:
```bash
iptables -D INPUT -d 127.0.0.2 --jump REJECT 
``` 
It is also possible to remove all rules: 
```bash
iptables -F
````

Check:<br> `iptables -vnL`

Try if it works:
```bash
ping -c 3 127.0.0.2
```

We can allow only outgoing traffic.
Here we specify **default** rules with `-P` option:
```bash
iptables -P INPUT DROP ; iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
```

Last rule allows only those packets, which are parts of some already established session.

Check:
```bash
ping -c 2 8.8.8.8
```

```bash
iptables -vnL 
```

ping should work and you should see increase in number of 'pkts' for "RELATED,ESTABLISHED" chain

```bash
ping -c 2 127.0.0.1
```

ping should not work, take longer time and exit without success.

And you should see increase in number of packets for default INPUT 'polycy DROP'

```bash
iptables -vnL 
```

> * Can you explain the reason of this difference?

Now we can clear (flush) all rules<br> and restore default actions for INPUT & OUTPUT chains:
```bash
iptables -P INPUT ACCEPT  ; iptables -F 
```

And check the difference:
```bash
iptables -vnL
```

```bash
ping -c2 8.8.8.8
```

```bash
ping -c2 127.0.0.1
```

We can block some specific IP-address or subnet
```bash
iptables -A INPUT -s 8.8.8.8/16 -j DROP
```

Check:
```bash
iptables -vnL
```

```bash
ping -c 2 8.8.8.8
```

```bash
ping -c 2 8.8.4.4
```

> Can you explain:
> *  why ping doesn't work, when we only restricted INPUT ?
> *  why DROP causes hanging, but REJECT not ?


Clear the rules:

```bash
iptables -F
```

Check:

```bash
iptables -vnL
```

Now everything should work

```bash
ping -c 2 8.8.8.8
```

```bash
ping -c 2 8.8.4.4
```

Now we set rules for OUTPUT chain

```bash
iptables -F ; iptables -A OUTPUT -d 8.8.8.8/16 -j DROP
```

Check:
```bash
iptables -vnL 
```

```bash
ping -c 2 8.8.8.8
```

```bash
ping -c 2 8.8.4.4
```

> Can you explain:
> * the difference of restricting only INPUT or OUTPUT ?
> * is one of them enough, or both are needed ?


Clear the rules:

```bash
iptables -F
```


We can block some port (note that the `-p` protocol option is required 
for ports)
```bash
iptables -A OUTPUT -p tcp --dport 80 -j DROP
```

Check:
```bash
iptables -vnL 
```

 
Try:
```bash
curl -v fb.com
```


> * Change the rule to filter in INPUT chain


```bash
iptables -A INPUT -p tcp --dport 22 -j REJECT
```
 
Try:
```bash
ssh 127.0.0.1 
```

> * What the difference will be if we set 'DROP' instead of 'REJECT'

Clear:
```bash
iptables -F
```

We can combine multiple options in rules, and also 
specify the network interface <br>(`-i` for _incoming_ , `-o` for _outgoing_).

```bash
iptables -A OUTPUT -o lo -p icmp -d 127.1.2.3/24 --icmp-type echo-request -j REJECT --reject-with icmp-host-prohibited 
```
Try:
```bash
ping -c 2 127.1.2.3
```

```bash
iptables -A INPUT -i enp0s3 -p icmp -s 9.9.9.9 --icmp-type echo-reply -j DROP
```

```bash
ping -c 2 9.9.9.9
```

> NOTE: 
> 1. we filter once for INPUT and another time for OUTPUT 
> 2. depending on the expected packet we use different ICMP types:
>    * `--icmp-type echo-request` - for OUTPUT
>    * `--icmp-type echo-reply` - for INPUT


> ICMP error messages that can be added if **REJECT** method is used:<br>
>* `--reject-with icmp-host-prohibited`
>* `--reject-with icmp-net-prohibited`
>* `--reject-with icmp-net-unreachable`
>* `--reject-with icmp-host-unreachable`


We can limit the number of connections per IP address (uses **connlimit** module)
Here we allow only 1 SSH connection per IP address:

```bash
iptables -A INPUT -p tcp --syn -d 127.0.0.1 --dport 22 -m connlimit --connlimit-above 1 -j REJECT 
```
Now try connecting with ssh twice.
Login first time:
```bash
ssh student@127.0.0.1
```

Now connect second time:
```bash
ssh student@127.0.0.1
```

But below will work both times, since we block only 127.0.0.1

Login first time:
```bash
ssh student@127.0.0.2
```
Now connect second time:
```bash
ssh student@127.0.0.2
```


Clear:
```bash
iptables -F
```


Check:
```bash
iptables -vnL 
```


Multiple ports can be blocked in ine rule with **multiport** module
Here we block Microsoft-DS and Netbios ports for both TCP & UDP

```bash
iptables -A OUTPUT -p tcp -m multiport --dport 80,443  -j REJECT
```

Try connecting port 80

```bash
curl -v http://fb.com
```

Try connecting port 443

```bash
curl -v https://fb.com
```

Both should not work and you should see increase in count numbers

```bash
iptables -vnL 
```


Examples with NAT

NAT is special case. To see NAT table specify table name with `-t nat`:
```bash
iptables -vnL -t nat
```

We need to set rules in special chain POSTROUTING:
```bash
iptables -t nat -A POSTROUTING -d 127.100.0.0/16 -j SNAT --to-source 127.5.5.5 
```


Try connecting to localhost with different IPs to see the effect of NAT rules
```bash
ssh student@127.100.0.20
```

Try connecting to localhost with different IPs to see the effect of NAT rules
```bash
ssh student@127.100.0.50
```

Now see where student connected from:

```bash
who
```

You should see **student** 2 logins from `127.5.5.5` instead of `127.100.0.20` & `127.100.0.50`

To clear/drop all current rules in NAT table specify table name with `-t`
```bash
iptables -F -t nat
```

### Nftables

The `nftables` is developed by **Netfilter**, the same organization that currently maintains `iptables`. 
It was created as a better variant than `iptables` and is very similar to it.
It has some improvements, for example, with `nftables` you can create both IPv4 and IPv6 rules at one place and keep them in sync.
(in case of `iptables` you had to do separate IPv6 rule config with separate tool `ip6tables`)

> In fact `nftables` replaces not only `iptables` and `ip6tables`, but also 
> `arptables` (for ARP rules) and `ebtables` (for Ethernet Bridge rules), 
> that we don't discuss here.

`nftables` has been included in the Linux kernel since 2014, (since Linux kernel 3.13)
and  it still slowly becomes more popular.

Nftables scheme is _(taken from https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks)_:

![nf-hooks.png](../linux_training/level3-2025-Additional/nf-hooks.png)

You can try to determine whether your Linux is currently includes Nftables, using the following methods:

1. Check for `nft` command:
```bash
which nft
```
If `nft` is found, it suggests NFtables package is installed (doesn't mean it is used by default)

2. Check for `nftables` config
```bash
find /etc -name "nftables*"
```
If `nftables.conf` is found, it suggests NFtables package is installed (doesn't mean it is used by default)

3. Check Service Status:

Check if `nftables` is running/enabled as service:
```bash
systemctl is-active nftables ; systemctl is-enabled nftables
```

Remember that even if `nftables` is disabled as a service you can still use 
command tools from this package to manage kernel filter.
The service itself only manages its configuration, not the filter itself.

Even more the rules you set with `netfilter` and `iptables` are to some extent managable by another tool.

#### Syntax difference between iptables and nftables
The syntax of `nftables` is different than syntax of `iptables`.

But there is `iptables-translate` utility, which will accept `iptables` options and convert them to the `nftables` equivalent. 
This is an easy way to see how the two syntaxes differ.

Let’s see some examples so that you can see how these commands differ from each other.

This command would block incoming connections from IP address `127.1.2.7`:
```bash
iptables-translate -A INPUT -s 127.1.2.7 -j DROP
```

You can now run the result of the above command.
and check if that rule has been added:

```bash
iptables -vnL | grep '127.1.2.7'
```

Also we can remove it with `iptables`
```bash
iptables -D INPUT -s 127.1.2.7 -j DROP
```

and check if that rule has been removed:
```bash
iptables -vnL | grep '127.1.2.7'
```

More examples:

Allow incoming SSH connections:
```bash
iptables-translate -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
```
You can now run the result of the above command.
and check if that rule has been added:

```bash
iptables -vnL | grep '22'
```

You can notice warning:
`table `filter' is incompatible, use 'nft' tool.`

So you can use the `nft` tool
```bash
nft list ruleset
```


Or with more details:

```bash
nft list table ip filter
```

To clear again use

```bash
iptables -F
```

This demonstrates that in real life it is not so smooth to use one tool instead of another.
This brings us to
> IMPORTANT NOTE:
> To prevent the different firewalling/packet filtering services/tools from influencing each other, 
> run only one of them per Linux host, and disable the other services.


More info:
* https://access.redhat.com/documentation/ru-ru/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-nftables_configuring-and-managing-networking
* https://linuxhandbook.com/iptables-vs-nftables/
* https://netfilter.org/projects/nftables/
* https://habr.com/ru/companies/ruvds/articles/580648/
* https://www.server-world.info/en/note?os=Ubuntu_22.04&p=nftables&f=2


### FirewallD

For **Red Hat**-based systems (including **AlmaLinux**, **Rocky Linux**), 
the default firewall tool is **firewalld**.

**FirewallD** is a dynamic firewall manager that uses either `iptables` or `nftables` as its backend. 

It provides a higher-level, more user-friendly interface 
with concepts like **Zones**, **Services**, **Ports**, ...

You can start and enable it with:

```bash
systemctl enable --now firewalld
```

Check the status with `firewall-cmd`, which is **FirewallD** main command.

```bash
firewall-cmd --state
```

> Remember that behind **FirewallD** there is still `iptables`, <br>
> so if firewalld is running you can also check the rules with
> `iptables -vnL`


#### Understanding the Basics

**FirewallD** uses **Zones** to manage the trust level of your network connections. 
Think of a zone as a **security profile**.

In default configuration there are 10 zones:
`block`, `dmz`,  `drop`,  `external`,  `home`,  `internal`, `nm-shared`, `public`, `trusted`, `work`

```bash
firewall-cmd --get-zones
```

EXAMPLE - imagine 
* **FIREWALLD** is OFFICE BUILDING 
* **ZONES** are Floors
* **SERVICES**/**PORTS** are Rooms
* **NETWORK INTERFACES** are Doors

Not all zones are **active** - working

**ACTIVE ZONE** has network interface activated/assigned (so real traffic/packets flow is managed by that zone),
i.e. there is a **DOOR** at to enter that **FLOOR**

By default only one **FLOOR** - `public` zone has **DOORS** - network interfaces

So you don't need to remember all at once, only the first floor !

Following diagram shows the **FIREWALLD** structure:

<pre>
┌─────────────────────────────────────────────────────────────────┐
│                     FIREWALLD "OFFICE BUILDING"                 │
│            Trust level decreases from top to bottom             │
└─────────────────────────────────────────────────────────────────┘

 10th Floor ┌─────────────────────────────────────────────────────┐
    🛡️      │ TRUSTED ZONE: "Full Access"                         │
   trusted  │ target: ACCEPT (Everything allowed)                 │
            │                                                     │
            │  🚪 ALL DOORS OPEN - NO RESTRICTIONS                 │
            │  Services: * (all services automatically allowed)   │
            │  Ports: * (all ports automatically open)            │
            │                                                     │
            │  CEO Office - complete freedom                      │
            └─────────────────────────────────────────────────────┘

 9th Floor  ┌─────────────────────────────────────────────────────┐
   🔒       │ BLOCK ZONE: "Hard Block"                            │
   block    │ target: %%REJECT%% (Everything rejected)            │
            │                                                     │
            │  🚫 ALL DOORS LOCKED - "ACCESS DENIED"              │
            │  Services: (none)                                   │
            │  Ports: (none)                                      │
            │                                                     │
            │  Server Room - strictly forbidden                   │
            └─────────────────────────────────────────────────────┘

 8th Floor  ┌─────────────────────────────────────────────────────┐
   🚫       │ DROP ZONE: "Complete Ignore"                        │
   drop     │ target: DROP (Packets disappear)                    │
            │                                                     │
            │  🚫 INVISIBLE DOORS - NO RESPONSE                   │
            │  Services: (none)                                   │
            │  Ports: (none)                                      │
            │                                                     │
            │  Archive - silent treatment                         │
            └─────────────────────────────────────────────────────┘

 7th Floor  ┌─────────────────────────────────────────────────────┐
   🏢       │ WORK ZONE: "Work Network"                           │
   work     │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ ssh         (remote access)                     │
            │  ├─ dhcpv6-client (auto-config)                     │
            │  ├─ samba-client (file sharing)                     │
            │  └─ cockpit      (management)                       │
            │                                                     │
            │  Development Department - work essentials           │
            └─────────────────────────────────────────────────────┘

 6th Floor  ┌─────────────────────────────────────────────────────┐
   🏠       │ HOME ZONE: "Home Network"                           │
   home     │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ ssh         (remote access)                     │
            │  ├─ mdns        (service discovery)                 │
            │  ├─ samba-client (file sharing)                     │
            │  ├─ dhcpv6-client (auto-config)                     │
            │  ├─ ipp-client  (printing)                          │
            │  └─ cockpit      (management)                       │
            │                                                     │
            │  Break Room - home services available               │
            └─────────────────────────────────────────────────────┘

 5th Floor  ┌─────────────────────────────────────────────────────┐
   🏢       │ INTERNAL ZONE: "Internal Network"                   │
   internal │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ ssh         (remote access)                     │
            │  ├─ mdns        (service discovery)                 │
            │  ├─ samba-client (file sharing)                     │
            │  ├─ dhcpv6-client (auto-config)                     │
            │  ├─ cockpit      (management)                       │
            │  └─ mysql        (database)                         │
            │                                                     │
            │  Internal services - corporate network              │
            └─────────────────────────────────────────────────────┘

 4th Floor  ┌─────────────────────────────────────────────────────┐
   🌐       │ EXTERNAL ZONE: "External Network"                   │
   external │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ ssh         (remote access)                     │
            │                                                     │
            │  🛡️ MASQUERADE ENABLED (IP hiding)                  │
            │                                                     │
            │  External Relations - minimal exposure              │
            └─────────────────────────────────────────────────────┘

 3rd Floor  ┌─────────────────────────────────────────────────────┐
   🛡️       │ DMZ ZONE: "Demilitarized Zone"                      │
   dmz      │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ http        (web service)                       │
            │  ├─ https       (secure web)                        │
            │  └─ ssh         (limited remote access)             │
            │                                                     │
            │  Public servers - restricted services               │
            └─────────────────────────────────────────────────────┘

 2nd Floor  ┌─────────────────────────────────────────────────────┐
   📱       │ NM-SHARED ZONE: "Shared Network"                    │
 nm-shared  │ target: default                                     │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ ssh         (remote access)                     │
            │  ├─ dhcpv6-client (auto-config)                     │
            │  ├─ dns         (name resolution)                   │
            │  └─ ipp-client  (printing)                          │
            │                                                     │
            │  Guest Wi-Fi - basic connectivity                   │
            └─────────────────────────────────────────────────────┘

 1st Floor  ┌─────────────────────────────────────────────────────┐
   🌍       │ PUBLIC ZONE: "Public Access" ← ACTIVE ZONE          │
   public   │ target: default                                     │
            │                                                     │
            │  TWO DOORS:                                         │
            │  🚪 enp0s3 (main DOOR)                               │
            │  🚪 enp0s8 (backup DOOR)                             │
            │                                                     │
            │  🚪 AVAILABLE ROOMS:                                 │
            │  ┌─ cockpit - port 9090 (web-based management tool) │
            │  ├─ dns   - port 53  (domain name system)           │
            │  ├─ http  - port 80  (unencrypted web traffic)      │
            │  ├─ https - port 443 (encrypted web traffic)        │
            │  └─ ssh   - port 22  (for remote administration)    │
            │                                                     │
            │  Everything else is locked!                         │
            └─────────────────────────────────────────────────────┘


</pre>

In the diagram above `target:` means general rule for the entire "FLOOR". 
`target: default` means **everything that is not permitted is not allowed**


#### FirewallD Configuration Runtime vs Permanent

**FirewallD** configuration is of two types:
* Runtime - current working configuration
* Permanent -  what will be applied after reboot

If you want to make the change permanent, you can either run 

* run `firewall-cmd --permanent <RULE>...` to make that rule permanent
* then run `firewall-cmd --reload` to activate it in runtime configuration

Or

* run `firewall-cmd <RULE>...` (without `--permanent`) instantly to activate the rule in runtime configuration
* then run `firewall-cmd --runtime-to-permanent` to make runtime configuration permanent

#### Basic commands

See the **default zone**

Almost always it will be `public` zone.

```bash
firewall-cmd --get-default-zone
```

See all active zones and what interfaces are assigned to them

```bash
firewall-cmd --get-active-zones
```

List everything for active zones (services, active rules)

```bash
firewall-cmd --list-all
```

If we want to see for some **not active zone**, we can add `--zone=` option

```bash
firewall-cmd --list-all --zone=block
```

```bash
firewall-cmd --list-all --zone=drop
```

Now let us permanently allow the HTTP service

```bash
firewall-cmd --permanent --add-service=http ;  firewall-cmd --reload
```

> The  `--permanent` flag means the rule survives a reboot.
> However, rules with `--permanent` are **NOT** active immediately.
> That is why we give the second `reload` to activate the permanent rule

Now verify the service was added

```bash
firewall-cmd --list-all
```

Now let us allow the HTTPS service, but in another way.
First add it in runtime configuration

```bash
firewall-cmd --add-service=https
```

Check that it is there

```bash
firewall-cmd --list-all
```

Check that it IS NOT in the permanent configuration

```bash
firewall-cmd --list-all --permanent
```

Now add it to permanent config too

```bash
firewall-cmd --runtime-to-permanent
```

Check

```bash
firewall-cmd --list-all --permanent
```


Open a custom port (8080/TCP)

```bash
firewall-cmd --add-port=8080/tcp
```

Check

```bash
firewall-cmd --list-all
```

Check that it IS NOT in the permanent configuration

```bash
firewall-cmd --list-all --permanent
```

Now add it to permanent config too

```bash
firewall-cmd --runtime-to-permanent
```

Check again. It should be there.

```bash
firewall-cmd --list-all --permanent
```


Now, let us remove the port rule

```bash
firewall-cmd --permanent --remove-port=8080/tcp
```

Check it is removed from **permanent** config

```bash
firewall-cmd --list-all --permanent
```

But it is still in **runtime** config

```bash
firewall-cmd --list-all
```

Reload **runtime** config

```bash
firewall-cmd --reload
```

Check

```bash
firewall-cmd --list-all
```

> RECOMMENDATION:
> Adding with **port** is less effective, because **firewalld** has more useful configuration for many **services**.
> And if we add **service** instead **port**, we can for example at once enable `TCP` and `UDP` and other important config.
> So it is recommended to use **service** where possible and use **port**, if there is a need to configure some custom port.


To see what **services** are configured in **firewalld** you can run 

```bash
firewall-cmd --get-services
```

That config files are located here:

```bash
ls /usr/lib/firewalld/services/
```

Details for service configuration can be found with command:

```bash
firewall-cmd --info-service=dns
```

```bash
firewall-cmd --info-service=smtp
```

```bash
firewall-cmd --info-service=smtps
```

```bash
firewall-cmd --info-service=imap
```

```bash
firewall-cmd --info-service=imaps
```

Enable Mail-related service ports

```bash
firewall-cmd --add-service={smtp,smtps,pop3,pop3s,imap,imaps}
```

Check if that ports are now in the current run-time config:

```bash
firewall-cmd --list-all
```

Check if they are also in the permanent config:

```bash
firewall-cmd --list-all --permanent
```

> WHY NOT ?

Now add it to permanent config too

```bash
firewall-cmd --runtime-to-permanent
```

Check again

```bash
firewall-cmd --list-all --permanent
```


#### PRACTICE

Add service `dns` to default public zone and make that config permanent.

> Remember you can do it in any of 2 ways.

Show that your changes are both in **runtime** and **permanent** configurations.


#### Some useful FirewallD docs

* https://www.redhat.com/sysadmin/beginners-guide-firewalld

* https://access.redhat.com/documentation/ru-ru/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-and-configuring-firewalld_configuring-and-managing-networking

* https://www.server-world.info/en/note?os=CentOS_Stream_8&p=firewalld&f=1

* https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-8

* https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-8-ru



### UFW

UFW is also the frontend tool of nftables/iptables for Debian/Ubuntu.

UFW Basic Operation is available at:

* https://www.server-world.info/en/note?os=Ubuntu_22.04&p=ufw&f=1
* https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-22-04
* https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands

