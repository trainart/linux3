# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)

## SSH Advanced


### Restricting key-based SSH access to particular IP addresses

Rather than just storing the public keys of connecting users 
`~/.ssh/authorized_keys` file also allows to 
specify some additional configuration entries for each key:



> Options are comma-separated and are documented in the `man sshd`, 
> under the section `"AUTHORIZED_KEYS FILE FORMAT"`. 
> 
> Here are the most useful ones:
> 
> * **from="<hostname/ip>"**  _- Prepending from="*.example.com" to the key line would only allow public-key authenticated login if the connection was coming from some host with a reverse DNS of example.com. You can also put IP addresses in here. This is particularly useful for setting up automated processes through keys with null passphrases._
>
>> ```bash
>> * - Matches zero or more characters
>> ? - Matches exactly one character
>> ! - Negates the host pattern match
>> ```
>
> * **command="<command>"**  _- Means that once authenticated, the command specified is run, and the connection is closed. Again, this is useful in automated setups for running only a certain script on successful authentication, and nothing else._
> 
> 
> * **no-agent-forwarding**  _- Prevents the key user from forwarding authentication requests 
> to an SSH agent on their client, using the -A or ForwardAgent option to ssh._
>
> 
> * **no-port-forwarding** - _Prevents the key user from forwarding ports using -L and -R._
>
> 
> * **no-X11-forwarding**  - _Prevents the key user from forwarding X11 processes._
>
> 
> * **no-pty** - _Prevents the key user from being allocated a tty device at all (does not allow interactive login)_
>    

#### PRACTICE 1

Add to your following to the line of you public key, before "**ssh-rsa ...**" 
in `~/.ssh/authorized_keys` file: 

```bash
from="127.0.0.1,10.1.10.*",command="w" ssh-rsa ...
```

Now try connecting:  
* from allowed IP address

You will get `w` command output
* from other IP address

You will get password prompt

Such restriction can be useful for remote backups scripts, 
as it can ensure that your remote user can only execute the 
expected command - and not anything else.

> Another useful restrictions are, to disable use of agent-forwarding, port-forwarding, X11 and interactive logins.
> 
> >  ```bash
> > no-agent-forwarding,no-port-forwarding,no-X11-forwarding,no-pty ssh-rsa ...
> > ```
> 
> Recent OpenSSH versions >=7.2 have additional option `restrict` in the `authorized_keys`.
> It enables all restrictions, i.e. disable the above altogether. Also any future restriction capabilities are added to this option.
> 

#### BONUS

`command=` option is not supporting arguments.
Solution is in creating additional intermediate script like one below. 

1. Create file `/opt/checkssh` and make it executable.

```bash
#!/bin/bash
if [ -n "$SSH_ORIGINAL_COMMAND" ]; then
    if [[ "$SSH_ORIGINAL_COMMAND" =~ ^ls\  ]]; then
        echo "`/bin/date`: $SSH_ORIGINAL_COMMAND" >> $HOME/ssh-command-log
        exec $SSH_ORIGINAL_COMMAND
    else
        echo "`/bin/date`: DENIED $SSH_ORIGINAL_COMMAND" >> $HOME/ssh-command-log
    fi
fi
```
> NOTE: Allowed **ls** command in the above script specifically ends with `\ `, 
> which means, there should be a space after the command (for options/arguments):
> 
> `^ls\ `
> 

`chmod +x /opt/checkssh`

2. Modify `~/.ssh/authorized_keys` file:
```bash
from="127.0.0.1,10.1.10.*",command="/opt/checkssh" ssh-rsa ...
```

3. Try run 

`ssh student@127.0.0.1 ls /opt`

`ssh student@127.0.0.1 ls -l /tmp`

The same way other restricted commands can be specified (like `rsync`, discussed below).  






### SSH as a filesystem: sshfs
Using the FUSE project with `sshfs`, it's possible to mount a remote 
filesystem over SSH. 

CentOS package for `sshfs` is available from **EPEL** repository in CentOS 7. 
Make sure you have EPEL and type:<br>

