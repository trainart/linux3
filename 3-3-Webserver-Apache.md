# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Web Server (Apache, SSL, PHP, MySQL)


Ensure you have proper hostname set 

> IMPORTANT REMINDER ! 
> lt0x.am  (x – student’s number assigned by teacher)
> Teacher’s domain is: lt00.am,  
> Students domains are: lt01.am, lt02.am, lt03.am, …
> So, here and below each student should 
> everywhere change the number `x` 
> to his/her number assigned by teacher.

```bash
hostnamectl set-hostname lt0x.am
```

Check:
```bash
hostnamectl ;\
hostname 
```

Install Apache and related stuff: 

> IMPORTANT !<br>
> For **CentOS 8** first run this: <br>
> `dnf config-manager --set-enabled powertools`

```bash
yum -y install httpd mod_ssl openssl elinks lynx curl
```

Enable & start: 
```bash
systemctl enable --now httpd 
```

#### PRACTICE
* Now when you have running Apache webserver, set current IP address of your Linux server to `www.lt0x.am` record in the BIND DNS server and restart BIND.<br><br>
* If you did everything correct you should be able to open it locally, with `links www.lt0x.am`<br><br>
* Now try also `links lt0x.am`. Did it open? Why? How to fix it?



### Apache configuration
Default website location directory is:  `/var/www/html` 

After installation Apache webserver is ready to use by default configuration.
But it would be good to fix some parameters as we do in examples below (to ensure more security hardening for production environment). More info at: `http://httpd.apache.org`. 
<br><br>
Apache configuration is stored in: `/etc/httpd`<br><br>  
Main config file is: `/etc/httpd/conf/httpd.conf` contains following lines:<br>
`include conf.d/*.conf`

Which means include any `*.conf` files from `/etc/httpd/conf.d` 

Now create a separate virtual host configuration file `/etc/httpd/conf.d/lt0x.am.conf`:

`/etc/httpd/conf.d/lt0x.am.conf`

```bash
<VirtualHost *:80> 
ServerName lt0x.am
ServerAlias www.lt0x.am
DocumentRoot /var/www/lt0x.am
CustomLog /var/log/httpd/lt0x.am-access.log combined
ErrorLog /var/log/httpd/lt0x.am-error.log
 <Directory /var/www/lt0x.am>
      DirectoryIndex index.php index.html
      Options -Indexes
      AllowOverride ALL
 </Directory>
</VirtualHost>
```


> REMEMBER to change `x` in every `lt0x.am` with your number.<br>
> You can do that with commands like<br>
> `sed -i 's/lt0x/lt01/g' /etc/httpd/conf.d/lt0x.am.conf`<br>
> or<br>
> `perl -pi -e "s/lt0x/lt01/" /etc/httpd/conf.d/lt0x.am.conf `<br>
> But change `lt01` to your number before running it




Create virtual host website directory:  
```bash
mkdir /var/www/lt0x.am
```

Put some text there as ‘index.html’ to be displayed as main page.
Create file `/var/www/lt0x.am/index.html`
with text:
```bash
HI this is APACHE page
```

Restart Apache:
```bash
systemctl restart httpd   
```

Check

`links lt0x.am`

`curl -s http://lt0x.am/ | grep APACHE`


### Access Control with .htaccess (http://www.htaccess-guide.com/)

It is possible to restrict visitors by IP address. 
This function id enabled on per-directory basis 
in the above VirtualHost configuration following option `AllowOverride ALL`.

It allows (re)configure restrictions without restarting Apache.
We can just create a `.htaccess` file in the directory we want to restrict access.

Let us create a restricted subdirectory `closed`

```bash
mkdir -p /var/www/lt0x.am/closed/img
```

Create file  `/var/www/lt0x.am/closed/test.html`
with text:
```bash
<h2> Hello Linux </h2>
```

Create file `/var/www/lt0x.am/closed/.htaccess`
with text:
```bash
order allow,deny 
allow from 127.
#allow from 10.
#allow from 192.168. 
options +indexes
```

Now try opening it
```bash
links www.lt0x.am/closed
```

To access it you need to uncomment `#allow from 10.` line in `.htaccess`
Remove hashtag and try again (NOTE! nothing needs to be restarted).

