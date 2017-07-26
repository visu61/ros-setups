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
Run `sudo apt-get install ntp`

Run `sudo nano /etc/ntp.conf` 
- rename the server in the file from  "0.debian.pool.ntp.org" to "server 0.sg.pool.ntp.org"

save the file and then

Run `sudo dpkg-reconfigure tzdata`

## Tools
```
Run `sudo apt-get -y install git`
Run `sudo apt-get -y install sudo less`
```
## Add host to the intel edison
`sudo nano /etc/hosts` and add the following line below in the hosts file.

localhost 127.0.0.1 edison

## Add User
`adduser px4`
`passwd px4` (set the password to px4)`usermod -aG sudo px4`
`usermod -aG dialout px4`

Login as px4 to continue.

# ROS/MAVROS Installation

As ROS packages for the Edison/Debian don't exist we will have to build it from source. This process will take about 1.5 hours but most of it is just waiting for it to build.
A script has been writen to automate the building and installation of ROS. Current testing has been copy-pasting line by line to the console. Willing testers are encouraged to try out running the script:

Run `sudo git clone -b jessie https://github.com/tcheehow/ros-setups`
"Clone forked branch (Jessie)  "

Run `sudo cd ros-setups/intel-edison/`

Run`./install_ros.sh`
if the above method produces error then follow this step otherwise just go ahead to install libraries like mraa

If all went well you should have a ROS installtion. Hook your Edison up to the Pixhawk and run a test. See this page for instructions: https://pixhawk.org/peripherals/onboard_computers/intel_edison


## Alternate method for installing ros 

```
#!/bin/bash

# The following installation is based on: http://wiki.ros.org/wiki/edison
# and http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Indigo%20on%20Raspberry%20Pi

```
check if you are root

```

if [ `whoami` == "root" ]; then
  echo "Do not run this as root!"
  exit 1
fi
```
" Update sources.list "
 
Run `sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu jessie main" > /etc/apt/sources.list.d/ros-latest.list' `

" Get ROS and Raspian keys "

Run ` sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116 `

Run `wget https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -O - | sudo apt-key add - `

Run `wget http://archive.raspbian.org/raspbian.public.key -O - | sudo apt-key add - `

" Update the OS "

Run `sudo apt-get -y update`

Run `sudo apt-get -y upgrade`

"Install required OS packages "

Run `sudo apt-get -y install pkg-config`

Run `sudo apt-get -y install python-setuptools python-pip python-yaml python-argparse python-distribute python-docutils python-dateutil  python-six `

'Install required ROS packages'

Run `sudo pip install rosdep rosinstall_generator wstool rosinstall`
 
 "Fix some permission issues"
 Run `sudo cd ~ `
 
 Run `sudo chown -R px4 .`
 
 Run `sudo rosdep init`
 
 Run `rosdep update `


Run `sudo apt-get -y update`

"Install console bridge "

Run `cd ~/ros_catkin_ws/external_src`
Run `sudo apt-get -y build-dep console-bridge`
Run `apt-get -y source -b console-bridge`
Run `sudo dpkg -i libconsole-bridge0.2*.deb libconsole-bridge-dev*.deb`

"Install liblz4-dev "

Run `sudo apt-get -y install liblz4-dev`

"rosdep install - Errors at the end are normal "
rUN `cd ~/ros_catkin_ws`

" Python errors after the following command are normal."
Run `rosdep install --from-paths src --ignore-src --rosdistro indigo -y -r --os=debian:jessie`

echo “******************************************************************”
echo “About to start some heavy building. Go have a looong coffee break.”
echo “******************************************************************”

## FIX THE BUILDING THE 69 PACAKGE 

Run `cd /home/px4/ros_catkin_ws/build_isolated/mavros && /opt/ros/indigo/env.sh make -j1 -l2 `
"the parallel process has the issue"

Run `cd ~/ros_catkin_ws/build_isolated/`

Run `sudo chown -R px4 .`

Run `cd ~/ros_catkin_ws/devel_isolated/`

Run `sudo chown -R px4 `

Run `cd ~/ros_catkin_ws`

Run `cd /home/px4/ros_catkin_ws/build_isolated/mavros && /opt/ros/indigo/env.sh make -j1 -l2`
"#NEED TO FIX THE CHMOD BUILD IN BUILOD ISOLATED"

"#Then rebuild the ros"

## Building ROS 
Run `sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/indigo`

```
if some error is captured then executed the line otherwise its fine go ahead
#cd ~/ros_catkin_ws/build_isolated/
#sudo chown -R px4 .

#sudo ln -sf /home/ros /opt/
```
"Updating .profile and .bashrc "

Run `echo "source /opt/ros/indigo/setup.bash" >> ~/.profile`

Run `source ~/.profile`

Run `echo "source ~/ros_catkin_ws/devel_isolated/setup.bash" >> ~/.bashrc`

Run `source ~/.bashrc`

Run `cd ~/ros_catkin_ws`


echo "*** FINISHED building the ros! ***"


## Install Edison swig installation
```
Method 1 ) easy 

```

Run `sudo apt-get update`

Run `sudo apt-cache search pcre`

"#please ensure the following packages are returned in the console as output" 
"#libpcre3-dev - Perl 5 Compatible Regular Expression Library - development files"

