# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Web Server (Nginx) (+ HAProxy) 


Nginx is another widely used webserver. 
Nginx is known for its high performance and low resource. 
Nginx can work as a standalone webserver or as front end reverse proxy of Apache web server, which will increase the performance.

If we have Apache server installed on the same Linux server, we need to ensure Apache and Nginx are running on different ports.
For example, we can change Apache config from default `80` port to `8080`. 

```bash
sed -i 's/Listen 80/Listen 8080/g' /etc/httpd/conf/httpd.conf
```


```bash
sed -i 's/*:80/*:8080/g' /etc/httpd/conf.d/lt0x.am.conf
```



Restart Apache:
```bash
systemctl restart httpd
```

Check that Apache listens port 8080: 
```bash
netstat -nlpt | grep httpd
```

Install Nginx:  
```bash
yum -y install nginx
```
Enable & start Nginx: 
```bash
systemctl enable --now nginx
```


#### PRACTICE
Now you should have running Apache webserver on port 8080, and Nginx on port 80

* Change Apache configuration to work on another port - 8888. After that you should be able to access `http://lt0x.am:8888/`
* Change it back to 8080

### Nginx configuration as front end reverse proxy of Apache web server

![img_2.png](img_2.png)

Create virtual host config file `/etc/nginx/conf.d/lt0x.am.conf` for our domain `lt0x.am`
```bash
server {
listen *:80;
access_log /var/log/nginx/lt0x.am-access.log;
error_log /var/log/nginx/lt0x.am-error.log;
server_name lt0x.am www.lt0x.am;
# Redirect to backend
location / {
proxy_pass http://127.0.0.1:8080/;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_connect_timeout 120;
proxy_send_timeout 120;
proxy_read_timeout 180;
}
# Serve static pages with nginx
location ~* \.(jpg|jpeg|gif|png|ico|css|bmp|swf|js|html|txt)$ {
root /var/www/lt0x.am;
}
}

```
Ensure SELinux is tuned off (to allow redirection from Nginx to Apache): 
```bash
setenforce 0
```
Restart Nginx: 
```bash
systemctl restart nginx
```

Check that Nginx listens port 80: 
```bash
netstat -nlpt | grep nginx
```

Check the logs of both Nginx & Apache (open in different terminals to see simultaneously): 
```bash
tail -f /var/log/nginx/lt0x.am-access.log
tail -f /var/log/httpd/lt0x.am-access.log
```

Now try opening some URLs at Nginx and you should see in above both logs redirection to Apache server. 
```bash
links lt0x.am
links lt0x.am/inf.php
```

> REMINDER: the above will work only if: 
> * you have local DNS server, with properly configured zone `lt0x.am` 
 and A record `lt0x.am` pointing to your server's IP address
> * and your `/etc/resolv.conf` refers to `nameserver 127.0.0.1` (or better to Teachers DNS that should have all domains in one)

### Nginx configuration as standalone server

We need to **change** configuration of our virtual host as follows

**REPLACE** file `/etc/nginx/conf.d/lt0x.am.conf`
with new one with text:

```bash
server {
listen *:80;
access_log /var/log/nginx/lt0x.am-access.log;
error_log /var/log/nginx/lt0x.am-error.log;
server_name lt0x.am www.lt0x.am;
# Redirect to backend
location / {
root /var/www/lt0x.am-nginx;
}
}
```

Create new `/var/www/nginx.lt0x.am` directory for Nginx virtual host


Create `/var/www/lt0x.am-nginx/index.html` index page there:
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
links lt0x.am
```

and

```bash
links lt0x.am:8080
```

You should see appropriate logs.


### HAProxy : HTTP Load Balancing
_(based on `https://www.server-world.info/en/note?os=CentOS_8&p=haproxy&f=1`)_
	
HAProxy allows to balance between multiple servers. Simple configuration of two servers we have Apache and Nginx follows.

This example is based on the environment like follows.
```bash
--------+---------------------+----------------------+------------
        |                     |                      |
        |10.10.x.1:80         |10.10.x.1:8080        |10.10.x.1:8088
+-------+--------+   +--------+---------+   +--------+---------+
|   [ lt0x.am ]  |   | [ lt0x.am:8080 ] |   | [ lt0x.am:8088 ] |
|     HAProxy    |   | Apache Server #1 |   |  Nginx Server#2  |
+----------------+   +------------------+   +------------------+

```

We already have Apache configured to listed port `8080`, 
but we need Nginx to be reconfigured to listen port `8088` as shown on scheme above.
this way we will free port `80` for HAProxy balancer.

Reconfigure Nginx to listen port `8088` instead `80`

In file `/etc/nginx/nginx.conf` change the following lines:
```bash
#        listen       80 default_server;
#        listen       [::]:80 default_server;
listen       8088 default_server;
listen       [::]:8088 default_server;
```
 
**REPLACE** file `/etc/nginx/conf.d/lt0x.am.conf` with another port:
```bash
server {
listen *:8088;
access_log /var/log/nginx/lt0x.am-access.log;
error_log /var/log/nginx/lt0x.am-error.log;
server_name lt0x.am www.lt0x.am;
# Redirect to backend
location / {
root /var/www/lt0x.am-nginx;
}
}
```

Restart Nginx:

```bash
systemctl restart nginx
```

Check that Nginx listens port 8088:

```bash
netstat -nlpt | grep nginx
```

Install HAProxy to configure Load Balancing Server.

```bash
yum -y install haproxy
```

Move default HAProxy config and create simple configuration.

```bash
mv /etc/haproxy/haproxy.cfg{,.backup} ;\
cat << "EOF1" > /etc/haproxy/haproxy.cfg
# define frontend ( any name is OK for [http-in] )
frontend http-in
    # listen 80 port
    bind *:80
    # set default backend
    default_backend    backend_servers
    # send X-Forwarded-For header
    option             forwardfor

# define backend
backend backend_servers
    # balance with roundrobin
    balance            roundrobin
    # define backend servers
    server             node01 127.0.0.1:8080 check
    server             node02 127.0.0.1:8088 check
EOF1

```

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
curl -s http://lt0x.am/ | grep -E '(APACHE|NGINX)' ; \
curl -s http://lt0x.am/ | grep -E '(APACHE|NGINX)' ; \
curl -s http://lt0x.am/ | grep -E '(APACHE|NGINX)' 
```


> NOTE: this is very simple configuration example. Production solution requires much more to configure as well as
> additional services like `keepalived`, `vrrp`, etc., which are our of scope of this tutorial
