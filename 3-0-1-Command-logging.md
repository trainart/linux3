# Linux Network Server (level 3) <br /> Linux ցանցային սերվեր (փուլ 3)

## Logging User Commands 

Below solution can help to keep track of commands that users run. 

1. First of all let us talk about 'PROMPT_COMMAND' Bash variable.

Try this
```bash
PROMPT_COMMAND='echo -n "DATE is: " && date "+%Y-%m-%d %H:%M:%S"'
```

Now run any commands. You will see the above echo runs before each command.
To remove that setting just run 

```bash
unset PROMPT_COMMAND
```

2. Since we want to log the commands let us create separate logging config for that.

Add new config `/etc/rsyslog.d/cmdlog.conf` to log message sent to 'local0' facility to '/var/log/cmnd.log'

```bash
cat > /etc/rsyslog.d/cmdlog.conf << "ENDTEXT"
local0.*                                                  /var/log/cmnd.log
ENDTEXT

```

and restart the rsyslog service:

```bash
systemctl restart rsyslog
```

Check that logging works

```bash
logger  -p local0.info "Logging this command to LOCAL0 - this message should appear in /var/log/cmnd.log"
```

```bash
tail /var/log/cmnd.log
```

3. Now let us create script to log user commands to '/var/log/cmnd.log'

### Bash code to log user commands 

```bash
cat  > ~/cmdlog.sh  << "ENDSCRIPT"
last_command=""

log_command() {
    # current time
    local current_time=$(date "+%Y-%m-%d %H:%M:%S")

    # user name, working directory, command history
    local user_name=$(whoami)
    local working_directory=$(pwd)
    local command_history=$(history 1 | sed 's/^[ ]*[0-9]*[ ]*//;s/^[^]]*][ ]*//;s/^ *//')
    local command=$(echo "$command_history" | awk '{print $1}')

    # ignore empty command
    if [[ -z "$command_history" || "$command_history" == "$last_command" ]]; then
        return
    fi

    # last command update
    last_command="$command_history"

    # message
    local mssg="$current_time, USR:$user_name, DIR:$working_directory, CMD:$command_history"

    # log command to local0 facility (ensure it is configured in rsyslog to log to proper file)
    logger  -p local0.info "$mssg"
}

# define "PROMPT_COMMAND" variable to run above 'log_command' function
# remember that "PROMPT_COMMAND" runs before each bash command execution 
PROMPT_COMMAND="log_command; $PROMPT_COMMAND"

ENDSCRIPT
chmod +x ~/cmdlog.sh
```

Now run to check it works for your current user only. 
Add it to the ~/.bashrc file

```bash
echo ""~/cmdlog.sh" >> ~/.bashrc

```

Since '~/.bashrc' is being run upon login, you either need to logout and login, or run new bash process.

After that you can run some commands and check if they are logged in `/var/log/cmnd.log`

```bash
tail /var/log/cmnd.log
```


### Modified Bash code to ignore some commands  

If we want not to log some basic commands our script will look like below.

```bash
cat  > ~/cmdlog.sh  << "ENDSCRIPT"
ignore_commands="cd pwd ls htop top df du free uptime who w uname cat more less tail head grep find ping ss netstat ifconfig traceroute nano joe mc"

last_command=""

log_command() {
    # current time
    local current_time=$(date "+%Y-%m-%d %H:%M:%S")

    # user name, working directory, command history
    local user_name=$(whoami)
    local working_directory=$(pwd)
    local command_history=$(history 1 | sed 's/^[ ]*[0-9]*[ ]*//;s/^[^]]*][ ]*//;s/^ *//')
    local command=$(echo "$command_history" | awk '{print $1}')

    # ignore empty command
    if [[ -z "$command_history" || "$command_history" == "$last_command" ]]; then
        return
    fi

    # last command update
    last_command="$command_history"

    # ignore commands in ignore_commands
    for ignore_cmd in $ignore_commands; do
        if [[ "$command" == "$ignore_cmd" ]]; then
            return
        fi
    done

    # message
    local mssg="$current_time, USR:$user_name, DIR:$working_directory, CMD:$command_history"

    # log command to local0 facility (ensure it is configured in rsyslog to log to proper file)
    logger  -p local0.info "$mssg"
}

# define "PROMPT_COMMAND" variable to run above 'log_command' function
# remember that "PROMPT_COMMAND" runs before each bash command execution 
PROMPT_COMMAND="log_command; $PROMPT_COMMAND"
ENDSCRIPT
chmod +x ~/cmdlog.sh
```


### Adding command logging for ALL users

In order to enable command logging for ALL users we need to add that script to be run in global profile config.

But first let us remove it from our current user. 
We need to delete last line we added in `~/.bashrc`

Check last line
```bash
tail  ~/.bashrc
```

Remove last line
```bash
sed -i '$d' ~/.bashrc
```

Check again
```bash
tail  ~/.bashrc
```

> Explanation:
> `-i` - edit the file in-place
> `$d` - delete the last line


Now let us add global config for all users.
One way of doing that is to copy our script to `/etc/profile.d` directory.

```bash
cp ~/cmdlog.sh /etc/profile.d
```
NOTE that this should be done as `root` !


Now we can become any user, run some commands and the check they all are logged.



### BONUS ! 

#### Sending messages via Telegram API

We can not only to log the user commands locally, but also send them to some Telegram Bot.

In order to do that we need to have working Telegram bot, have that Telegram bot's token and chat ID.
And in above code instead of 'logger' command put a command like below.
(remember to pur proper `API_TOKEN` and `CHAT_ID` there).
```bash
curl -s -X POST "https://api.telegram.org/bot<API_TOKEN>/sendMessage" --data-urlencode "chat_id=<CHAT_ID>" --data-urlencode "text=$mssg" >/dev/null 2>&1
```

For Telegram message we may want each parameter to be on new line. So we can have modified version of below line:
`local message="TIME:$current_time"$'\n'"USER:$user_name"$'\n'"DIR:$working_directory"$'\n'"CMD:$command_history"`


##### Creating a Telegram Bot and Obtaining Token and Chat ID
To create a Telegram bot and obtain the necessary token and chat ID, follow these steps:

* Open the Telegram app and search for the “BotFather” bot.
* Start a conversation with the BotFather by sending `/start` and send the `/newbot` command to create a new bot.
* Provide a name and username for your bot as prompted by the BotFather.
* Upon successful creation, you will receive an API token for your bot. Make sure to keep this token secure and confidential.
* To obtain the chat ID, follow these steps:
* Start a conversation with the “IDBot” bot.
* After sending `/start`, send the `/getid` command to obtain the chat ID.
* The chat ID will be displayed in the chat window. Make sure to keep this chat ID secure and confidential.

With the Telegram bot token and chat ID in hand, you’re ready to proceed with adding the to the above script.

