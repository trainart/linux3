# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Proxy server (Squid)


Squid (http://www.squid-cache.org/) is a caching proxy for the Web supporting HTTP, HTTPS, FTP, and more. It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages. Squid has extensive access controls and makes a great server accelerator.

Install Squid:

```bash
yum -y install squid 
```

Config directory is:   `/etc/squid/` 
Main configuration file is: `/etc/squid/squid.conf`

Enable and start:
```bash
systemctl enable --now squid 
```

Check existence:
```bash
pstree | grep squid
```

Check which port is squid listening:
```bash
netstat -nlpt | grep squid
```
After installation with default configuration Squid is ready 
to serve the clients if they connect from the internal LAN IP addresses (defined in RFC1918 https://tools.ietf.org/html/rfc1918): 
* 10.0.0.0/8 
* 172.16.0.0/12 
* 192.168.0.0/16 

To use proxy server in terminal set `http_proxy` variable:
```bash
export http_proxy=http://127.0.0.1:3128
```


Check.
In one terminal open this:  
```bash
tail -f /var/log/squid/access.log
```
In another try opening this URL:
```bash
lynx http://all-nettools.com/toolbox/proxy-test.php
```

Another way to test with `curl`

> `curl` is available by default on all modern Windows, Mac, and Linux environments, so you can open any local shell to run this command:
> for remote connetion just change `127.0.0.1` with the external IP address of the server
```bash
curl -v -x http://127.0.0.1:3128 https://ya.ru/
```

> Also try it with some other Browser on your Windows system


All below modifications are to be made in `/etc/squid/squid.conf`

Squid configuration documentation is located in: `/usr/share/doc/squid-*.*.*/squid.conf.documented`

After any changes you should restart squid server `systemctl restart squid`

Log files are stored in: `/var/log/squid`

* `/var/log/squid/access.log`
* `/var/log/squid/cache.log`
 

Configuration examples

#### Set Anonymous Headers to hide proxy use

Check
```bash
lynx http://all-nettools.com/toolbox/proxy-test.php
```

Try the same with `curl`
```bash
curl -v -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep detected
```

Now hide HTTP headers that reveal you are behind the proxy

```bash
cat <<EOF4  >> /etc/squid/squid.conf
request_header_access Via deny all
request_header_access X-Forwarded-For deny all
request_header_access From deny all
request_header_access Link deny all
request_header_access Server deny all
EOF4
```
Reload Squid
```bash
systemctl reload squid
```

Check
```bash
lynx http://all-nettools.com/toolbox/proxy-test.php
```

Try the same with `curl`
```bash
curl -v -x http://127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected"
```


#### Change the server public hostname and Squid version, appearing in case of errors

Access wrong URL
```bash
lynx http://1
```

Now change 
```bash
cat <<EOF4  >> /etc/squid/squid.conf
visible_hostname MYPROXY
httpd_suppress_version_string on
EOF4
```
Reload Squid
```bash
systemctl reload squid
```

Access wrong URL again and notice change in message
```bash
lynx http://1
```


#### User/password based access

In `/etc/squid/squid.conf` add this at appropriate places

Under
> `# Recommended minimum configuration:`

add:
```bash
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/pss
auth_param basic children 5
auth_param basic realm AUTHENTICATE
auth_param basic credentialsttl 2 hours
acl pss proxy_auth REQUIRED
```
Under

> `# Recommended minimum Access Permission configuration:`

add:
```bash
#http_access allow  localnet
#http_access allow  localhost
http_access deny  !pss
http_access allow  pss
```

Now create Users/Passwords with ‘htpasswd’ command:
> NOTE: to change password use the same command without `-c`
```bash
htpasswd -c /etc/squid/pss demo
```

Reload Squid
```bash
systemctl reload squid
```

Try accessing some URL:
```bash
lynx http://all-nettools.com/toolbox/proxy-test.php
```

Try the same with `curl`
```bash
curl -v -x http://demo:123456@127.0.0.1:3128 http://all-nettools.com/toolbox/proxy-test.php | grep "not detected"
```


### SquidAnalyzer - Squid access log report generation tool (https://github.com/darold/squidanalyzer)

SquidAnalyzer parse native access log format of the Squid proxy and 
generate general statistics about hits, bytes, users, networks, top url,
top second level domain and denied URLs.

Ensure you have `git` package
```bash
yum install git
```

Install SquidAnalyzer & run it manually once

```bash
cd ;\
git clone https://github.com/darold/squidanalyzer.git ;\
cd squidanalyzer ; \
perl Makefile.PL ; \
make && make install ; \
ln -s /var/log/squid/ /var/log/squid3 ;\
/usr/local/bin/squid-analyzer ;\
ln -s /var/www/squidanalyzer /var/www/lt01.am/squidanalyzer
```

> _For production use following cronjob should be configured to run squid-analyzer daily:_
>
> ```bash
> # SquidAnalyzer log reporting daily
> 0 2 * * * /usr/local/bin/squid-analyzer > /dev/null 2>&1
> ```
> 

Try accessing the report

```bash
lynx http://127.0.0.1/squidanalyzer/
```


#### PRACTICE

* Create new proxy user `student` with some password
* Access some URL with that user, while having `tail -f /var/log/squid/access.log` on another terminal. You should see in the logs that you access with user `student`
* Run SquidAnalyzer manually again 
* Check the new report and find statistics for user `student`
