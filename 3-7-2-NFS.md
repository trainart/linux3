# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## NFS (Network FileSystem)
_(based on https://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-setup-nfs-server-on-centos-7-rhel-7-fedora-22.html)_

_NFS usage: 
    File / Folder sharing between *nix systems
    Allows to mount remote filesystems locally
    Can be acted as Centralized Storage system
    It can be used as a Storage Domain ( Datastore) for VMware and other Virtualization Platform.
    Allows applications to share configuration and data files with multiple nodes.
    Allows to have updated files across the share._

Important Services

The following are the important NFS services, included in `nfs-utils` packages.

* rpcbind : Converts RPC program numbers into universal addresses.
* nfs-server :  Enables the clients to access NFS shares.
* nfs-lock / rpc-statd : Implements NFS file locking/lock recovery when an NFS server crashes and reboots.
* nfs-idmap : Translates User and Group IDs into names and vice versa

Important Configuration Files

You would be working mainly on below configuration files, to setup NFS server and Clients.

* /etc/exports : Main NFS config file, controls which file systems are exported to remote hosts and sets options.
* /etc/fstab : Used to control what file systems including NFS directories are mounted when the system boots.

### Install and Configure NFS Server

Install NFS Package
```bash
yum -y install nfs-utils libnfsidmap
```

Enable and start NFS services
```bash
systemctl enable --now rpcbind ;\
systemctl enable --now nfs-server ;\
systemctl start rpc-statd ;\
systemctl start nfs-idmapd
```

**If Firewalld is enabled** set proper settings 
```bash
firewall-cmd --state
firewall-cmd --list-services
firewall-cmd --permanent --zone public --add-service mountd
firewall-cmd --permanent --zone public --add-service rpc-bind
firewall-cmd --permanent --zone public --add-service nfs
firewall-cmd --reload
```

Create NFS Share
```bash
mkdir -p /srv/nfsshare/sharedir
```

Modify /etc/exports and add the following 
(Change the IP range if needed for clients of that share):
```bash
cat <<EOF1 >> /etc/exports
/srv/nfsshare/sharedir 192.168.0.0/16(rw,sync,no_root_squash)
EOF1
```
Options meaning:
* rw : Writable permission to shared folder
* sync :  all changes to the according filesystem are immediately flushed to disk; the respective write operations are being waited for.
* no_root_squash : By default, any file request made by user root on the client machine is treated as by user nobody on the server. (Exactly which UID the request is mapped to depends on the UID of user “nobody” on the server, not the client.) If no_root_squash is selected, then root on the client machine will have the same level of access to the files on the system as root on the server.

More info about the options: `man exports`

Activate the export
```bash
exportfs -r
```

More info:
```bash
exportfs -v : Displays a list of shares files and export options on a server
exportfs -a : Exports all directories listed in /etc/exports
exportfs -u : Unexport one or more directories
exportfs -r : Reexport all directories after modifying /etc/exports
```
Check the export availability
```bash
showmount -e
```


### Install and Configure NFS Client

Install NFS Package
```bash
yum -y install nfs-utils libnfsidmap
```

Enable and start NFS services
```bash
systemctl enable --now rpcbind
```

Check the NFS server share availability 
```bash
showmount -e 192.168.1.1
```
Mount NFS share
```bash
mkdir /mnt/nfsshare
mount 192.168.1.1:/srv/nfsshare/sharedir /mnt/nfsshare
```

Verify the mounted share on client
```bash
mount | grep nfs  or  df -hT
```

Create a test file on the mounted directory to verify the read and write access on NFS share
```bash
touch /mnt/nfsshare/testfile
```
Try to unmount shared directory
```bash
umount /mnt/nfsshare
```

#### Automount NFS Shares
To mount the shares automatically on every reboot, 
following line should be added to `/etc/fstab`
(correct IP address as needed)
```bash
192.168.1.1:/srv/nfsshare/sharedir /mnt/nfsshare nfs rw,sync,hard,intr 0 0
```

Manually check if it works
```bash
mount -a ; df -hT
```

Reboot to really check
```bash
init 6
```

> In addition you can configure AutoFS to mount NFS share only when they are accessed by a user
> https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/how-to-install-and-configure-autofs-on-centos-7-fedora-22-ubuntu-14-04.html

