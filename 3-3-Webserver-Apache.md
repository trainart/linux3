# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Web Server (Apache, SSL, PHP, MySQL)


Ensure you have proper hostname set 

```bash
hostnamectl set-hostname lt01.am
```

Check:
```bash
hostnamectl ;\
hostname 
```

Install Apache and related stuff: 

For CentOS 8 first run this:  `dnf config-manager --set-enabled powertools`

```bash
yum -y install httpd mod_ssl openssl elinks lynx
```

Enable & start: 
```bash
systemctl enable --now httpd 
```

#### PRACTICE
* Now when you have running Apache webserver, set current IP address of your Linux server to `www.lt01.am` record in the BIND DNS server and restart BIND.<br><br>
* If you did everything correct you should be able to open it locally, with `links www.lt01.am`<br><br>
* Now try also `links lt01.am`. Did it open? Why? How to fix it?



### Apache configuration
Default website location directory is:  `/var/www/html` 

After installation Apache webserver is ready to use by default configuration.
But it would be good to fix some parameters as we do in examples below (to ensure more security hardening for production environment). More info at: `http://httpd.apache.org`. 
<br><br>
Apache configuration is stored in: `/etc/httpd`<br><br>  
Main config file is: `/etc/httpd/conf/httpd.conf` contains following lines:<br>
`include conf.d/*.conf`

Which means include any `*.conf` files from `/etc/httpd/conf.d` 

Now create a separate virtual host configuration file /etc/httpd/conf.d/lt01.am.conf:

```bash
cat  > /etc/httpd/conf.d/lt01.am.conf << "EOF1"
<VirtualHost *:80> 
ServerName lt01.am
ServerAlias www.lt01.am
DocumentRoot /var/www/lt01.am
CustomLog /var/log/httpd/lt01.am-access.log combined
ErrorLog /var/log/httpd/lt01.am-error.log
 <Directory /var/www/lt01.am>
      DirectoryIndex index.php index.html
      Options -Indexes
      AllowOverride ALL
 </Directory>
</VirtualHost>
EOF1

```

Create virtual host website directory:  
```bash
mkdir /var/www/lt01.am
```

Put some file there as ‘index.html’ to be displayed as main page:
```bash
cat  > /var/www/lt01.am/index.html << "EOF1"
HI this is APACHE page
EOF1

```

Restart Apache:
```bash
systemctl restart httpd   
```

Check

`links lt01.am`

`curl -s http://lt01.am/ | grep APACHE`


### Access Control with .htaccess (http://www.htaccess-guide.com/)

It is possible to restrict visitors by IP address. 
This function id enabled on per-directory basis 
in the above VirtualHost configuration following option `AllowOverride ALL`.

It allows (re)configure restrictions without restarting Apache.
We can just create a `.htaccess` file in the directory we want to restrict access.

Let us create a restricted subdirectory `closed`
```bash
mkdir -p /var/www/lt01.am/closed/img ;\
cat << EOF1 > /var/www/lt01.am/closed/test.html
<h2> Hello Linux </h2>
EOF1
cat << EOF2 > /var/www/lt01.am/closed/.htaccess
order allow,deny 
allow from 127.
#allow from 10.
#allow from 192.168. 
options +indexes
EOF2

```

Now try opening it
```bash
links www.lt01.am/closed
```

To access it you need to uncomment `#allow from 10.` line in `.htaccess`
Remove hashtag and try again.

Now you should be able to access the directory, and you will see it's file list, because we have
`options +indexes` option.

Try commenting it an open it again
```bash
links www.lt01.am/closed
```

You will not see the directory contents. 
But you can access files by the name:

```bash
links www.lt01.am/closed/test.html
```

Now we can create an index file 
```bash
cat << EOF1 > /var/www/lt01.am/closed/index.html
<h2> THIS IS INDEX OF CLOSED DIR </h2>
EOF1

```

Try to open it again
```bash
links www.lt01.am/closed
```

