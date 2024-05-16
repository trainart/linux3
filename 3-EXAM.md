# Linux Network Server (level 3) EXAM

## Install following service on separate VM and present as your EXAM result

### Before you start with exam task on new VM 

1. Since `SELinux` and `firewalld` may cause problems for this exam task
turn off firewalld & SELinux

2. Teacher will group you per VM and assign a domain for your group
   * `l3exam01.am`
   * `l3exam02.am`
   * `l3exam03.am`
   * ...
   
   Set proper VM **hostname** according to your group domain.


###  Network configuration

Add second network interface to your VM. 
Use 'nmtui' to assign static IPs to second interface `enp0s8`
`10.10.x.1/24`
`10.10.x.100/32`
`10.10.x.200/32`
`10.10.x.10/32`


###  DNS Server

1. Install and configure chrooted BIND DNS Server
2. Configure bind-chroot to 
   1. listen-on port 53 `any` IP addresses
   2. allow-query to `any` IP addresses 
3. Create zone for you group domain name `l3exam0x.am`, ... 
     with following records
   * `l3exam0x.am` NS ns.l3exam0x.am
   * `l3exam0x.am` MX 0 mail
   * `ns.l3exam0x.am` A 10.10.1.1
   * `www.l3exam0x.am` A 10.10.1.1
   * `mail.l3exam0x.am` A 10.10.1.1
   * `apache.l3exam0x.am` A 10.10.1.100
   * `nginx.l3exam0x.am` A 10.10.1.200
   * `ha.l3exam0x.am` A 10.10.1.10
   

4. Configure static resolver to 127.0.0.1

   * Configure your `/etc/resolv.conf` to refer `nameserver 127.0.0.1`. 
   * Make it `immutable` and not changeable by NetworkManager


5. Prepare exam output

```bash
host -t ns l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t mx l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a ns.l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a www.l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a apache.l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a nginx.l3exam0x.am  >> /tmp/l3exam0x-dns.out ;\
host -t a mail.l3exam0x.am  >> /tmp/l3exam0x-dns.out
```

### Install and configure Apache Web Server

1. Install Apache and related stuff 
   * Change Apache config to listen port `80` only for ip address `10.10.x.100`. 
   * Create separate virtual host for `l3exam0x.am` website
   * Put some index page there with word `APACHE`
   
2. Prepare exam output

```bash
curl -v  http://apache.l3exam0x.am/ | grep APACHE >> /tmp/l3exam0x-apache.out

```

### Install and configure NGINX Web Server

1. Install Nginx:  
   * Change default config to listen only port `80` of IP `10.10.x.200`
   * Create separate virtual host for `l3exam0x.am` website 
   * Put some index page there with word `NGINX`

`2. Prepare exam output

```bash
curl -v  http://nginx.l3exam0x.am/ | grep NGINX >> /tmp/l3exam0x-nginx.out

```
`
### Install and configure HAProxy

1. Install and configure HAProxy to work in the environment like follows.

```bash
--------+---------------------+----------------------+------------
        |                     |                      |
        |10.10.x.10:80        |10.10.x.100:80        |10.10.x.200:80
+-------+--------+   +--------+---------+   +--------+---------+
| [ ha.lt0x.am ] |   | [apache.lt0x.am] |   | [ nginx.lt0x.am] |
|     HAProxy    |   | Apache Server #1 |   |  Nginx Server#2  |
+----------------+   +------------------+   +------------------+

```


2. Prepare exam output

```bash
netstat -nlpt | grep -E '(haproxy|http|nginx)' >> /tmp/l3exam0x-webservers.out

```

```bash
curl -s http://ha.l3exam0x.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam0x-haproxy.out ; \
curl -s http://ha.l3exam0x.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam0x-haproxy.out ; \
curl -s http://ha.l3exam0x.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam0x-haproxy.out

```

### Squid 

1. Install Squid
2. Hide HTTP headers that reveal you are behind the proxy
3. Prepare exam output

```bash
curl -s -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected" ;\
curl -s -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected" >> /tmp/l3exam0x-squid.out

```

### Install and configure Mail server (Postfix, Dovecot)

1. Install Postfix & Dovecot 
2. Add Rsyslog configuration to log Postfix and Dovecot logs separately.
   1. "postfix" to `/var/log/postfix.log`
   2. "dovecot" to `/var/log/dovecot.log`
3. Configure for `l3exam0x.am` domain
4. Create user `exam@l3exam0x.am`
5. Send test email from `root` to `exam@l3exam0x.am`
6. Install `pflogsumm` log reporting tool
7. Prepare exam output


```bash
/usr/sbin/pflogsumm -d today -e /var/log/postfix.log >> /tmp/l3exam0x-mail.out ;\
cat /tmp/l3exam0x-mail.out

```


### After you finish

Present your results 
```bash
tar zcvf l3exam0x.tgz /tmp/l3exam0x*

```

