# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Web Server (Nginx) (+ HAProxy) 


Nginx is another widely used webserver. 
Nginx is known for its high performance and low resource. 
Nginx can work as a standalone webserver or as front end reverse proxy of Apache web server, which will increase the performance.

If we have Apache server installed on the same Linux server, <br>
we need to ensure Apache and Nginx are running on different addresses.


#### PRACTICE

1. Now add new `A` DNS records `nginx.lt0x.am` &  `apache.lt0x.am` for your domain 

* `apache.lt0x.am` A resource record:
  * name:   `apache` 
  * type:   `A` 
  * value:  `10.10.x.100`

* `nginx.lt0x.am` A resource record:
  * name:   `nginx` 
  * type:   `A` 
  * value:  `10.10.x.200`

 
2. Use 'nmtui' to assign additional static IPs 
   1. `10.10.x.100` 
   2. `10.10.x.200` 

to you second interface `enp0s8` 




Change Apache config to listen port `80` only for ip address `10.10.x.100`. 

```bash
sed -i 's/Listen 80/Listen 10.10.x.100:80/g' /etc/httpd/conf/httpd.conf
```

```bash
sed -i 's/*:80/10.10.x.100:80/g' /etc/httpd/conf.d/lt0x.am.conf
```

Restart Apache:
```bash
systemctl restart httpd
```

Check that Apache listens only port 80 of that IP: 
```bash
netstat -nlpt | grep :80
```

Try opening: 
```bash
lynx http://apache.lt0x.am
```


#### Install Nginx

Install Nginx:  
```bash
yum -y install nginx
```

Enable & start Nginx: 
```bash
systemctl enable --now nginx
```

> It should give ERROR. WHY?


Change default config to listen only port `80` of IP `10.10.x.200`

```bash
sed -i 's/listen       80/listen       10.10.x.200:80/g' /etc/nginx/nginx.conf
```

Try starting Nginx again: 
```bash
systemctl restart nginx
```

Check that Nginx listens another IP's port 80: 
```bash
netstat -nlpt | grep :80
```

Try opening: 
```bash
lynx http://nginx.lt0x.am
```


#### Hide Nginx version

First check current state:

```bash
curl -sI http://nginx.lt0x.am | grep Server
```

```bash

cat << "EOF1" > /etc/nginx/conf.d/tokens-off.conf
server_tokens off;
EOF1

```

Restart Nginx: 
```bash
systemctl restart nginx
```

And check again:
```bash
curl -sI http://nginx.lt0x.am | grep Server
```



Create configuration of `nginx.lt0x.am` virtual host:

Add new file `/etc/nginx/conf.d/nginx.lt0x.am.conf`
with new one with text:

```bash
server {
listen 10.10.x.200:80;
access_log /var/log/nginx/nginx.lt0x.am-access.log;
error_log /var/log/nginx/nginx.lt0x.am-error.log;
server_name nginx.lt0x.am;
location / {
root /var/www/nginx.lt0x.am;
}
}
```

> REMEMBER to change `x` in every `lt0x.am` with your number.<br>
> You can do that with commands like<br>
> `sed -i 's/lt0x/lt00/g' /etc/nginx/conf.d/nginx.lt0x.am.conf`<br>
> But change `lt00` to your number before running it


Create new `/var/www/nginx.lt0x.am` directory for Nginx virtual host

```bash
mkdir /var/www/nginx.lt0x.am
```

Create `/var/www/nginx.lt0x.am/index.html` index page there:
with text:
```bash
HI this is NGINX page
```

Restart Nginx: 

```bash
systemctl restart nginx
```

Now try opening some `Nginx` URL and `Apache` URL. 
```bash
links nginx.lt0x.am
```

and

```bash
links apache.lt0x.am
```

You should see appropriate logs.
Check the logs of both Nginx & Apache (open in different terminals to see simultaneously): 
```bash
tail -f /var/log/nginx/lt0x.am-access.log
tail -f /var/log/httpd/lt0x.am-access.log
```

### HAProxy : HTTP Load Balancing
_(based on `https://www.server-world.info/en/note?os=CentOS_8&p=haproxy&f=1`)_
	
HAProxy allows to implement load balancing between multiple servers. <br>
Simple configuration of two servers Apache and Nginx follows.

This example is based on the environment like follows.
```bash
--------+---------------------+----------------------+------------
        |                     |                      |
        |10.10.x.10:80        |10.10.x.100:80        |10.10.x.200:80
+-------+--------+   +--------+---------+   +--------+---------+
| [ ha.lt0x.am ] |   | [apache.lt0x.am] |   | [ nginx.lt0x.am] |
|     HAProxy    |   | Apache Server #1 |   |  Nginx Server#2  |
+----------------+   +------------------+   +------------------+

```

We already have Apache & Nginx configured.


1. Now add new `A` DNS records `ha.lt0x.am` for your domain 

* `ha.lt0x.am` A resource record:
  * name:   `ha` 
  * type:   `A` 
  * value:  `10.10.x.10`
  
2. Use 'nmtui' to assign additional static IP
   1. `10.10.x.10` 
to you second interface `enp0s8` 

Install HAProxy to configure Load Balancing Server.

```bash
yum -y install haproxy
```

Move default HAProxy config and create simple configuration.

```bash
mv /etc/haproxy/haproxy.cfg{,.backup}
```

Create new `/etc/haproxy/haproxy.cfg` file:
```bash
# define frontend ( any name is OK for [http-in] )
frontend http-in
    # listen 80 port
    bind 10.10.x.10:80
    # set default backend
    default_backend    backend_servers
    # send X-Forwarded-For header
    option             forwardfor

# define backend
backend backend_servers
    # balance with roundrobin
    balance            roundrobin
    # define backend servers
    server             node01 10.10.x.100:80 check
    server             node02 10.10.x.200:80 check

```

> Change `10.10.x` to you number.

Enable and start HAProxy
```bash
systemctl enable --now haproxy
```

Check the status of all related services:
```bash
netstat -nlpt | grep -E '(haproxy|http|nginx)'
```

Now you connect to `lt0x.am` several times. 
You should see Apache and Nginx pages in rotation

```bash
curl -s http://ha.lt0x.am/ | grep -E '(APACHE|NGINX)'
```

```bash
curl -s http://ha.lt0x.am/ | grep -E '(APACHE|NGINX)'
```

```bash
curl -s http://ha.lt0x.am/ | grep -E '(APACHE|NGINX)'
```


> NOTE: this is very simple configuration example. Production solution requires much more to configure as well as
> additional services like `keepalived`, `vrrp`, etc., which are out of scope of this training material.
