# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)


## Cockpit 

**Cockpit** is a web-based graphical interface for administrating Linux servers.
It is available for most modern Linux versions.

**Cockpit** provides a easy to use graphical interface to remote Linux servers.
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

For Ubuntu/Debian you can run
```bash
apt update ;\
apt -y install cockpit

```


That's it. 
Now check Cockpit is there

```bash
ss -nlpt ; grep 9090

```

And if so, try access with web browser:
(you need to ignore security warning, since the SSL certificate is self-signed)

`https://[ipaddress]:9090`

