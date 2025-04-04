### /etc/config/network
```sh
config interface 'xray'
        option device 'tun0'
        option proto 'static'
        option ipaddr '172.16.250.1'
        option netmask '255.255.255.0'

```

### /etc/config/firewall
```sh
config zone
        option name 'xray'
        option forward 'REJECT'
        option output 'ACCEPT'
        option input 'REJECT'
        option masq '1'
        option mtu_fix '1'
        option device 'tun0'
        option family 'ipv4'

config forwarding
        option name 'lan-tun'
        option dest 'tun'
        option src 'lan'
        option family 'ipv4'
```
Рестартуем службу:
$ service network restart

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

Далее:
```sh
chmod +x /etc/init.d/tun2socks
ln -s ../init.d/tun2socks /etc/rc.d/S40tun2socks
service tun2socks start
```


## xray
### /etc/xray/config.json
```sh
{
    "log": {
      "loglevel": "warning"
    },
    "routing": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "port": "0-65535",
          "outboundTag": "proxy"
        }
      ]
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": "2080",
            "protocol": "socks",
            "settings": {
              "auth": "noauth",
              "udp": true,
              "ip": "127.0.0.1"
            }
      }
    ],
  "outbounds": [
    {
        "tag": "proxy",
        "protocol": "vless",
        "settings": {
            "vnext": [
                {
                    "address": "",
                    "port": 443,
                    "users": [
                        {
                            "id": "",
                            "flow": "",
                            "encryption": "none",
                            "level": 0
                        }
                    ]
                }
            ]
        },
        "streamSettings": {
            "network": "tcp",
            "security": "reality",
            "realitySettings": {
                "publicKey": "",
                "fingerprint": "chrome",
                "serverName": "mirror.timeweb.ru",
                "shortId": "",
                "spiderX": "/"
            }
        }
    },
    {
      "protocol": "freedom",
      "tag": "direct"
    }
  ]
}
```

### /etc/config/xray
