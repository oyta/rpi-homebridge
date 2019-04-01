# rpi-homebridge
Setup of RPi Zero W with Homebridge to function with, among others, Noble.
## Intro
Homebridge is a great piece of software letting us add alot of stuff to Apple Homekit. It can be installed on different kind of hardware, also the cheap and small Raspberry Pi Zero W. The RPi Zero W has built in WiFi and Bluetooth.

An easy way to install everything is simply using the [Docker Homebridge by Oznu](). However, I'm not experienced with Docker and didn't manage making it work with Bluetooth. This description is how I setup my RPi Zero W with Homebridge.

## References
* [Homebridge by nafarina](https://github.com/nfarina/homebridge)
* [Docker Homebridge by Oznu]()
* [Node install]()
* [NPM Homebridge Plugins]()


## Step by step
Short step by step.
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
* Install needed packages. I use `vim` but you can use whatever is your favorite editor.
   ```
   sudo apt-get install vim 
* If having more than one Raspberry Pi on the network, I find it useful to change the hostnames:
   ```
   sudo apt-get upa
