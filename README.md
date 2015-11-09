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


### /etc/supervisor/conf.d/sshtunnel.conf

Note the stopsignal.
The script its able to trap the TERM signal into a function which lets it cleanup the actual autossh underneath it

```
[program:sshtunnel]
command=/root/bin/sshtunnel
directory=/tmp
user=root
autostart=true
autorestart=true
stdout_logfile=/var/log/sshtunnel.log
redirect_stderr=true
numprocs=1
stopsignal=TERM
```

### /root/bin/sshtunnel

```
#!/usr/bin/env bash

#load the config
CONFIG=/etc/myconfig/tunnel.conf
if [ -f "$CONFIG" ]; then
  source "$CONFIG"
else
  echo "=> ERROR: no config found @ $CONFIG"
  exit 1
fi


# supervisord should be able to send a SIGINT to stop this services/daemon
trap finish TERM
finish(){
  kill $JOB
  echo "=> recieved SIGINT and exited OK"
  exit 0
}


echo "=> Tunneling localhost:${TUNNEL_PORT_LOCAL} to ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PORT} (on port ${TUNNEL_PORT_REMOTE}) using key ${KEY}"
autossh \
  -M 0 \
  -o "ServerAliveInterval 20" -o "ServerAliveCountMax 3" -o "StrictHostKeyChecking=no" \
  -NR ${TUNNEL_PORT_REMOTE}:127.0.0.1:${TUNNEL_PORT_LOCAL} \
  -i "${KEY}" \
   ${REMOTE_USER}@${REMOTE_HOST} &

# record the JOB (should be backgrounded)
JOB="$(jobs -p)"

while true ; do
  if [ -z "$JOB" ]; then
    echo "=> $$ job: $JOB not running"
  else
    echo "=> $$ job: $JOB running"
  fi
  sleep 10
done

echo "=> Exiting"
```
