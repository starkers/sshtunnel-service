# sshtunnel-service
Manage autossh with supervisord

Took a little tweaking but this lets me stop-start autosshd from supervisor!

There are numour "issues" with autossh that are frankly shit.
EG.. it will launch a regular ssh command but not fucking notice that its dead (or I'm just impatient)
This script performs these following tasks then:

- the variables are in a human understandable format (methinks) see tunnel.conf
- supervisord takes care of the script
-- the script takes care of `autossh` AND its child `ssh` proc

TODO:
- BACKOFF supervisord  (EG: the script doesn't work.. stop attempting to connect and backoff for an hour.. -sic fail2ban)
- imporve the output to include timestamps for the log file
- whack in a logrotate config
- monit/pgagent script to alert if its down
- more testing on worse connections

MAYBE:
- make the script check if its online before attempting autossh anyway..
- test using an echo service on the remote host?


### Sample config
- /etc/myconfig/tunnel.conf

```
#Connect to this server
REMOTE_HOST[$i]="remote.server.net"

# Connect to it on this port (I have added "Port 443" to the sshd_config)
REMOTE_PORT[$i]=443

# Using this ssh key  (ssh-keygen -b 4096 -f /etc/myconfig/tunnel.key -C `hostname` )"
KEY[$i]="/etc/myconfig/tunnel.key"

# Connect as this user
REMOTE_USER[$i]="remote_username"

# Forward this port (on the client)
TUNNEL_PORT_LOCAL[$i]=22

# and expose it (on the remote server) on this port
# If not logging as root this port must be higher 1024
TUNNEL_PORT_REMOTE[$i]=5032
```

# usage

## Start it
```
supervisorctl start sshtunnel
```

## stop
```
supervisorctl stop  sshtunnel
```

## restart
```
supervisorctl restart sshtunnel
```

## view log
```
supervisorctl tail -f sshtunnel
```
