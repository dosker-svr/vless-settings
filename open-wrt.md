### /etc/init.d/tun2socks

```sh
#!/bin/sh /etc/rc.common

USE_PROCD=1

# starts after network starts
START=40
# stops before networking stops
STOP=89

PROG=/usr/bin/tun2socks
IF="tun0"
PROTO="socks5"
HOST="127.0.0.1"
PORT="2080"

start_service() {
        if [ -n "$METHOD_USER" ] && [ -n "$PASS" ]; then
            PROXY_URI="$PROTO://$METHOD_USER:$PASS@$HOST:$PORT"
        else
            PROXY_URI="$PROTO://$HOST:$PORT"
        fi

        procd_open_instance
        procd_set_param command "$PROG" -device "$IF" -proxy "$PROXY_URI"
        procd_set_param stdout 1
        procd_set_param stderr 1
        procd_set_param respawn ${respawn_threshold:-3600} ${respawn_timeout:-5} ${respawn_retry:-5}
        procd_close_instance
}
```

