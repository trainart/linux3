# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Email service, Mail server (Postfix, Dovecot)
_(partially based on https://www.server-world.info/en/note?os=CentOS_8&p=mail&f=1)_

### How email works (MTA, MDA, MUA)

Email is based around the use of electronic mailboxes. 
When an email is sent, the message is routed from server to server, 
all the way to the recipient's email server. 
More precisely, the message is sent to the mail server tasked 
with transporting emails (called the **MTA**, for Mail Transport Agent) 
to the recipient's **MTA**. On the Internet, **MTA**s communicate with one 
another using the protocol **SMTP** (Simple Mail Transfer Protocol), 
and so are logically called SMTP servers (or sometimes "outgoing mail servers"). 

The recipient's **MTA** then delivers the email to the incoming mail server 
(called the **MDA**, for Mail Delivery Agent), 
which stores the email as it waits for the user to retrieve it. 
**MAA** (Mail Access Agent) is used to retrieve mails from  mailboxes.

There are two main protocols used for retrieving email from MAA: 
* **POP3** (Post Office Protocol), the older of the two, which is used for retrieving email and, in certain cases, leaving a copy of it on the server. 


* **IMAP** (Internet Message Access Protocol), which is used for coordinating the status of emails (read, deleted, moved) across multiple email clients. With IMAP, a copy of every message is saved on the server, so that this synchronization task can be completed. 

For this reason, _incoming mail servers_ (**MAA**s) 
are called **POP** servers or **IMAP** servers, 
depending on which protocol is used. 


![img_3.png](img_3.png)


To use a real-world analogy, **MTA**s act as the post office 
(the sorting area and mail carrier, which handle message transportation), 
while **MDA**s act as mailboxes, which store messages 
(as much as their volume will allow) until the recipients check the box. 
This means that it is not necessary for recipients to be connected in 
order for them to be sent email. 

That is why Email service is not considered an **online** service, but instead is "**store-and-forward**" service 

To keep everyone from checking other users' emails, **MAA** is protected by login/password. 
Retrieving mail is done using a software program called an **MUA** (Mail User Agent). 
When the **MUA** is a program installed on the 
user's system, it is called an email client 
(such as Mozilla Thunderbird, Microsoft Outlook). 

When it is a web interface used for interacting with the 
incoming mail server, it is called **Webmail**. 

### Ensure you have prepare DNS configuration

Before going to mail server, let's ensure you have prepared the DNS system for that.

#### Separate domains for each student

* Each student should have configured separate domain **master** zone (lt01.am,lt02.am,lt03.am,...)
* Teacher will configure **slave** zones for each such domain (lt01.am,lt02.am,lt03.am,...)
* Make sure your Linux system has teacher's IP in /etc/resolv.conf as first `nameserver`. As a result all will know about all domains.


#### Define hostname

Set the hostname of each student to match appropriate separate domain (lt01.am,lt02.am,lt03.am,...).

```bash
hostnamectl set-hostname lt0x.am ; hostname 
```


### Install & configure Postfix as SMTP Server

Install Postfix.

```bash
yum -y install postfix
```

Configure Postfix

Open `/etc/postfix/main.cf` and go to line 95 (_first line we want to edit now_)
```bash
nano +95 /etc/postfix/main.cf
```
or
```bash
vi +95 /etc/postfix/main.cf
```

> In `nano` goto the some line number with `Ctrl-Shift-_`
> 
> In `vi`   goto the some line number  with `Esc`, 
> then type the line number, and then press `Shift-g`
> 
> Just `Esc`+`Shift-g` will take you to the end of file

```bash	
# goto line 95: uncomment and specify hostname
myhostname = lt0x.am
# goto line 102: uncomment and specify domain name
mydomain = lt0x.am
# goto line 118: uncomment
myorigin = $mydomain
# goto line 135: change
inet_interfaces = all
# goto line 183: add
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
# goto line 283: uncomment and specify your local network
mynetworks = 127.0.0.0/8, 10.0.0.0/8
# goto line 438: uncomment (use Maildir)
home_mailbox = Maildir/
# goto line 593: add
smtpd_banner = $myhostname ESMTP

# add following to the end of file

# SMTP-Auth setting
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
smtpd_recipient_restrictions = permit_mynetworks, permit_auth_destination, permit_sasl_authenticated, reject

```


Enable and start Postfix:
```bash
systemctl enable --now postfix
```

Disable Firewalld
```bash
systemctl disable --now firewalld
```
> In case of active Firewalld we will need to allow SMTP ports for Postfix and POP/IMAP ports for Dovecot. <br>
> ```bash
> firewall-cmd --add-service={smtp,smtps,pop3,pop3s,imap,imaps} --permanent ;\
> firewall-cmd --reload
> ```
> But we will not do that for this training.
>

### Install Dovecot to configure POP/IMAP Server.

Install Dovecot 
```bash
yum -y install dovecot
```

Configure Dovecot to provide SASL (Simple Authentication and Security Layer) capability to Postfix

```bash
nano +30 /etc/dovecot/dovecot.conf
```

```bash
# add following after line 30
listen = *
```

```bash
nano +10 /etc/dovecot/conf.d/10-auth.conf
```

```bash
# add following after line 10
disable_plaintext_auth = no
# go to line 100: and change as follows
auth_mechanisms = plain login
```

```bash
nano +30 /etc/dovecot/conf.d/10-mail.conf
```
```bash
# add following after line 30
mail_location = maildir:~/Maildir
```

```bash
nano +107 /etc/dovecot/conf.d/10-master.conf
```

```bash
# uncomment line 107-109 and add some lines as follows
# Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
```

```bash
nano +8 /etc/dovecot/conf.d/10-ssl.conf
```

```bash
# change line 8 as follows (means SSL is not required)
ssl = yes
```

Enable and start Dovecot:
```bash
systemctl enable --now dovecot
```

We have configured basic Postfix & Dovecot settings.
We can now create some new Linux OS user account to test email

> NOTE: Modern production solutions of mailserver are more complex 
> and generally use some database (like MySQL) to store email user data.
> But for this training we will use basic Linux user variant

Install simple terminal mail client program
```bash
yum -y install mailx
```
Set environment variables to use Maildir:
```bash
echo 'export MAIL=$HOME/Maildir' >> /etc/profile.d/mail.sh
```

Add a user `tester`
```bash
useradd tester ;\
passwd tester
```

Now try the above mail sending with `telnet`

Install telnet if needed
```bash
yum -y install telnet
```

#### Add **Rsyslog** configuration to log Postfix and Dovecot logs separately.


```bash
cat > /etc/rsyslog.d/postfix.conf << "ENDTEXT"
if $programname == "postfix" then /var/log/postfix.log
ENDTEXT

```

```bash
cat > /etc/rsyslog.d/dovecot.conf << "ENDTEXT"
if $programname == "dovecot" then /var/log/dovecot.log
ENDTEXT

```

Restart rsyslog:
```bash
systemctl restart rsyslog
```

Restart postfix:
```bash
systemctl restart postfix
```

Restart dovecot:
```bash
systemctl restart dovecot
```

Check

```bash
tail /var/log/postfix.log
```

Check

```bash
tail /var/log/dovecot.log
```

#### SMTP Session example
Try following example

```bash
telnet lt0x.am 25
```

> Trying 192.168.1.1...
> Connected to lt0x.am.
> Escape character is '^]'.
> 220 lt0x.am SMTP on Fri, 3 Aug 2001 10:38:06 +0400

```bash
helo lo
```

> 250 Hello root@lt0x.am, pleased to meet you

```bash
mail from: user@yahoo.com
```
> 250 user@yahoo.com... Sender ok

```bash
rcpt to: tester@lt0x.am     
```

> 250 tester@lt0x.am... Recipient ok

```bash
data
```

> 354 Enter mail, end with "." on a line by itself

```bash
From: "TEST" <test@mail.com>
To: "TEST" <test@mail.com>
Subject: Test message
Date: Mon, 02 Feb 1991 13:00:57 +0400

Hello, This is a test message.

Yours truly,
Administrator
.
```

> 250 KAA24894 Message accepted for delivery

```bash

quit
```


Try sending mail via terminal `mail` command 
```bash
mail tester@lt0x.am
```

Switch to `tester` user and check mail
(type `q` to exit  `mail` program)
```bash
su - tester
mail
```

Check logs

```bash
tail /var/log/postfix.log
```

Send mail to Teacher's server

```bash
mail tester@lt00.am
```

Check logs

```bash
tail /var/log/postfix.log
```

Check mail queue

```bash
mailq
```

## Mail client configuration

Make changes in DNS configuration to have additional records for incoming and outgoing servers.<br>
This helps for **Mail client autoconfiguration**.

* Add `smtp.lt0x.am`  record

  * name:   `smtp` 
  * type:   `A` 
  * value:  `10.10.x.1`

* Add `imap.lt0x.am`  record

  * name:   `imap` 
  * type:   `A` 
  * value:  `10.10.x.1`

<br><br>

Now install and configure `thunderbird` graphical Mail Client. 

```bash
yum -y install thunderbird
```


>
>  Portable Windows version is at:
> `https://portableapps.com/apps/internet/thunderbird_portable`
>


> IMPORTANT! When configuring MailClient specify username as `tester` without domainname, 
> since we use Linux users as test users.
> In production nowadays mailbox users are generally not related to Linux users, 
> but are created separately in some database, like MySQL.



## Webmail infterface

Here we try to install and use `Snappymail` (https://snappymail.eu/)


Upgrade PHP to version 7.4

```bash
dnf -y module reset php:7.2 && dnf -y module enable php:7.4
```

```bash
yum -y install php php-common php-gd php-xml php-mbstring php-mysqlnd php-gd
```

Restart Apache
```bash
systemctl restart httpd
```
`

Prepare directory and get the package

```bash
mkdir -p /var/www/lt0x.am/webmail
```
```bash
cd /var/www/lt0x.am/webmail
```

Download and extract Snappymail

```bash
wget --inet4-only https://snappymail.eu/repository/latest.tar.gz
```
```bash
tar -xzf latest.tar.gz
```

```bash
rm -f latest.tar.gz
```

Set proper permissions

```bash
find /var/www/lt0x.am/webmail -type d -exec chmod 755 {} \;
```

```bash
find /var/www/lt0x.am/webmail -type f -exec chmod 644 {} \;
```

```bash
chown -R apache:apache /var/www/lt0x.am/webmail
```

Now try accessing  `http://apache.lt0x.am/webmail`

You should be able to login with user `tester@lt0x.am`

You should see incoming messages, <br>
**but will not be able to send yet**.

We need to enable authorization for sending.

```bash
sed -i 's/"useAuth": false/"useAuth": true/' /var/www/lt0x.am/webmail/data/_data_/_default_/domains/lt0x.am.json
```


(Also we might need to manually create folders Sent, ...)

> NOTE! Production installation requires Admin access configuration according to:
> https://github.com/the-djmaze/snappymail/wiki/Installation-instructions#now-access-the-admin-page
> 

## Mail routing

Mail routing may be organised in two ways:

* Public MX-based routing
  * to specify destination of mails coming from the world to **our domain**. 
* Internal static forwarding to another mailserver
  * to forward mails from one server to another 
  * may be used for both **our domain** and **other domains**


### MX configuration to route mails via central Hub (Teacher's Server)

Make changes in DNS configuration 

* **Modify** `MX` resource record for your domain to point to `mx2.lt0x.am`
 
  * type:   `MX` 
  * value:  `0 mx2.lt0x.am.`

* Add `A` record for it.

  * name:   `mx2` 
  * type:   `A` 
  * value:  `10.10.x.111`
  

Update `PTR` record `111.x.10.10.in-addr.arpa.`

* `mx2.lt0x.am` PTR record:

  * type:		`PTR`
  * name:       `111`
  * value:	    `mx2.lt0x.am.`

Now your domain mails will go to Teacher's server.  
But it doesn't mean they will be accepted there.
To have them accepted we need to add some configuration there too.

In order mail for some domain to be accepted by mailserver
it should either be registered as: 
1. **local domain**, or 
2. **domain to forward mails somewhere**.

Below we configure the second for student's domains.

### Configuration of central Hub (Teacher's Server)

Add following lines to `/etc/postfix/transport`

With contents:
```bash
cat >> /etc/postfix/transport << "ENDTEXT"
lt01.am smtp:[mail.lt01.am]:25
lt02.am smtp:[mail.lt02.am]:25
lt03.am smtp:[mail.lt03.am]:25
lt04.am smtp:[mail.lt04.am]:25
lt05.am smtp:[mail.lt05.am]:25
lt06.am smtp:[mail.lt06.am]:25
lt07.am smtp:[mail.lt07.am]:25
ENDTEXT
```


Build that config
```bash
postmap /etc/postfix/transport
```


Add that to Postfix main config file `/etc/postfix/main.cf`

```bash
cat >> /etc/postfix/main.cf << "ENDTEXT"
transport_maps = hash:/etc/postfix/transport
ENDTEXT

```

Restart Postfix

```bash
systemctl restart postfix
```

Now try to send mail from one to another
and check logs where do they go.


### Second MX as backup

Make changes in DNS configuration 

* **Add** second `MX` record for your domain to point to `bkpmx.lt0x.am`
 
  * type:   `MX` 
  * value:  `10 bkpmx.lt0x.am.`

* Add `A` record for it.

  * name:   `bkpmx` 
  * type:   `A` 
  * value:  `10.10.x.1`
  

> NOTE ! We have set lower priority `10`
> So mail will go here if first server will not respond.

Now Teacher will shutdown his postfix.
and you can try sending mails to another student's domains.

After sending check logs to see how mail was routed.


```bash
tail /var/log/postfix.log
```


### Configuration of Students servers

Here we will configure your domain mail to route to Teacher's server without MX.

Add following lines to `/etc/postfix/transport`

With contents:
```bash
cat >> /etc/postfix/transport << "ENDTEXT"
lt01.am smtp:[mail.lt00.am]:25
lt02.am smtp:[mail.lt00.am]:25
lt03.am smtp:[mail.lt00.am]:25
lt04.am smtp:[mail.lt00.am]:25
lt05.am smtp:[mail.lt00.am]:25
lt06.am smtp:[mail.lt00.am]:25
lt07.am smtp:[mail.lt00.am]:25
ENDTEXT
```

Build that config
```bash
postmap /etc/postfix/transport
```


Add that to Postfix main config file `/etc/postfix/main.cf`

```bash
cat >> /etc/postfix/main.cf << "ENDTEXT"
transport_maps = hash:/etc/postfix/transport
ENDTEXT

```

Restart Postfix

```bash
systemctl restart postfix
```
    

Now try sending mail to another domain and check the logs:

```bash
tail -f /var/log/postfix.log
```







### Mail Log Report : pflogsumm

Install `pflogsumm` which is the Postfix Log reporting tool.

```bash
yum -y install postfix-perl-scripts
```

Generate mail log summary for today
```bash
pflogsumm -e /var/log/postfix.log | less
```

