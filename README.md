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
