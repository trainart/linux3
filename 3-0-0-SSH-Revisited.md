# Linux Network Server (level 3) <br /> Լինուքս ցանցային սերվեր (փուլ 3)

## SSH Revisited

This is to revisit SSH and learn more advanced things about it.


### Student account's @trainer Linux (TO BE DONE BY TRAINER ONLY)

According to the number in student participant list trainer will create accounts for all students (student1, student2, ...)

Create accounts for each student & assign password (123456)

```bash
for i in {1..15}; do useradd -m student$i && echo "student$i:123456" | chpasswd; done
```

Check

```bash
tail -5 /etc/passwd ; tail -5 /etc/shadow
```

### Trainer account @student Linux (TO BE DONE BY STUDENTS ONLY)

```bash
useradd trainer && echo "trainer:123456" | chpasswd
```


### Generate SSH key pair

Each student can use `ssh-keygen` on his local system to generate public and private key pair.

Run below command as `student`

```bash
ssh-keygen
```

Just go forward with _Enter_' <br>
(you may add security by specifying the passphrase, but it has to be entered every time the key is used for authentication).

> _As a result we will get two new keys:_ 
> * `~/.ssh/id_rsa`
> * `~/.ssh/id_rsa.pub`<br><br>
>_KEEP IN SECRET `~/.ssh/id_rsa`_<br> 
>_it is your **private key**_<br>
>_`~/.ssh/id_rsa.pub` is your **public key**_ <br>
>_It could be copied to the location you want access without password._


Now securely copy your public key the `~/.ssh/id_rsa.pub` file to the
`~/.ssh` directory on the remote system (trainer's linux system), using `ssh-copy-id`:


Student's should do

>Before running change `{1-15}` to your number in the participant list

Run below command as `student`

Enter password last time. After that you should be able to login without password

```bash
ssh-copy-id student{1-15}@[trainer IP]
```

Now try to login - you should be able to login without password

```bash
ssh student{1-15}@[trainer IP]
```


Trainer should do

```bash
ssh-copy-id trainer@[each student IP]
```

After that trainer should be able to login without password

```bash
ssh trainer@[each student IP]
```

### Restricting key-based SSH access to particular command

Each student now has access to trainer system.

Trainer will now restrict access to one command `mc`
(SHOULD BE DONE ONLY BY TRAINER)

```bash
#!/bin/bash
for i in {1..15}; do
    user="student$i"
    authorized_keys="/home/$user/.ssh/authorized_keys"
        # Check if the user exists and has authorized_keys file
    if id "$user" &>/dev/null && [ -f "$authorized_keys" ]; then
        # sed -i.bak will create the backup automatically
        sed -i.bak 's/^ssh-/command="mc" &/' "$authorized_keys"    
        echo "Updated $authorized_keys (backup created as $authorized_keys.bak)"
    else
        echo "User $user or authorized_keys file not found, skipping..."
    fi
done
```

Now if you try to login again at trainer system, you should only get one command `mc` running anf after exiting it you will be logged out.


#### PRACTICE

Do that same for trainer access to you system.

Change `/home/trainer/.ssh/authorized_keys` in your system <br>
and add `command="mc" ` before ` ssh-rsa ...`


### Restricting key-based SSH access to particular IP addresses

So we understand that `~/.ssh/authorized_keys` file 
not only stores the public keys of connecting users 
but also allows to specify some additional configuration entries for each key.

We have tried `command` option above.
There are other useful options.

> Options are comma-separated and are documented in the `man sshd`, 
> under the section `"AUTHORIZED_KEYS FILE FORMAT"`. 
> 
> Here are the most useful ones:
> 
> * **from="<hostname/ip>"**  _- Prepending from="*.example.com" to the key line would only allow public-key authenticated login if the connection was coming from some host with a reverse DNS of example.com. You can also put IP addresses in here. This is particularly useful for setting up automated processes through keys with null passphrases._
>
>> 
>> * - Matches zero or more characters
>> ? - Matches exactly one character
>> ! - Negates the host pattern match
>> 
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
> * **restrict** -  NEW option that automatically enables all of these restrictions (most of the above):
>   * no-port-forwarding
>   * no-agent-forwarding
>   * no-X11-forwarding
>   * no-pty (no shell access)
>   * no-user-rc (no ~/.ssh/rc execution)
> 

**So you can use only `restrict` !** 

#### PRACTICE

Add `from=127.0.0.1` to the line of trainer public key, before "**ssh-rsa ...**" 
in `/home/traner/.ssh/authorized_keys` file: 

the line should look like

```bash
from=127.0.0.1,command="mc" ssh-rsa ...
```

Check 

```bash
cat /home/traner/.ssh/authorized_keys
```

Now try connecting:  
* from allowed IP address (127.0.0.1)

```bash
trainer@127.0.0.1 
```

You will get `mc` command output

* trainer trying from other IP address

Should not be able to login without password


Now edit `/home/traner/.ssh/authorized_keys` again and add trainer's IP address


the line should look like

```bash
from="127.0.0.1,[TRAINER IP]",command="mc" ssh-rsa ...
```

Now trainer should be able to login


#### Restrict most options with `restrict`

Now edit `/home/traner/.ssh/authorized_keys` again and add `restrict` option in the beginning.

the line should look like

```bash
restrict,from="127.0.0.1,[TRAINER IP]",command="mc" ssh-rsa ...
```

Now trainer should not be able to login.

Why ?



### Resume

Such restriction can be useful for remote backups scripts, 
as it can ensure that your remote user can only execute the 
expected command - and not anything else.
