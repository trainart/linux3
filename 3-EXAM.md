# Linux Network Server (level 3) EXAM

## Install following service on separate VM and present as your EXAM result

### Before you start with exam task on new VM 

1. Since `SELinux` and `firewalld` may cause problems for this exam task
turn off firewalld & SELinux

2. Teacher will group you per VM and assign a domain for your group
   * `l3exam1.am`
   * `l3exam2.am`
   * `l3exam3.am`
   * ...
   
   Set proper VM **hostname** according to your group domain.


###  Network configuration

Add second network interface to you VM and configure static IP address
`10.10.1.1`, `10.10.2.1`, ... (third number should be your group number) 

###  DNS Server

1. Install and configure chrooted BIND DNS Server
2. Configure bind-chroot to 
   1. listen-on port 53 `any` IP addresses
   2. allow-query to `any` IP addresses 
3. Create zone for you group domain name `l3exam1.am`, ... 
     with following records
   * `l3exam1.am` NS ns.l3exam1.am
   * `l3exam1.am` MX 0 mail
   * `ns.l3exam1.am` A 10.10.1.1
   * `www.l3exam1.am` A 10.10.1.1
   * `mail.l3exam1.am` A 10.10.1.1

4. Configure static resolver to 127.0.0.1

   * Configure your `/etc/resolv.conf` to refer `nameserver 127.0.0.1`. 
   * Make it `immutable` and not changeable by NetworkManager


5. Prepare exam output

```bash
host -t ns l3exam1.am  >> /tmp/l3exam1-dns.out ;\
host -t mx l3exam1.am  >> /tmp/l3exam1-dns.out ;\
host -t a l3exam1.am  >> /tmp/l3exam1-dns.out ;\
host -t a ns.l3exam1.am  >> /tmp/l3exam1-dns.out ;\
host -t a www.l3exam1.am  >> /tmp/l3exam1-dns.out ;\
host -t a mail.l3exam1.am  >> /tmp/l3exam1-dns.out
```

### Install and configure Apache Web Server

1. Install Apache and related stuff 
   * Change Apache to listen port `8080` instead `80`
   * Create separate virtual host for `l3exam1.am` website
   * Put some index page there with word `APACHE`
   
2. Prepare exam output

```bash
curl -v  http://www.l3exam1.am:8080/ | grep APACHE >> /tmp/l3exam1-apache.out

```

### Install and configure NGINX Web Server

1. Install Nginx:  
   * Change Nginx to listen port `8088` instead `80`
   * Create separate virtual host for `l3exam1.am` website 
   * Put some index page there with word `NGINX`

`2. Prepare exam output

```bash
curl -v  http://www.l3exam1.am:8088/ | grep NGINX >> /tmp/l3exam1-nginx.out

```
`
### Install and configure HAProxy

1. Install and configure HAProxy to work in the environment like follows.

```bash
--------+---------------------+----------------------+------------
        |                     |                      |
        |[IP address]:80      |[IP address]:8080     |[IP address]:8088
+-------+--------+   +--------+----------+   +--------+----------+
|  [l3exam1.am]  |   | [l3exam1.am:8080] |   | [l3exam1.am:8088] |
|     HAProxy    |   | Apache Server #1  |   |  Nginx Server#2   |
+----------------+   +-------------------+   +-------------------+

```

2. Prepare exam output

```bash
netstat -nlpt | grep -E '(haproxy|http|nginx)' >> /tmp/l3exam1-webservers.out

```

```bash
curl -s http://l3exam1.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam1-haproxy.out ; \
curl -s http://l3exam1.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam1-haproxy.out ; \
curl -s http://l3exam1.am/ | grep -E '(APACHE|NGINX)' >> /tmp/l3exam1-haproxy.out

```

### Squid 

1. Install Squid
2. Hide HTTP headers that reveal you are behind the proxy
3. Prepare exam output

```bash
curl -s -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected" ;\
curl -s -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected" >> /tmp/l3exam1-squid.out

```

### Install and configure Mail server (Postfix, Dovecot)

1. Install Postfix & Dovecot 
2. Configure for `l3exam1.am` domain
3. Create user `exam@l3exam1.am`
4. Send test email to `exam@l3exam1.am`
6. Install `pflogsumm` log reporting tool
5. Prepare exam output


```bash
perl /usr/sbin/pflogsumm -d today /var/log/maillog >> /tmp/l3exam1-mail.out ;\
cat /tmp/l3exam1-mail.out

```


### After you finish

Present your results 
```bash
tar zcvf l3exam1.tgz /tmp/l3exam1*

```