Now you should be able to access the directory, and you will see it's file list, because we have
`options +indexes` option.

Add hashtag before line `options +indexes` to comment it
and try to open again
```bash
links www.lt0x.am/closed
```

You will not see the directory contents. 
But you can access files by the name:

```bash
links www.lt0x.am/closed/test.html
```

Now we can create an index file 
Create file `/var/www/lt0x.am/closed/index.html`
with text:
```bash
<h2> THIS IS INDEX OF CLOSED DIR </h2>
```

Try to open it again
```bash
links www.lt0x.am/closed
```

#### QUESTION
Why didn't we see `test.html` file automatically, but 
`index.html` was shown without specifying filename ?



#### Error handling
We can add handling of HTTP protocols errors, like `404` - `Not found`

ADD following line to file  `/var/www/lt0x.am/closed/.htaccess`

```bash
ErrorDocument 404 http://ping.eu
```

(This way you can redirect visitor to some special page if they try to open the page, that doesn't exist)

Now if we try non existing URL, we will be redirected to `ping.eu`
```bash
links www.lt0x.am/closed/123
```

#### Password protection

Let's create another subdirectory `secure` and make it password-protected

Create directory `/var/www/lt0x.am/secure/docs`
(you can use `mkdir -p`)

create file `/var/www/lt0x.am/secure/index.html`
with text:
```bash
<h1> Linux Training </h1>
```
create file `/var/www/lt0x.am/secure/.htaccess`

with text:
```bash
AuthName "SECURE Area"
AuthUserFile /etc/httpd/conf.d/.htpasswd
AuthType Basic
require valid-user
```


Create user ‘test123’ and set the password 
```bash
htpasswd -c /etc/httpd/conf.d/.htpasswd  test123
```

> NOTE: `-c` is needed only once to create the new file
> To change password you will need to run:
> ```bash
> htpasswd /etc/httpd/conf.d/.htpasswd test123
> ```

Now try access this URL. You will need to enter user/password
```bash
links www.lt0x.am/secure/
```


### Installing PHP with database support

Install required packages:
```bash
yum -y install php php-common php-gd php-xml php-mbstring php-mysqlnd php-gd
```

Restart Apache:
```bash
systemctl restart httpd
```
Create test php script:   
```bash
echo '<?php 
$name = "Linux Student";
echo "Hi $name. PHP works here !";

?>' > /var/www/lt0x.am/phpcheck.php

```

Check: 
```bash
links http://www.lt0x.am/phpcheck.php
```

> (PHP configuration can be done in config files: `/etc/php.ini`, `/etc/php.d/`)

Now we can create an PHP index file 
Create file `/var/www/lt0x.am/index.php`
with text:
```bash
<?php echo "<h3>HI THIS PHP INDEX</h3>"; ?>
```

Check: 
```bash
links http://www.lt0x.am/
```


### Install MariaDB (MySQL)
```bash
yum -y install mariadb mariadb-server
```

Enable and Start MySQL:
```bash
systemctl enable --now mariadb  
```

Set  root password for MySQL:
```bash
mysqladmin -u root password '123456'
```

> NOTE: we do this for testing, but in production special command `mysql_secure_installation` should be executed.

Simple check of how MySQL works with website.  

Create new database and insert some data there

```bash
mysql -u root --password=123456 << EOF3
CREATE DATABASE example_database;
USE example_database;
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);
INSERT INTO example_database.todo_list (content) VALUES ("First important item");
INSERT INTO example_database.todo_list (content) VALUES ("Second important item");
INSERT INTO example_database.todo_list (content) VALUES ("Third important item");
INSERT INTO example_database.todo_list (content) VALUES ("Another important thing");
EOF3

```

Check the data from that database

```bash
mysql -u root --password=123456 << EOF3
USE example_database;
SELECT * FROM example_database.todo_list;
EOF3

```

Create `/var/www/lt0x.am/mysqltest.php`
with text
```bash
<?php
$user = "root";
$password = "123456";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h4>MY TODO LIST</h4><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

Check: 
```bash
links http://lt0x.am/mysqltest.php
```

Add data to database:
```bash
mysql -u root --password=123456 << EOF3
USE example_database;
INSERT INTO example_database.todo_list (content) VALUES ("NEW ITEM");
EOF3

```

Check again: 
```bash
curl http://lt0x.am/mysqltest.php
```


### Configure ‘mod_ssl’ for Apache

> REMEMER TO CHANGE `lt0x.am` below

Self-signed Certificate generation:

```bash
openssl req -x509 -batch -nodes -days 3650 -newkey rsa:4096 -keyout lt0x.am.key -out lt0x.am.crt -subj "/C=AM/ST=Yerevan/L=Yerevan/O=AITC/OU=Linux Training/CN=lt0x.am"

```

Put certificates at their place:
```bash
mv lt0x.am.crt /etc/pki/tls/certs ;\
mv lt0x.am.key /etc/pki/tls/private
```

> Nowadays there are several ways to get free production SSL certificate from CAa like `LetsEncrypt` or `ZeroSSL`.
> They only give certificates for 3 month, so there should be some automated procedure to update it regularly.
> This is done by scripts like `certbot` 
> Unfortunately to try all the above, you should have real domain configured for your dns name and have real IP access from outside world.
> That's why here we only use self-signed certificates (in fact note, they are of the same strength, just not known to Client/Browsers).

### Create SSL Virtual Host

Create `/etc/httpd/conf.d/lt0x.am-ssl.conf`
with text:

```bash
<VirtualHost *:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/lt0x.am.crt
        SSLCertificateKeyFile /etc/pki/tls/private/lt0x.am.key
        ServerName lt0x.am
        ServerAlias www.lt0x.am
        DocumentRoot /var/www/lt0x.am
        ErrorLog logs/lt0x.am-ssl_error_log
        TransferLog logs/lt0x.am-ssl_access_log
        <Directory /var/www/lt0x.am >
        DirectoryIndex index.php index.html
        Options -Indexes
        AllowOverride All
        </Directory>
</VirtualHost>
```

> REMEMBER to change `x` in every `lt0x.am` with your number.<br>
> You can do that with commands like<br>
> `sed -i 's/lt0x/lt01/g' /etc/httpd/conf.d/lt0x.am-ssl.conf`<br>
> or<br>
> `perl -pi -e "s/lt0x/lt01/" /etc/httpd/conf.d/lt0x.am-ssl.conf `<br>
> But change `lt01` to your number before running it


Restart Apache: 
```bash
systemctl restart httpd 
```

Check byt directly accessing with HTTPS: 

```bash
linx https://lt0x.am/
```

> NOTE! We use here `lynx` browser, since it is more loyal to self-signed certificates

Redirect 80 port to 443 (SSL)

Add following line to  `/etc/httpd/conf.d/lt0x.am.conf`
after `ServerAlias` line:

```bash
Redirect permanent  /  https://www.lt0x.am/
```

It should look like:
```bash
<VirtualHost *:80> 
ServerName lt0x.am
ServerAlias www.lt0x.am
Redirect permanent  /  https://www.lt0x.am/
DocumentRoot /var/www/lt0x.am
CustomLog /var/log/httpd/lt0x.am-access.log combined
ErrorLog /var/log/httpd/lt0x.am-error.log
 <Directory /var/www/lt0x.am>
      DirectoryIndex index.php index.html
      Options -Indexes
      AllowOverride ALL
 </Directory>
</VirtualHost>
```

Restart Apache: 
```bash
systemctl restart httpd 
```

Now if you access the site at `80` port you will be automatically redirected to `443`

Check: 
```bash
linx http://lt0x.am/
```

You will see the "301 Moved Permanently" message.
Now to make curl follow it and get the ssl you need to run: 

> NOTE: '--insecure' option is needed because we use self-signed certificate

```bash
curl --location --insecure http://lt0x.am/mysqltest.php
```


### Hardening Apache

* It is always good practice to hide Apache version.
The following options can be set in configuration for that.


First check current state:

```bash
curl -sI http://lt0x.am | grep Server
```

Now add options and restart Apache

```bash
cat  > /etc/httpd/conf.d/harden.conf  << "EOF1"
ServerTokens Prod
ServerSignature Off
Timeout 45
LimitRequestBody 1048576
EOF1

systemctl restart httpd   

```


And check again:
```bash
curl -sI http://lt0x.am | grep Server
```

You should not see any details now.
