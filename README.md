# RPi Zero W with Homebridge as a service
Setup of RPi Zero W with Homebridge to function with, among others, Noble.

## Intro
Homebridge is a great piece of software letting us add alot of stuff to Apple Homekit. It can be installed on different kind of hardware, also the cheap and small Raspberry Pi Zero W. The RPi Zero W has built in WiFi and Bluetooth.

An easy way to install everything is simply using the [Docker Homebridge by Oznu](). However, I'm not experienced with Docker and didn't manage making it work with Bluetooth. This description is how I setup my RPi Zero W with Homebridge.

## References
* [Homebridge by nafarina](https://github.com/nfarina/homebridge)
* [Docker Homebridge by Oznu](https://github.com/oznu/docker-homebridge)
* [Node.js dists](https://nodejs.org/dist/latest/)
* [NPM Homebridge Plugins](https://www.npmjs.com/search?q=keywords:homebridge-plugin)
* [Noble](https://github.com/noble/noble#readme)

## Step by step
I've tried to include the necessary steps below. Please make sure you understand the commands you are running.

### Raspbian
* Download the latest Raspbian Lite distro for RPi. Currently it is the Strech version. [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/).
* Follow the installation instructions on [raspberrypi.org](https://www.raspberrypi.org/documentation/installation/installing-images/README.md). It is basically to downloading Etcher and use it to load the image on the micro SD-card.

### Headless RPi config
* Put the micro SD card with Raspbian in your SD card reader.
* Create two files on the root directory of the SD card
  * One file named "ssh". It must be empty. No file extension.
  * A second fil named "wpa_supplicant.conf". Add the following content, and replace the info with your own WiFi info.
   ```
   country=GB
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   update_config=1

   network={
       ssid="YOUR_WIFI_SSID"
       psk="YOUR_WIFI_PASSWORD"
   }
   ```
   Now, the SD card is ready for use. When you plug the power in your raspberry pi, it will automatically connect to your WiFi network with SSH enabled.


### Initial config
* Login in with SSH via Terminal for Mac or other similar tools.
   ```ssh pi@raspberrypi.local```
* Default password is "raspberry".
* Change the password with the command `passwd`
* Update everything
   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```
* You can do some inital config with `raspi-config`. Setting up the language packages among other.
* Install needed packages. I use `vim` but you can use whatever is your favorite editor.
   ```
   sudo apt-get install vim git 
   sudo apt-get install libavahi-compat-libdnssd-dev bluetooth bluez libbluetooth-dev libudev-dev
   sudo apt-get install libcap2-bin
   ```
* If having more than one Raspberry Pi on the network, I find it useful to change the hostnames:
   ```
   vim /etc/hostname
   vim /etc/hosts
   ```
* Rebooot with `sudo reboot`.

### Install Node.js
The Raspberry Pi Zero W has ARM6L architecture. Hence the following procedure is required to install Node.Js.
```
wget https://nodejs.org/dist/latest/node-v11.13.0-linux-armv6l.tar.gz
tar -xvf node-v11.13.0-linux-arm6l.tar.gz
cd node-v11.13.0-linux-arm6l
sudo cp -R * /usr/local/
```
Now `node`and `npm`should be in the path. Test it by getting the version:
```
pi@rpi-zw-01:~ $ node -v
v11.13.0
pi@rpi-zw-01:~ $ npm -v
6.7.0
```
Give Node sudo rights for running BLE Scan:
```
sudo setcap cap_net_raw+eip $(eval readlink -f `which node`)
```

### Install Homebridge
```
sudo npm install -g --unsafe-perm homebridge
```

### Making homebridge run as a service on the Rpi
Check out https://gist.github.com/johannrichard/0ad0de1feb6adb9eb61a/#file-homebridge by for the original description and files.
```
git clone https://gist.github.com/0ad0de1feb6adb9eb61a.git
  
sudo cp ./0ad0de1feb6adb9eb61a/homebridge /etc/default/homebridge
sudo cp ./0ad0de1feb6adb9eb61a/homebridge.service /etc/systemd/system

# User setup
sudo useradd -M --system homebridge
sudo groupadd homebridge_group
sudo usermod -a -G homebridge_group homebridge

# Folder
sudo mkdir /var/lib/homebridge

# Folder user priveleges
sudo chown -R homebridge /var/lib/homebridge

# Systemctl
sudo systemctl daemon-reload
sudo systemctl enable homebridge
sudo systemctl start homebridge
```

To making the homebridge user run without password open the sudoers file with:
```sudo visudo```

Then add a line with the following:
```homebridge ALL=(ALL) NOPASSWD: ALL```

### config.json
Homebridge needs a config.json file. In this setup it must be placed at `/var/lib/homebridge/config.json`.
See the sample config.json at [nfarina's Homebridge site](https://github.com/nfarina/homebridge).

### Homebridge-ui-x plugin
Oznu has made a UI for Homebridge where one can e.g. edit config.json and see the running status of Homebridge. You'll find more information about the install etc. at [npmjs.com](https://www.npmjs.com/package/homebridge-config-ui-x), but I have added some specifics for my setup below.
* Edit the service file with  `sudo vim /etc/default/homebridge.service`
* Add and `-I` argument to make Homebridge run in insecure mode. It should look like this:
   ```
   # Defaults / Configuration options for homebridge
   # The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)
   HOMEBRIDGE_OPTS=-U /var/lib/homebridge -I

   # If you uncomment the following line, homebridge will log more
   # You can display this via systemd's journalctl: journalctl -f -u homebridge
   # DEBUG=*
   ```
* Install the plugin with `sudo npm install -g --unsafe-perm homebridge-config-ui-x`.