```bash
yum -y install fuse-sshfs
```

> NOTE: For CentOS 8 you need to install from **powertools** repository (disabled by default)<br>
> `dnf --enablerepo=powertools -y install fuse-sshfs`
> <br><br>
> For Ubuntu package name is just `sshfs`
> `apt install sshfs` 


> IMPORTANT: once `sshfs` is installed, it can be used by any user. Users can mount remote directories somewhere in their home
 

Syntax is like:<br>
`sshfs user@remote.host:/somedir ~/localdir  -o reconnect`

> You should have key-based access configure for that user

#### PRACTICE
1. Mount remote directory via ssh link
```bash
mkdir /opt/sshfs
sshfs student@10.1.10.1:/tmp/rs1  /opt/sshfs -o reconnect
df -h
```
> To set other UID/GID on copied files we can add:<br>
> _-o uid=1000,gid=1000_<br>
> See `man sshfs` for more options

2. Unmount it
```bash
fusermount -u /opt/sshfs
df -h
```

> Automounting can be done by adding the appropriate line to `/etc/fstab`, 
> like:
> ```bash
> sshfs#user@remote.host:/somedir /somemydir fuse uid=1000,gid=1000 0 0
> ```

 


### Fail2ban – protect from brute-force attacks (SSH example)

If you pay attention to application logs for SSH service, you may see repeated, systematic login attempts that represent brute-force attacks by users and bots alike.

Fail2ban can mitigate this problem by creating rules that automatically alter your firewall configuration based on a predefined number of unsuccessful login attempts. This will allow your server to respond to illegitimate access attempts without intervention from you.

#### Install and configure Fail2ban

While Fail2ban is not available in the official CentOS package repository, 
it is packaged for the EPEL project. 
EPEL, standing for Extra Packages for Enterprise Linux, 
can be installed with a release package that is available from CentOS: 

`yum -y install epel-release`

Now we should be able to install the fail2ban package:  

`yum -y install fail2ban`

Once the installation has finished, use systemctl to enable the fail2ban service:

`systemctl enable fail2ban`

`systemctl start fail2ban`

Fail2ban service keeps its configuration files in the `/etc/fail2ban` directory. There, you can find a file with default values called `jail.conf`. 
Since this file may be overwritten by package upgrades, we shouldn't edit it in-place. Instead, we'll write a new file called `jail.local`. Any values defined in `jail.local` will override those in `jail.conf`.

#### PRACTICE

1. Create `/etc/fail2ban/jail.local`

```bash
[DEFAULT]
# Ban hosts for one hour
bantime = 3600
# Ban IP that does 'maxretry' failures within 'findtime' /600 seconds - 10 minutes/
findtime = 600
maxretry = 3
# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = iptables-multiport
[sshd]
enabled = true
```

2. Restart the fail2ban service using systemctl:

`systemctl restart fail2ban`

3. Check that the service is running:

`fail2ban-client status`

4. Get more detailed information about a specific jail:

`fail2ban-client status sshd`

5. Now try to login with ssh and enter wrong password 3 or more times. 

6. After that run again:

`fail2ban-client status sshd`

You should see your IP banned.

> Also the following command will show the current firewall rules enabled, 
where you will see you IP:
> 
> ```
> iptables -L -n
> iptables -S
> ```

7. Now remove your IP from ban list by:

`fail2ban-client set sshd unbanip <IP address>`

8. Create whitelist of your subnets with the following option added to `/etc/fail2ban/jail.local`:
```bash
ignoreip = 127.0.0.1/8 192.168.0.0/16 10.0.0.0/8
```

9. Restart the fail2ban:

`systemctl restart fail2ban`

10. Now again try to login with ssh and enter wrong password 3 or more times.
You should not be banned. Run again:

`fail2ban-client status sshd`

You should not see your IP address banned.
And you should be able to login.

<br><br>


For protecting other services you may see what kind of other filters are available:

`ls /etc/fail2ban/filter.d`

After any change remember to restart the fail2ban service:

`systemctl restart fail2ban`
