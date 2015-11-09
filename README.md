# sshtunnel-service
Manage autossh with supervisord

Took a little tweaking but this lets me stop-start autosshd from supervisor!

I'm calling it sshtunnel for ease



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

supervisorctl start sshtunnel
supervisorctl stop  sshtunnel
supervisorctl restart sshtunnel
supervisorctl tail -f sshtunnel
