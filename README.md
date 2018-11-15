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

After some setup values, each network's details will be in a network 'block'. A recommended configuration is below (and included in the files):
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
https://thepihut.com/blogs/raspberry-pi-tutorials/19668676-renaming-your-raspberry-pi-the-hostname
TODO: Expand this section

## Connecting to the hotspot
The ssid of the hotspot is set to `penguinpi:xx:xx:xx` where `xx:xx:xx` will correspond to the end of your MAC address.
If you wish to change this, edit the `/etc/hostapd/hostapd.conf` file and change the following option: `ssid=myNewHotSp0t`.
The default password is `egb439123`. It can be also changed in the `hostapd.conf` file.
The IP address of the robot will be `192.168.50.5`. This will also be the default gateway IP for any device connecting to the hotspot.

## Using ethernet cable
The ethernet port can be used. It will be assigned an IP address from the DHCP server. If you need the IP address to stay the same, it is highly recommended you don't use a static IP, but try using a hostname. 
If you still must have a static IP address, do not edit the `/etc/network/interfaces` file. Since Raspian Jessie, you should use the `/etc/dhcpcd.conf` file to set a static IP address.
Follow the instructions here: https://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip-address/74428#74428 

If you want access the internet over the PenguinPis wifi hotspot (basically using the raspberry pi as a wifi router, you will have to enable IP forwarding.
Edit the `sysctl.conf` file:
```shell
sudo nano /etc/sysctl.conf
```
look for the line
```shell
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
```
and remove the # so it is
```shell
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

## Debugging wifi
You may need a keyboard and screen if you can't remotely access the PenguinPi. 
The state of the autohotspot service can be viewed with the command:
```shell
systemctl status autohotspot.service
```
Check this for any errors.