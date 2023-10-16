

The n2n peer-to-peer VPN, it's great software but it may not check the status.

That's the Debian or Ubuntu system.
>Especially when deployed at remote sites.

## Check script
```shell
cat <<EOF> /usr/local/n2n_check.sh
#! /bin/bash
# You need to change the n2n_ip 
ip=10.10.10.251
n2n_ip=10.10.10.250
n2n_ser=main.domain.xyz:25000
ping -c 2 \$ip
if [ \$? -eq 0 ]; then
        echo "ping \$ip success!"
else
        kill \$(ps -ef | grep edge | grep -v grep| awk '{print $2}')
        /usr/sbin/edge -a \$n2n_ip -c 1024 -k 1024 -l \$n2n_ser -r
        sleep 1 && echo "will ping test."
        ping -c 2 \$ip
fi
EOF
chmod +x /usr/local/n2n_check.sh
```

## Schedule server

```shell
cat <<EOF> /etc/systemd/system/n2nPeerToPeer.timer
[Unit]
Description=n2n peer to peer VPN

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Unit=n2nPeerToPeer.service

[Install]
WantedBy=timers.target
EOF
```

## Running server

```shell
cat <<EOF> /etc/systemd/system/n2nPeerToPeer.service
[Unit]
Description=N2N Peer To Peer VPN, tunnel live check, and restart.

[Service]
User=root
ExecStart=/bin/bash /usr/local/n2n_check.sh
KillMode=process
EOF
```

## Enable server

```shell
systemctl daemon-reload
systemctl enable  n2nPeerToPeer.timer
systemctl enable --now n2nPeerToPeer.timer
systemctl status n2nPeerToPeer
```
