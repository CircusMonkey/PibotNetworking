# PiNetworking

## Getting started
This guide will instruct you how to setup the PenguinPi robots networking.

### What the new configuration will do
Remember multiple wifi networks and setup your preference of order in which to connect.
If none are available, a wifi hotspot will be generated.

### Updating the configuration of an old PenguinPi
Only follow these instructions if your PenguinPi has not been setup to make it's own hotspot.
If your PenguinPi is already able to make it's own hotspot, skip to the next step.
The `update_networking_script.sh` will perform most of the nessecary steps. Please only run this file once as below:
```
sudo ./update_networking_script.sh
```

## Setting up wifi networks to remember

### wpa_supplicant.conf
The `wpa_supplicant.conf` file stores the details about the wifi networks to connect to.
Edit this file (using nano, or your prefered text editor):
```shell
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

After some setup values, each network's details will be in a network 'block'. A recommended configuration is below:
```shell
country=AU
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

# Your mobile hotspot. You can fall back on this if everything else fails.
network={
	ssid="my_mobile_hotspot"
	psk=9187c952cf11306f12f9e421b6605f531d5b481c3840b823dff4e846fbedb84a
    priority=4
}

# Your home network.
network={
	ssid="my_home_wifi"
	psk=bacfce796f8677ff7356ce26bb55c235c091602f61dcfb53987879826523fde5
    priority=3
}

# The EGB439 network on S9
network={
	ssid="EGB439"
	psk="egb439123"
    priority=2
}

# Your QUT student wireless network access to use elsewhere around campus
network={
	ssid="QUT"
	priority=1
	key_mgmt=WPA-EAP
	pairwise=CCMP
	eap=PEAP
	identity="n0000001"
	password=hash:e5c53515388155f527917a9ee5231538
	phase1="peaplabel=0"
	phase2="auth=MSCHAPV2"
}
```
Notes:
* There is some extra setup options for enterprise networks.
* The EGB439 wifi password is stored in plain text (notice the quote marks ""). This is ok for that network, but not for your passwords for your QUT login. The other networks have a lot of hex characters. We will do this for your wifi in the next section.
* The priority allows you to preference one network over the other, in the case there are multiple available. The higher priority networks will be chosen first.


### Hiding your passwords
For personal networks, you can use the `wpa_passphrase` tool.
```shell
wpa_passphrase my_mobile_hotspot my_password
```
will output a network block.
```python
network={
	ssid="my_mobile_hotspot"
	#psk="my_password"
	psk=bebb15a62c377f565f9e17280df9e2b9ece627325be93653c0b98556c9216f49
}
```
 Update the `psk=` line in the `/etc/wpa_supplicant/wpa_supplicant.conf` file to the new value.

 For the QUT enterprise network, hash your password with:
 ```bash
echo -n 'YOUR_REAL_PASSWORD' | iconv -t utf16le | openssl md4
 ```

This will output 
```bash
(stdin)= e5c53515388155f527917a9ee5231538
```

Ignore the `(stdin)= ` and copy and paste the hashed password into the QUT network block. Ensure it begins with `password=hash:`. The final result will be similar to `password=hash:e5c53515388155f527917a9ee5231538`.

Save your changes to the `wpa_supplicant.conf` file.

Now we need to erase the history so that noone can get your password by looking at the commands you typed.
Find the history file and erase all lines containing your passwords.
```bash
nano ~/.zsh_history
```
Save the file. The passwords are still accessable by pressing the up arrow.

and reboot your Raspberry Pi (`sudo reboot`).

## Setting up a hostname

## Connecting to the hotspot

