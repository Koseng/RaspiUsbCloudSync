# RaspiAsUSBStickWithCloudSync

## Raspberry Pi Zero W as USB stick

For using Linux USB "On-The-Go" (OTG)  with the Raspberry Pi Zero W, a different USB driver is necessary. The standard heavily patched and performance oriented driver does not support OTG mode. Instead the dwc2 driver needs to be used.

For OTG mode there are different [gadget drivers](http://www.linux-usb.org/gadget/). A Raspberry Pi Zero W can behave like an USB stick with the [g_mass_storage](https://www.kernel.org/doc/Documentation/usb/mass-storage.txt) driver. A more complex alternative is using libcomposite with configfs.

### Install Raspi OS Debian Bookworm
Use [Raspberry Pi Imager](ttps://www.raspberrypi.com/software/). 
https://www.raspberrypi.com/documentation/computers/getting-started.html

- Select operating system: Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32bit)
- Before writing, open settings with gear symbol button
    - Set Hostname: raspberrypi.local
    - Enable SSH with Password
    - Username: pi
    - Password: YOUR_PASSWORD
    - Setup Wifi

Instructions for Raspi OS Debian Bookworm

### Load the driver

> In `/boot/firmware/config.txt` comment out `otg_mode` and add `dtoverlay=dwc2`

```
sudo grep -qxF '#otg_mode=1' /boot/firmware/config.txt || sudo sed -i '/^otg_mode=1/s/^/#/' /boot/firmware/config.txt
sudo sh -c "grep -qxF 'dtoverlay=dwc2' /boot/firmware/config.txt || echo 'dtoverlay=dwc2' >> /boot/firmware/config.txt"
```

> In `/etc/modules` add `dwc2`:

```
sudo sh -c "grep -qxF 'dwc2' /etc/modules || echo 'dwc2' >> /etc/modules"
```

### Create the 3GB USB share

> First install packages for exfat file system. Then create and format usb image file piusb.bin. Finally prepare mounting.

 ```
sudo apt-get update
sudo apt-get -y install exfat-fuse
sudo apt-get -y install exfat-utils
sudo dd bs=1M if=/dev/zero of=/piusb.bin count=3072 # create file
sudo mkfs.exfat /piusb.bin # format
sudo mkdir /mnt/usb_share
```
> In `/etc/fstab` add `/piusb.bin /mnt/usb_share exfat noauto,nofail,users,umask=000 0 2`
```
sudo sh -c "grep -qF '/piusb.bin /mnt/usb_share' /etc/fstab || echo '/piusb.bin /mnt/usb_share exfat noauto,nofail,users,umask=000 0 2' >> /etc/fstab"
```

### Activate USB mass storage
Create system service to activate USB mass storage at the very end of the boot process. Therefore setting the type to idle is important. 

Copy `usb-storage.service` to `/etc/systemd/system/` then
```
sudo mv /home/pi/usb-storage.service /etc/systemd/system/
sudo chmod 644 /etc/systemd/system/usb-storage.service
sudo systemctl enable usb-storage.service
```

### Transfer to Linux server via SCP
- Create the destination directory on the server e.g. /home/myuser/transfer
- Install expect `sudo apt-get -y install expect`
- Change password in scp-copy.exp

### Activate python scripts

```
sudo apt-get -y install python3-dropbox
```

### Activate copy and upload service
```
sudo chmod 644 /etc/systemd/system/usb-copy.service
sudo systemctl enable usb-copy.service
sudo chmod 644 /etc/systemd/system/usb-upload.service
sudo systemctl enable usb-upload.service
```

 


