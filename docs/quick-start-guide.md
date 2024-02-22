# Quick start with a Raspberry Pi

Here's your brand new shiny Rasberry Pi 4 Model B set up with the latest Rasbian, such as bookworm.

![Raspberry Pi 4B](Raspberry-Pi-4B.jpg " Raspberry-Pi-4B with a Samsung 500GB SSD")

## Install prerequisites
Execute the following.

```bash
sudo apt update
sudo apt install lua-socket
sudo apt install lua-filesystem
sudo apt install lua-sec
```

## openLuup Installation
This is very straight-forward: create a directory **cmh-ludl/** for the openLuup installation on your machine (in your home directory is a good place, or for compatibility with Vera use **/etc/cmh-ludl/** but be careful with permissions) **cd** to it, and retrieve the file **openLuup_install.lua** from the GitHub repository using:

```bash
wget https://github.com/akbooer/openLuup/raw/master/Utilities/openLuup_install.lua
```

Run the file using the command line:

```bash
lua5.1 openLuup_install.lua
```

When successful, the script produces console output like this:

```text
lua install.lua

openLuup_install 2016.06.08 @akbooer
getting latest openLuup version tar file from GitHub...
un-zipping download files...
getting dkjson.lua...
creating required files and folders
initialising...
downloading and installing AltUI...
Tue Nov 8 11:08:32 2016 device 2 ' openLuup' requesting reload
openLuup downloaded, installed, and running...
visit http://172.16.42.131:3480 to start using the system

```

and browsing the **reported URL** http://172.16.42.131:3480 will take you to the AltUI interface and show two devices: openLuup and AltUI. From here on, the interface can be used to configure the system.

Access the console at: http://172.16.42.131:3480/console

That's it!!

## Persist openLuup
Setup a systemd service to start up openLuup at boot time. Refer to the following.

[systemctl](/openluup?id=systemctl-with-etcsystemdsystemopenluupservice)

## json encoders / decoders
You can optimise the json decoding, if needed. Refer to [json parser](openluup-and-json.md).

## Not working?
Post your challenge to the [Smarthome Community Forum](https://smarthome.community/)!

## WinSCP
The latest RasPi OS releases don't set up a root login. But you can set up WinSCP so it has root priviledges:

In WinSCP:
Select Site and Edit then select Advanced and navigate to Environment > SCP/Shell

In the Shell pull down list: change from 'Default' to 'sudo -s' and save. Your account will log in using sudo privileges, letting you SCP files anywhere as root.