#### Error handling
We can add handling of HTTP protocols errors, like 404 - Not found
```bash
cat << EOF3 >> /var/www/lt01.am/closed/.htaccess
ErrorDocument 404 http://ping.eu
EOF3

```
(This way you can redirect visitor to some special page if they try to open the page, that doesn't exist)

Now if we try non existing URL, we will be redirected to `ping.eu`
```bash
links www.lt01.am/closed/123
```

#### Password protection

Let's create another subdirectory `secure` and make it password-protected
```bash
mkdir -p /var/www/lt01.am/secure/docs ;\
cat << EOF3 > /var/www/lt01.am/secure/index.html
<h1> Linux Training </h1>
EOF3
cat << EOF4 > /var/www/lt01.am/secure/.htaccess
AuthName "SECURE Area"
AuthUserFile /etc/httpd/conf.d/.htpasswd
AuthType Basic
require valid-user
EOF4

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

Now try access this URL. You will require user/password
```bash
links www.lt01.am/secure/
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
echo '<?php phpinfo(); ?>' > /var/www/lt01.am/inf.php
```

Check: 
```bash
links http://www.lt01.am/inf.php
```
> (PHP configuration can be done in config files: `/etc/php.ini`, `/etc/php.d/`)

Now we can create an PHP index file 
```bash
cat << EOF1 > /var/www/lt01.am/index.php
<?php echo "<h3>HI THIS PHP INDEX</h3>"; ?>
EOF1

```

Check: 
```bash
links http://www.lt01.am/
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

> NOTE: we do this for testing, but in production it should be done via `mysql_secure_installation`

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

Create mysqltest.php

```bash
cat > /var/www/lt01.am/mysqltest.php << "EOF3"
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
EOF3

```

Check: 
```bash
curl http://lt01.am/mysqltest.php
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
curl http://lt01.am/mysqltest.php
```


### Configure ‘mod_ssl’ for Apache

Self-signed Certificate generation:

```bash
openssl req -x509 -batch -nodes -days 3650 -newkey rsa:4096 -keyout lt01.am.key -out lt01.am.crt -subj "/C=AM/ST=Yerevan/L=Yerevan/O=AITC/OU=Linux Training/CN=lt01.am"

```

Put certificates at their place:
```bash
mv lt01.am.crt /etc/pki/tls/certs ;\
mv lt01.am.key /etc/pki/tls/private
```

> Nowadays there are several ways to get free production SSL certificate from CAa like `LetsEncrypt` or `ZeroSSL`.
> They only give certificates for 3 month, so there should be some automated procedure to update it regularly.
> This is done by scripts like `certbot` 
> Unfortunately to try all the above, you should have real domain configured for your dns name and have real IP access from outside world.
> That's why here we only use self-signed certificates (in fact note, they are of the same strength, just not known to Client/Browsers).

### Create SSL Virtual Host

```bash
cat <<EOF1  > /etc/httpd/conf.d/lt01.am-ssl.conf 
<VirtualHost *:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/lt01.am.crt
        SSLCertificateKeyFile /etc/pki/tls/private/lt01.am.key
        ServerName lt01.am
        ServerAlias www.lt01.am
        DocumentRoot /var/www/lt01.am
        ErrorLog logs/lt01.am-ssl_error_log
        TransferLog logs/lt01.am-ssl_access_log
        <Directory /var/www/lt01.am >
        DirectoryIndex index.php index.html
        Options -Indexes
        AllowOverride All
        </Directory>
</VirtualHost>
EOF1

```

Restart Apache: 
```bash
systemctl restart httpd 
```

Redirect 80 port to 443 (SSL)

Add following line to  `/etc/httpd/conf.d/lt01.am.conf`
> Redirect permanent  /  https://www.lt01.am/

Following command will REPLACE previous config file.

```bash
cat  > /etc/httpd/conf.d/lt01.am.conf << "EOF1"
<VirtualHost *:80> 
ServerName lt01.am
ServerAlias www.lt01.am
Redirect permanent  /  https://www.lt01.am/
DocumentRoot /var/www/lt01.am
CustomLog /var/log/httpd/lt01.am-access.log combined
ErrorLog /var/log/httpd/lt01.am-error.log
 <Directory /var/www/lt01.am>
      DirectoryIndex index.php index.html
      Options -Indexes
      AllowOverride ALL
 </Directory>
</VirtualHost>
EOF1

```

Restart Apache: 
```bash
systemctl restart httpd 
```

Now if you access the site at 80 port you will be automatically redirected to 443

Check: 
```bash
curl http://lt01.am/mysqltest.php
```

You will see the "301 Moved Permanently" message.
Now to make curl follow it an get the ssl you need to run: 

> NOTE: '--insecure' option is needed because we use self-signed certificate

```bash
curl --location --insecure http://lt01.am/mysqltest.php
```


### Hardening Apache

* In order to hide Apache version, the following options are to be set in configuration:

`ServerTokens Prod`
`ServerSignature Off`


* Change
```bash
AddType application/x-httpd-php .php 
```
to

```bash
AddType application/x-httpd-php .php .php~
AddHandler php5-script .php .php~
AddType text/html .php .php~
```


* Remove `Indexes` from Options 

* Hide PHP version (X-Powered-By) in `php.ini`
(it could be located in different places /etc/php.ini, /etc/php5/apache2/php.ini, ) 
> expose_php = Off

* Reduce Timeout:		
> Timeout 45

* Limit big requests:	
> LimitRequestBody 1048576
