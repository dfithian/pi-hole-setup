# Pi Hole
## Installation
* get raspberry pi
* download and unzip raspbian lite
* get etcher
* flash sd card with raspbian using etcher
* touch file "ssh" in the "boot" drive root folder of the sd card
* put the raspberry pi case and mount heat syncs and turn on and plug into router
* get private ip address of router
* get private ip address of pi from router
* ssh to pi: `pi@<ip-address>` password `raspberry`
* change password: `passwd`
* sudo apt-get update
* sudo apt-get upgrade
* install pi hole: `curl -sSL https://install.pi-hole.net | sudo bash`
  * save the password that is needed for the admin screen
* configure local DNS to use private ip of raspberry pi, apply settings
* browse to `http://192.168.1.121/admin/`
  * use the password from above

## Upgrade
* ssh to pi: `pi@<ip-address>`
* sudo apt-get update
* sudo apt-get upgrade
* `pihole -up`

## DNS over HTTPS Steps
* https://docs.pi-hole.net/guides/dns-over-https/
* `wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz`
* `tar -xvzf cloudflared-stable-linux-arm.tgz`
* `sudo cp ./cloudflared /usr/local/bin`
* `sudo chmod +x /usr/local/bin/cloudflared`
* `cloudflared -v` # verify
* `sudo useradd -s /usr/sbin/nologin -r -M cloudflared`
* `sudo vi /etc/default/cloudflared`
  * ```
    # Commandline args for cloudflared
    CLOUDFLARED_OPTS=--port 5053 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query
    ```
* `sudo chown cloudflared:cloudflared /etc/default/cloudflared`
* `sudo vi /lib/systemd/system/cloudflared.service`
  * ```
    [Unit]
    Description=cloudflared DNS over HTTPS proxy
    After=syslog.target network-online.target

    [Service]
    Type=simple
    User=cloudflared
    EnvironmentFile=/etc/default/cloudflared
    ExecStart=/usr/local/bin/cloudflared proxy-dns $CLOUDFLARED_OPTS
    Restart=on-failure
    RestartSec=10
    KillMode=process

    [Install]
    WantedBy=multi-user.target
    ```
* `sudo systemctl enable cloudflared`
* `sudo systemctl start cloudflared`
* `sudo systemctl status cloudflared`
* `dig @127.0.0.1 -p 5053 google.com` # verify

## Multiple WLANs

Source: https://wiki.dd-wrt.com/wiki/index.php/Multiple_WLANs

* This is _not_ a WAP
* Do everything the article says up to "3.2 Command Method"
* Change `dhcp-option=br1,6,[DNS IP 1],[DNS IP 2]` to `dhcp-option=br1,6,8.8.8.8`
* Under "Restricting Access", did this:
```bash
iptables -t nat -I POSTROUTING -o `get_wanface` -j SNAT --to `nvram get wan_ipaddr`
iptables -I FORWARD -i br1 -m state --state NEW -j ACCEPT
iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
iptables -I FORWARD -i br1 -o br0 -m state --state NEW -j DROP
iptables -I FORWARD -i br0 -o br1 -m state --state NEW -j DROP
iptables -I FORWARD -i br1 -d `nvram get wan_ipaddr`/`nvram get wan_netmask` -m state --state NEW -j DROP
iptables -I FORWARD -i br1 -d `nvram get lan_ipaddr`/`nvram get lan_netmask` -m state --state NEW -j DROP
iptables -t nat -I POSTROUTING -o br0 -j SNAT --to `nvram get lan_ipaddr`
```
* Always be sure to save/apply/reboot whenever in doubt. Incremental progress is better than no progress.

## Other Notes
### From apt-get upgrade:
```
apt-listchanges: Can't set locale; make sure $LC_* and $LANG are correct!
Reading changelogs... Done
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "en_US.UTF-8",
	LANG = "en_GB.UTF-8"
    are supported and installed on your system.
```

## GCP

https://github.com/rajannpatel/Pi-Hole-on-Google-Compute-Engine-Free-Tier-with-Full-Tunnel-and-Split-Tunnel-Wireguard-VPN-Configs/blob/master/CONFIGURATION.md