Run `sudo apt-get install libpcre3-dev`

Run `sudo apt-get install git`

Run `sudo apt-get install cmake`

Run `sudo apt-get install python-dev`

Run `sudo apt-get install swig`

 
-- If the swig installation fails then build from method 2

```
Method 2) build from scratch
*Before building and installing mraa library be sure to install the following swig 
*To build swig  by method 2 be sure to run the given command before going to method 2


```
" please run only for method 2"

Run `sudo apt-get install bison automake autoconf build-essential g++.`
-It might ask to fix broken packages

Run `sudo apt-get -f install`

Run `git clone https://github.com/swig/swig.git`

Run `cd swig`

Run `chmod +x ./autogen.sh`

Run `sudo ./autogen.sh`

Run `sudo chown -R px4 .`

Run `sudo ./configure --prefix=/some/directory`

*please make sure that some directory is created after this command
Run `sudo make`

Run `sudo make install`


-If everything went fine build would be successful for the swig 

## Install Edison mraa libraries
libmraa is not in apt so we’ll have to compile it from source. Don’t worry, it’s easy:

```
git clone https://github.com/intel-iot-devkit/mraa.git
mkdir mraa/build && cd $_
cmake .. -DBUILDSWIGNODE=OFF
make
sudo make install
cd
```
Important: Make sure you run the final command, “make install” with root or “sudo.”

"That DBUILDSWIGNODE flag turns off node.js support, which isn’t available in the version of swig in apt. If you need node.js, you can compile a newer version of swig from source (3.01+)."

## Update Shared Library Cache
"To use the library in C or C++ programs, we need to add it to our shared library cache. With root (or using “sudo”), open up the ld.so.conf file:"

Run `sudo nano /etc/ld.so.conf`
Scroll down to the bottom of the file and add:

`/usr/local/lib/i386-linux-gnu/`

Your ld.so.conf file should look like this:
![alt text](https://github.com/AMAN3003/UAV/blob/master/uav1.png)

Save and exit (‘Crtl-x’ and ‘y’ with nano). Type the command (using root or “sudo”):

run `sudo ldconfig`

You can check to make sure that the cache was updated by typing the command:
Run `sudo ldconfig -p | grep mraa`
![alt text](https://github.com/AMAN3003/UAV/blob/master/uav2.png)

## Export Library Path for Python

If you plan to use Python with mraa, you will need to export its location to the Python path. Enter the command:

Run `export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))`

Note that this command lets us use the mraa module for this terminal session only. If we restart the Edison, we will have to retype the command.

Optional: You can modify the .bashrc file to run the commands automatically every time the Edison starts. Open the .bashrc file with your favorite editor. For example:

Run `nano ~/.bashrc`

Scroll all the way down to the bottom of the file, and add the command from above in a new line.

`export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))`
The bottom of your .bashrc file should look like the screenshot below.
![alt text](https://github.com/AMAN3003/UAV/blob/master/uav3.png)


Save and exit (‘Crtl-x’ and ‘y’ with nano).


## Freeing up Space on the Root Partition
You will need more space on the root partition. Run the following commands:

Run `mv /var/cache /home/`

Run `ln -s /home/cache /var/cache`

Run `mv /usr/share /home/`

Run `ln -s /home/share /var/share`




 Run `mkdir ~/ros_catkin_ws `
 
 Run `cd ~/ros_catkin_ws `

 Run `Chmod 777 /ros_catkin_ws `

## Ros Install 

"This will install only mavros and not mavros-extras (no image support which the Edison can’t really handle well anyway)."
Run `rosinstall_generator ros_comm mavros --rosdistro indigo --deps --wet-only --exclude roslisp --tar > indigo-ros_comm-wet.rosinstall`

" wstool installation "

`sudo wstool init src -j3 indigo-ros_comm-wet.rosinstall `

if there is wstool failure then run the following command

Run ` sudo wstool update -t src -j3 `

```
while [ $? != 0 ]; do
  echo "*** wstool - download failures, retrying ***"
  sudo wstool update -t src -j3
done
```

Run `sudo cd ~/ros_catkin_ws `

" Install cmake and update sources.list "

Run `sudo mkdir ~/ros_catkin_ws/external_src `

Run `sudo apt-get -y install checkinstall cmake `

Run `sudo sh -c 'echo "deb-src http://mirrordirector.raspbian.org/raspbian/ testing main contrib non-free rpi" >> /etc/apt/sources.list' `

``` 
run the below code only when there is error in previous one 
#sudo sh -c 'echo "deb http://http.debian.net/debian jessie-backports main" >> /etc/apt/sources.list'

```



# Python Flight App

Once you have a functional ROS setup you can *very carefully* perform an offboard flight using the setpoint_demo.py script. This script assumes that you have already successfully run `roslaunch mavros px4.launch`.

WARNING WARNING: Make sure you can take control via RC transmitter at any time, things can go quite wrong. Also be aware that there isn't any velocity control currently and the multirotor will use max velocity at times. Read the code before you fly so you know what to expect.

Launch the demo by running:

`./setpoint_demo.py`

and once it is running activate offboard control on your RC transmitter.

## Freeing up Space on the Root Partition

Once again we will remove unneeded files from the root partition. You can delete the files in root's home directory (that's /root) or move them to the home partition.
