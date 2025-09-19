# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)


## Cockpit - SysAdmin Web console

**Cockpit** is a web-based graphical interface for administrating Linux servers.
(Cockpit - օդաչուների խցիկ)
It is available for most modern Linux versions.

**Cockpit** provides an easy to use graphical interface to remote Linux servers.
The interface enables admins to manage things like users/groups, 
firewall settings, hardware info and more...


### Install and enable Cockpit 

```bash
yum -y install cockpit ;\
systemctl enable --now cockpit.socket

```


You may need to add following for firewalld if it is enabled

```bash
firewall-cmd --permanent --zone=public --add-service=cockpit ;\
firewall-cmd --reload

```

That's it. 
Cockpit should listen port `9090`.
Check it:

```bash
ss -nlpt | grep 9090

```

If you see that, you can now try to access Cockpit with the web browser.
You should access URL like **https://[ipaddress]:9090**

Let us get that URL with following command.

```bash
ip -4 -o a | grep -v '127.0.0.1' | awk '{print $4}' | awk -F'/' '{print $1}' | sed 's/.*/https:\/\/&:9090/'
```

> PRACTICE 
> 1. Explain the above command sequence.
> 2. If you see more than one line, suggest what to add there to get only the first URL
> 

While opening the URL, remember that you ignore browser's security warning, since the SSL certificate is self-signed.


### Try Cockpit

* Try login to Cockpit interface with `student`

* Click the above `Administrative access` button to switch to Admin privileges.

* Try several admin functions from the left menu.


