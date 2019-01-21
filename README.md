# Steps
* get raspberry pi
* download and unzip raspbian lite
* get etcher
* flash sd card with raspbian using etcher
* touch file "ssh" in the "boot" drive root folder of the sd card
* put the raspberry pi case and mount heat syncs and turn on and plug into router
* get private ip address of router
* get private ip address of pi from router
* ssh to router: pi@<ip-address> password raspberry
* change password: `passwd`
* sudo apt-get update
* sudo apt-get upgrade
* install pi hole: `curl -sSL https://install.pi-hole.net | sudo bash`

# Other Notes
## From apt-get upgrade:
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
