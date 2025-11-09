# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)

## Samba - network resources sharing

Samba is a free & open-source implementation of **network resources sharing** service for integrating servers and desktops running Linux or Unix in environments with Microsoft’s Active Directory directory service. 
Samba can act either as a primary domain controller or as a member. I can just be as single server to share network resources: files, printers, etc.

Samba uses the frequently used client/server protocols SMB (Server Message Block) or nowadays CIFS (Common Internet File System). 
The latter is an open variant of SMB. If applications are compatible with SMB or CIFS, they can communicate with the Samba Server.

### Install Samba 

```bash
dnf install -y samba samba-client cifs-utils
```

Add firewall rule to runtime config, add it to permanent config too and list

```bash
firewall-cmd --add-service samba ; firewall-cmd --runtime-to-permanent ; firewall-cmd --list-services
```

### Start Samba services

```bash
systemctl start {smb,nmb} ; systemctl is-active {smb,nmb}
```

### Create simple configuration

```bash
mv /etc/samba/smb.conf{,.backup} ;\
cat << "EOF1" > /etc/samba/smb.conf
[global]
workgroup = LINUXTRAINING
server string = "SAMBA Linux Server"
log file = /var/log/samba/log.%m
max log size = 50
wins support = Yes

security = user
passdb backend = tdbsam

[homes]
comment = Home Directories
browseable = no
writable = yes

[docs]
path = /srv/samba/docs
read only = yes
guest ok = yes
browsable = yes

[tmp]
path = /tmp
read only = No
EOF1
```


Create directories for `docs` share

```bash
mkdir -p /srv/samba/docs/
```

Create test file in `docs` share

```bash
echo "Test text"  > /srv/samba/docs/testfile.txt
```

Restart Samba services

```bash
systemctl restart {smb,nmb} 
```

We have configured 3 shared resource:

* homes - user writable home directory (each authenticated user will see here own home directory)
* docs - public readonly resource
* tmp - public writable /tmp directory


Let's see them

### Register existing Linux user in Samba system

Type some password that will be used for Samba access only.

```bash
smbpasswd -a student
```

Check Samba shared resources

```bash
smbclient -L localhost -U student
```


### Access Samba share via Windows

* Open the "File Explorer" and on the left-panel right-click on "This PC".
* Select "Add a network location", Then from menu select: 
* _Choose custom network location_
* _Next_
* In "**Internet or network address**" type:
`\\<ip address>\homes`
* Either go with `Next` or click on `Browse`

> `<ip address>` will be your address in the same subnet with Windows.

After entering credentials of user `student`, you should be able to see `student`'s home directory.

Try access one level up.
You should now see 3 shared resources:

* homes - user writable home directory (each authenticated user will see here own home directory)
* docs - public readonly resource
* tmp - public writable /tmp directory

Add text to test file and check via Windows Explorer that text was added

```bash
echo "Test text 2"  >> /srv/samba/docs/testfile.txt
```

### Mount Samba share

`cifs-utils` package we installed before is for mounting Samba share to Linux.

```bash
mkdir /srv/smbsharemnt ; mount -t cifs -o username=student,pass=123456 //10.10.x.1/docs /srv/smbsharemnt
```

Now you have mounted the Samba share locally to `/srv/smbsharemnt`. 
Check:

```bash
df -h | grep '/srv/smbsharemnt'
```

Check file we created, but from mount point

```bash
cat /srv/smbsharemnt/testfile.txt
```

Add text

```bash
echo "Test text"  >> /srv/samba/docs/testfile.txt
```

Check again

```bash
cat /srv/smbsharemnt/testfile.txt
```

### PRACTICE

Mount Trainer's Samba share `docs` to your `media` directory


