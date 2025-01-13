# Clustering Setup

- 3x RPI 4b (4gb)
- DietPi
- 3x Offical Raspberry Pi 4 USB-C Power Supply
- 3x USB-SATA (2x EasyULT Adaptateur USB 3.0 vers SATA III, 1 Unkown Model)
- 3x SATA SSD (2x Origin TLC830, 1x SD8SB8U-256G-100)
- 1x TP-Link TL-SG105S
- GeeekPi Raspberry Pi 4 Cluster Case
- 1x MicroSD Card (8GB)

[Reference](https://rpi4cluster.com/)

---

## DietPI Setup

1. Flash the Raspberry Pi 4 EEPROM boot recovery to the SD card to enable boot from USB
   1. Download Raspberry Pi Imager
   2. "Choose OS"
   3. "Misc utility images"
   4. "Bootloader"
   5. "USB Boot"
   6. Insert into RPI, power on and wait a minute or two until the green light next to the power starts blinking periodically
   7. Rinse and repeat for all Pis
2. Flash DietPI to SSD 1. Flash DietPI to SSD via Etcher 2. Once flashed, edit the `dietpi.txt` config found on the newly created partition

   1. Use the following config, **MAKING SURE TO UPDATE LOCALE, KB LAYOUT, IP SETTINGS AND HOSTNAME**

   ```
   AUTO_SETUP_ACCEPT_LICENSE=1
   AUTO_SETUP_LOCALE=C.UTF-8
   AUTO_SETUP_KEYBOARD_LAYOUT=us
   AUTO_SETUP_TIMEZONE=Europe/Bratislava
   AUTO_SETUP_NET_ETHERNET_ENABLED=1
   AUTO_SETUP_NET_WIFI_ENABLED=0
   AUTO_SETUP_NET_ETH_FORCE_SPEED=0
   AUTO_SETUP_NET_WIFI_COUNTRY_CODE=SK

    AUTO_SETUP_NET_USESTATIC=1
    AUTO_SETUP_NET_STATIC_IP=192.168.0.18
    AUTO_SETUP_NET_STATIC_MASK=255.255.255.0
    AUTO_SETUP_NET_STATIC_GATEWAY=192.168.0.1
    AUTO_SETUP_NET_STATIC_DNS=1.1.1.1 8.8.8.8

    AUTO_SETUP_DHCP_TO_STATIC=0

    AUTO_SETUP_NET_HOSTNAME=cube08

    AUTO_SETUP_BOOT_WAIT_FOR_NETWORK=1
    AUTO_SETUP_SWAPFILE_SIZE=1
    AUTO_SETUP_SWAPFILE_LOCATION=/var/swap
    AUTO_SETUP_HEADLESS=1
    AUTO_UNMASK_LOGIND=0
    AUTO_SETUP_CUSTOM_SCRIPT_EXEC=0
    AUTO_SETUP_BACKUP_RESTORE=0
    AUTO_SETUP_SSH_SERVER_INDEX=-2
    AUTO_SETUP_LOGGING_INDEX=-1
    AUTO_SETUP_RAMLOG_MAXSIZE=200

    AUTO_SETUP_WEB_SERVER_INDEX=0
    AUTO_SETUP_DESKTOP_INDEX=0
    AUTO_SETUP_BROWSER_INDEX=0
    AUTO_SETUP_AUTOSTART_TARGET_INDEX=7
    AUTO_SETUP_AUTOSTART_LOGIN_USER=root
    AUTO_SETUP_GLOBAL_PASSWORD=dietpi
    AUTO_SETUP_AUTOMATED=1
    SURVEY_OPTED_IN=0

    #OpenSSH Client
    AUTO_SETUP_INSTALL_SOFTWARE_ID=0
    #Samba Client
    AUTO_SETUP_INSTALL_SOFTWARE_ID=1
    #vim
    AUTO_SETUP_INSTALL_SOFTWARE_ID=20
    #RPi.GPIO
    AUTO_SETUP_INSTALL_SOFTWARE_ID=69
    #OpenSSH Server
    AUTO_SETUP_INSTALL_SOFTWARE_ID=105
    #Python 3 pip
    AUTO_SETUP_INSTALL_SOFTWARE_ID=130

    CONFIG_CPU_GOVERNOR=schedutil
    CONFIG_CPU_ONDEMAND_SAMPLE_RATE=25000
    CONFIG_CPU_ONDEMAND_SAMPLE_DOWNFACTOR=40
    CONFIG_CPU_USAGE_THROTTLE_UP=50

    CONFIG_CPU_MAX_FREQ=Disabled
    CONFIG_CPU_MIN_FREQ=Disabled

    CONFIG_CPU_DISABLE_TURBO=0

    CONFIG_PROXY_ADDRESS=MyProxyServer.com
    CONFIG_PROXY_PORT=8080
    CONFIG_PROXY_USERNAME=
    CONFIG_PROXY_PASSWORD=

    CONFIG_G_CHECK_URL_TIMEOUT=10
    CONFIG_G_CHECK_URL_ATTEMPTS=5
    CONFIG_CHECK_CONNECTION_IP=8.8.8.8
    CONFIG_CHECK_DNS_DOMAIN=google.com

    CONFIG_CHECK_DIETPI_UPDATES=1
    CONFIG_CHECK_APT_UPDATES=1
    CONFIG_NTP_MODE=2
    CONFIG_SERIAL_CONSOLE_ENABLE=0
    CONFIG_SOUNDCARD=none
    CONFIG_LCDPANEL=none
    CONFIG_ENABLE_IPV6=0

    CONFIG_APT_RASPBIAN_MIRROR=http://raspbian.raspberrypi.org/raspbian/
    CONFIG_APT_DEBIAN_MIRROR=https://deb.debian.org/debian/
    CONFIG_NTP_MIRROR=debian.pool.ntp.org

    #----------------------------------------------------------------------------------

    ##### DietPi-Software settings

    #----------------------------------------------------------------------------------
    SOFTWARE_DISABLE_SSH_PASSWORD_LOGINS=0

    #----------------------------------------------------------------------------------

    ##### Dev settings

    #----------------------------------------------------------------------------------
    DEV_GITBRANCH=master
    DEV_GITOWNER=MichaIng

    #----------------------------------------------------------------------------------

    ##### Settings, automatically added by dietpi-update

    #----------------------------------------------------------------------------------
   ```

   2. Edit the `cmdline.txt` file also found on the partition, append `group_enable=cpuset cgroup_enable=memory cgroup_memory=1` to the end of the line
   3. Rinse and repeat for each pi
   4. Once you've booted a pi, after a minute or two you should be able to ssh on (if you opted for DHCP look on your router to find the IP address) and monitor install progress, default creds are **root** and **dietpi**

### Touble shooting

After installing DietPI I ran into an issue where my pi's would start refusing SSH connections, or stop responding to pings. Very weird behaviour which I first thought was down to dodgy pis or powersupplys.

After a bit of web crawling I ran into https://forums.raspberrypi.com/viewtopic.php?f=28&t=245931

Which relates to poor preformance on USB-SATA USB3.0.

Incase the post gets taken down do the following to identify if this is an issue effecting your pis.

> I was running my setups completely headless so this way a bit of a pain, if you are able to connect a display I highly would recommend doing that!

1. Run `dmesg` and check the output for entries such as

```
[ 501.594683] usbcore: registered new interface driver usb-storage
[ 501.599729] scsi host6: uas
[ 501.599800] usbcore: registered new interface driver uas
...
[ 573.203294] sd 6:0:0:0: [sda] tag#29 uas_eh_abort_handler 0 uas-tag 9 inflight: CMD OUT
[ 573.203302] sd 6:0:0:0: [sda] tag#29 CDB: Write(10) 2a 00 00 4f a0 00 00 04 00 00
[ 573.205063] sd 6:0:0:0: [sda] tag#28 uas_eh_abort_handler 0 uas-tag 10 inflight: CMD OUT
[ 573.205070] sd 6:0:0:0: [sda] tag#28 CDB: Write(10) 2a 00 00 4f a4 00 00 04 00 00
[ 573.208537] sd 6:0:0:0: [sda] tag#27 uas_eh_abort_handler 0 uas-tag 6 inflight: CMD OUT
...
[ 573.269992] scsi host6: uas_eh_device_reset_handler start
[ 573.393710] usb 2-4: reset SuperSpeed Gen 1 USB device number 2 using xhci_hcd
[ 573.414256] scsi host6: uas_eh_device_reset_handler success
```

2. If you can confirm your getting messages simular to above, you will want to do the following
3. Identify the VID and PID of your USB SSD, run `dmesg | grep "New USB device found"` and identify your USB SSD via the device number, for example my was plugged into port 2 so I looked for _"USB device number 2"_
   - You can also unplug your USB-SATA and re-plug it in if your not booting from it, then look at the latest entry for your VID and PID
   - Your VID and PID entry will look like `idVendor=2109, idProduct=0715`
4. Add `usb-storage.quirks=aaaa:bbbb:u` to the start of the line in `/boot/cmdline.txt`
   - Where "aaaa" is your `idVendor` and "bbbb" is your `idProduct`
   - If you are doing this headless it will be a pain, as a side effect of this issue causes SSH connections to reset etc, so your have to keep trying to connect until you get in, once in quickly run these commands
   - If you can atleast get the VID and PID you can then plug your SSD into PC/Laptop and edit the `cmdline.txt` from the mounted drive
5. Once done reboot
6. Verify this worked by running `dmesg | grep usb-storage`
   - You should see a line such as `Quirks match for vid 2109 pid 0715: 800000`

Doing the above should resolve the issues, it atleast resolved mine!
