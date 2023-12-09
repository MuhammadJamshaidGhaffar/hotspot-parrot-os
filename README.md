# hotspot-parrot-os

## step 1: 
check to see if the system supports access point supports for this run the command <br/>
   ```iw list``` <br/>
   ![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/b4889b6a-d743-46db-8cdc-48e399089ff0)


You should see an output as below

![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/fb063d18-b3c2-460b-bf3a-642421c6838a)


Scroll up to where it says supported interface modes and check to see if AP is listed. If so we are good to go…

## step 2: 
Now we move on the most important step setting up a virtual wireless interface on top of our existing interface. this is also done using the iw command <br/>
```sudo iw phy phy0 interface add <interface_name> type __ap```
![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/ccb5735e-e5f5-47d8-b71e-b241cf85361f)


run the above command and then run ```ifconfig -a``` to list all devices. <br/>In the above, you may need to replace phy0 to your device's physical name to see all physical devices use ```iw phy``` command.
![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/57218634-37da-4d04-82d5-5fb4767ee6ba)

So far we have created and configured a virtual interface, now to move on one of the most important steps is, to use hostapd to create the actual hotspot.

## step 3: Setting up the hostapd
Install hostapd by this command if it is not installed ```apt-get install hostapd``` <br/>
Now we first need to configure the hostapd. I am going to create a hostapd.conf file in my home directory and then open it. I am going to use vi. a sample configuration is shown below.

```
interface=<YOUR-INTERFACE-NAME>
driver=nl80211
ssid=<YOUR-HOTSPOT-NAME>
channel=7
hw_mode=g
wme_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=3
wpa_passphrase=<YOUR-PASSWORD>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/f25bac7b-09ff-45b6-b8c2-7853bd40b704)

the final step is to run it using the command : ```sudo hostapd hostapd.conf```

![image](https://github.com/MuhammadJamshaidGhaffar/hotspot-parrot-os/assets/75721211/f4df287e-2903-4199-a0bc-9e067665ed8f)

now if you were to scan for the WiFi you will see your hotspot. If you try to connect now with your password you can authorize on to the network but your device will fill fail to obtain any configurations either you can set it manually which is quite troublesome. But in most common networks we have a DHCP server and clients get their configurations from this server so let’s set up one for our local access point network.


## step 4: Setting up dnsmasq

Run this command to install dnsmasq
```apt-get install hostapd dnsmasq```
dnsmasq will be used to assign ip addresses to devices connected to this access point

In the terminal type
```
sudo service hostapd stop
sudo service dnsmasq stop
```
set up config files in terminal type:

 ```gedit /etc/dnsmasq.conf```

 add those lines to the config file:

```
# Bind to only one interface
bind-interfaces
# Choose interface for binding
interface=hotspot
# Specify range of IP addresses for DHCP leasses
dhcp-range=192.168.150.2,192.168.150.10
```
## step 5: Creating hotspot.sh script
Now create anywhere you want a file named it hotspot.sh (best way to save script on Desktop) Edit it with any text editor like this:

```
#!/bin/bash
# Start
# Configure IP address for WLAN
sudo ifconfig hotspot 192.168.150.1
# Start DHCP/DNS server
sudo service dnsmasq restart
# Enable routing
sudo sysctl net.ipv4.ip_forward=1
# Enable NAT
sudo iptables -t nat -A POSTROUTING -o wlp2s0 -j MASQUERADE
# Run access point daemon
sudo hostapd /etc/hostapd.conf
# Stop
# Disable NAT
sudo iptables -D POSTROUTING -t nat -o wlp2s0 -j MASQUERADE
# Disable routing
sudo sysctl net.ipv4.ip_forward=0
# Disable DHCP/DNS server
sudo service dnsmasq stop
sudo service hostapd stop
```

You will probably need to change wlp2s0 in this to eth0 or any other number which refers to your wired connection from where you're actually getting internet.

Now change the permissions of hotspot.sh to make it executable
```chmod +x hotspot.sh```

## step 6: Start the hostpot.sh

Now you can start your hotspot by starting script. Just run it...  For me it looks like this:
```
root@kali:~# cd /root/Desktop/
root@kali:~/Desktop# ./hotspot.sh
```
