# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Web Server Nginx as Proxy 

Nginx can also work as frontend reverse proxy of other web server, which will increase the performance.

#### PRACTICE

1. Now add new `A` DNS record `reverse.lt0x.am` for your domain

* `reverse.lt0x.am` A resource record:
  * name:   `reverse` 
  * type:   `A` 
  * value:  `10.10.x.222`

 
2. Use 'nmtui' to assign additional static IPs 
   1. `10.10.x.222` 

to you second interface `enp0s8` 


### Nginx configuration as frontend reverse proxy of HA Proxy server

Create configuration of `reverse.lt0x.am` virtual host:

Add new file `/etc/nginx/conf.d/reverse.lt0x.am.conf`
with text:

```bash
server {
listen 10.10.x.222:80;
access_log /var/log/nginx/reverse-lt0x.am-access.log;
error_log /var/log/nginx/reverse-lt0x.am-error.log;
server_name reverse.lt0x.am;
# Redirect to backend
  location / {
    proxy_pass http://ha.lt0x.am/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_connect_timeout 120;
    proxy_send_timeout 120;
    proxy_read_timeout 180;
    }
}

```

Restart Nginx: 
```bash
systemctl restart nginx
```

Now try opening some `Nginx` URL and `Apache` URL. 
```bash
links reverse.lt0x.am
```

Check that Nginx listens 10.10.x.222 address: 
```bash
netstat -nlpt | grep nginx
```

Check the logs: 
```bash
tail -f /var/log/nginx/reverse-lt0x.am-access.log
```
