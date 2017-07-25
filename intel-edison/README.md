# Intel Edison Setup
Instructions and scripts for the installation of CUSTOM ROS ON THE EDISON
# Before you Start

Before starting with the installation it's a good idea to boot the Edison straight out of the box to make sure it's working. This way we can make sure we have a functional board before proceeding and we won't be mistakenly blaming setup issues if something is wrong here.

Connect one USB cable to the cosole port and then start your temrminal app (see next section for more information on this). Once you are connected plug in the second USB cable for power and after 15 seconds you should see the system booting. If you want to login the user name is root (no password).

# Flash Debian

To build Debian Jessie carefully follow the instruction from ros-setups/README.md
If Windows is used, dfu-util is required. Download the latest version from this page:
http://dfu-util.sourceforge.net/releases/
Make sure you have the console USB cable in place and use it so you know when the installation has finished.
You MUST NOT remove power before it’s done or it could be bricked.
If you don't have a console connection make sure you wait 2 minutes at the end of the installation as it instructs.

During this time it is completing the installation which should n't be interrupted. 
If you don't get any update on your console after this message is displayed restart your console terminal connection.

Connect to the console with 115000 8N1, for example:

`screen /dev/USB0 115200 8N1`

and login as username: px4 (password: px4) 


## Post Debian Install
```
After Debian has been installed you will end up with the following partitions:
Filesystem       Size  Used Avail Use% Mounted on
rootfs           1.4G  813M  503M  62% /
/dev/root        1.4G  813M  503M  62% /
devtmpfs         480M     0  480M   0% /dev
tmpfs             97M  292K   96M   1% /run
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            193M     0  193M   0% /run/shm
tmpfs            481M     0  481M   0% /tmp
/dev/mmcblk0p7    32M  5.3M   27M  17% /boot
/dev/mmcblk0p10  1.3G  2.0M  1.3G   1% /home
```

For some reasons, the /home partition is not mounted after the first boot. To fix this issue, add the follow to the bottom of /etc/fstab .

`sudo nano /etc/fstab` 
Add the below line in the file of fstab

/dev/disk/by-partlabel/home     /home       auto    defaults     1   1

after saving the file 

`sudo resize2fs /dev/mmcblk0p5`

If the above not worked try the below as rootfs is in partion 8
mmcblk0p* here * denote the partion no 

`sudo resize2fs /dev/mmcblk0p8`

## Wifi
copy the interface to home one 
Run `sudo cp /etc/network/interfaces /etc/network/interfaces.home`

Run wpa_passphrase your wifi-ssid your-wifi-password to generate pka Like 

Run `wpa_passphrase cs2@sutd CsTwo@SuTd`

`cd /etc/network`

Edit both "/etc/network/interfaces.home" and "/etc/network/interfaces.work"

Add the following line in interaface and interface.home file
```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo  
iface lo inet loopback

auto wlan0  
iface wlan0 inet dhcp  
    wpa-ssid ExampleWifi
    wpa-psk 81088ba3b4b387ea4d22a4ad369ffa42f4966d2f3d61f6c65cdc001460239dc4
```
- Change wpa-ssid to your ssid 
- Change wpa-pka to generated pka 
- Comment out `auto usb0` plus the three lines that follow it (interface definition)

-Uncomment `auto wlan0`
-Save 

-Run: `ifup wlan0`

Create two script one for homenet 
Run `sudo nano ~/homenet.sh` and 
other for the worknet
Run `sudo nano ~/worknet.sh` 
Make script executable
`sudo chmod +x ~/homenet.sh`
`sudo chmod +x ~/worknet.sh`

Fill the following in the script

<pre><code>
#!/bin/bash
# Change Network to Home Network
cp /etc/network/interfaces.home /etc/network/interfaces
echo "Disable wlan0"
ifdown wlan0
echo "Re-enable wlan0"
ifup wlan0
</code></pre>

If you want to use a static IP then your config will look something like this:
``
auto wlan0
iface wlan0 inet static
    # For WPA
    wpa-ssid <your-ssid>
    wpa-psk <your-ssid-psk>

    address 192.168.43.101
    netmask 255.255.255.0
```

For the remaining steps you may wish to login via ssh instead.


```
## updates
Add the following to the bottom of sources list (/etc/apt/sources.list)

Run`sudo nano /etc/apt/sources.list`

add the following line to the sources list for updating it from other server

```
deb http://ftp.sg.debian.org/debian jessie main contrib non-free
#deb-src http://http.debian.net/debian jessie main contrib non-free

deb http://ftp.sg.debian.org/debian jessie-updates main contrib non-free
#deb-src http://http.debian.net/debian jessie-updates main contrib non-free

deb http://security.debian.org/ jessie/updates main contrib non-free
#deb-src http://security.debian.org/ jessie/updates main contrib non-free

#deb http://ubilinux.org/edison wheezy main

deb http://ftp.sg.debian.org/debian jessie-backports main
```

execute the below line after saving the file 

Run `sudo apt-get -y update`

Run `sudo apt-get -f install`

Run `sudo apt-get -y upgrade`

## Locales

Run `sudo apt-get install locales`
Run `sudo dpkg-reconfigure locales`

then Select only "en_US.UTF8" followed by region "Asia" follwed by "Singapore" and finally select "None" as the default on the confirmation page that follows.
Run `Update-locale`

Update the /etc/default/locale file and ensure LANG=en_US.UTF-8 and it uncommented out. Add LC_ALL=C.
Run `sudo nano etc/default/locale` 
- uncomment the LANG=en_US.UTF-8
- add the line LC_ALL=C

reboot the system to make changes 

in case of any locale errors go through 
```
---Solution for locale error --
Note that if you receive warning messages about missing or wrong languages this is likely to be due to the locale being forwarded when using SSH. Either ignore them or complete this step via the serial console by commenting out the SendEnv LANG LC_* line in the local /etc/ssh/ssh_config file on your machine (not the Edison).
```

## Timezone
`sudo dpkg-reconfigure tzdata`

## Tools
```
apt-get -y install git
apt-get -y install sudo less
```

## Add User
`adduser px4`
`passwd px4` (set the password to px4)
`usermod -aG sudo px4`
`usermod -aG dialout px4`

Login as px4 to continue.

# ROS/MAVROS Installation

As ROS packages for the Edison/Ubilinux don't exist we will have to build it from source. This process will take about 1.5 hours but most of it is just waiting for it to build.

A script has been writen to automate the building and installation of ROS. Current testing has been copy-pasting line by line to the console. Willing testers are encouraged to try out running the script:

```
git clone https://github.com/UAVenture/ros-setups
cd ros-setups/intel-edison/
./install_ros.sh
```

If all went well you should have a ROS installtion. Hook your Edison up to the Pixhawk and run a test. See this page for instructions: https://pixhawk.org/peripherals/onboard_computers/intel_edison

# Python Flight App

Once you have a functional ROS setup you can *very carefully* perform an offboard flight using the setpoint_demo.py script. This script assumes that you have already successfully run `roslaunch mavros px4.launch`.

WARNING WARNING: Make sure you can take control via RC transmitter at any time, things can go quite wrong. Also be aware that there isn't any velocity control currently and the multirotor will use max velocity at times. Read the code before you fly so you know what to expect.

Launch the demo by running:

`./setpoint_demo.py`

and once it is running activate offboard control on your RC transmitter.

## Freeing up Space on the Root Partition

Once again we will remove unneeded files from the root partition. You can delete the files in root's home directory (that's /root) or move them to the home partition.
