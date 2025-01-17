# kiwix-bootable-hotspot-howto

Quick how-to for creating a bootable hotspot with a Raspberry Pi (or similar SBC) for serving offline Wikipedia and other information.

This has been tested on a RaspberryPi 4 with Raspbian OS and an OrangePi Zero 2W with Armbian OS.  The default OrangePi OS is preconfigured to only use Huawei package repositories which seemed sketchy to me.  Armbian seems to work just fine but there are extra steps (detailed below) compared to Raspbian.  I used v25.2 rolling based on Debian Bookworm.

**SD Card Size Note**:  Obviously you need enough space on your SD card to store all the data you want to serve.  The `wikipedia_en_all_maxi_2024-01.zim` I used is approximately 109GB alone so a 256GB card is suggested.

## Armbian Extra Steps : 

Armbian does not seem to have `nmcli` or `dnsmasq` installed by default which prevents your access point from providing IP addresses via DHCP.  If using Armbian, perform the following installation and configuration steps : 
```bash
sudo apt install network-manager
sudo apt install dnsmasq
# Stop / Disable dnsmasq to avoid conflicts with the instance started by Network Manager
sudo systemctl disable dnsmasq
sudo systemctl stop dnsmasq
```

Then, add this configuration item to `/etc/NetworkManager/NetworkManager.conf` :
# kiwix-bootable-hotspot-howto

Quick how-to for creating a bootable hotspot with a Raspberry Pi (or similar SBC) for serving offline Wikipedia and other information.

This has been tested on a RaspberryPi 4 with Raspbian OS and an OrangePi Zero 2W with Armbian OS.  The default OrangePi OS is preconfigured to only use Huawei package repositories which seemed sketchy to me.  Armbian seems to work just fine but there are extra steps (detailed below) compared to Raspbian.  I used v25.2 rolling based on Debian Bookworm.

**SD Card Size Note**:  Obviously you need enough space on your SD card to store all the data you want to serve.  The `wikipedia_en_all_maxi_2024-01.zim` I used is approximately 109GB alone so a 256GB card is suggested.

## Armbian Extra Steps : 

Armbian does not seem to have `nmcli` or `dnsmasq` installed by default which prevents your access point from providing IP addresses via DHCP.  If using Armbian, perform the following installation and configuration steps : 
```bash
sudo apt install network-manager
sudo apt install dnsmasq
# Stop / Disable dnsmasq to avoid conflicts with the instance started by Network Manager
sudo systemctl disable dnsmasq
sudo systemctl stop dnsmasq
```

Then, add this configuration item to `/etc/NetworkManager/NetworkManager.conf` :

```bash
[main]
dns=dnsmasq
```


## Instructions
1. Install RaspiOS Lite preconfigured with
   1. Username = whatever
   2. Password = whatever
   3. Hostname = kiwix.local (or whatever you prefer)
   4. Enable SSH (PW)
2. Boot and connect via SSH using the hostname, username, and password configured above.
3. If you didn't enable Avahi in the OS config, install and enable it now.  This allows you to access the device by hostname (`kiwix.local`) instead of IP address.
   1. `sudo apt install avahi-daemon`
   2. `sudo systemctl start avahi-daemon`
   3. `sudo systemctl enable avahi-daemon`
4. Install `kiwix-tools` - this includes `kiwix-serve` and `kiwix-manage`
   1. `sudo apt install kiwix-tools`
5. Create the directory for your Kiwix library.  I chose to put it in my home directory
   1. `mkdir ~/kiwix-library`
6. Transfer all desired ZIM files into this directory via SFTP
7. Create a `library.xml` config for Kiwix and add ZIM files to make them accessible by `kiwix-serve`.  I chose to put it in the same directory as my ZIM files
   1. `kiwix-manage ~/kiwix-library/library.xml add <zimFile>`
   2. EG: `kiwix-manage ~/kiwix-library/library.xml add wikipedia_en_top_mini_2023-10.zim`
8.   Add any other ZIM files using the `kiwix-manage` command above and specifying the file paths
9.   Create systemctl service config (see "Kiwix Service" section below) at `/etc/systemd/system/kiwix.service`
10.  Enable and start the kiwix service that we created in step #9
     1. `sudo systemctl enable kiwix`
     2. `sudo systemctl start kiwix`
11.  Configure WiFi in Access Point / HotSpot mode.  **!! THIS WILL TERMINATE YOUR SSH SESSION IF CONNECTED VIA WIFI !!**  Prepare accordingly or ensure that you have a different way to connect if problems arise (ethernet).  SSH is still accessible via WiFi but you must be connected to the newly created `kiwix.local` network which does not have internet access.
     1. `sudo nmcli connection add type wifi con-name "kiwix" autoconnect yes wifi.mode ap wifi.ssid "kiwix.local" ipv4.method shared ipv6.method disabled connection.autoconnect true`
12. Open the wifi settings on your phone/computer and connect to the newly available `kiwix.local` SSID.  WPA is disabled intentionally so users can easily connect without needing a key/password.
13. Open a browser and navigate to http://kiwix.local:80
14. To ensure all of your settings are correct and persistent, reboot and verify that :
    1.  AP/Hotspot is available
    2.  http://kiwix.local:80 is accessible
    3.  Library (ZIM) files are present in the web UI and you can successfully load, navigate, and search them.
15. All done.  Enjoy your offline information.



### Kiwix Service (`/etc/systemd/system/kiwix.service`)
```bash
[Unit]
Description=Kiwix Service
After=network.target

[Service]
Type=simple
User=root
Group=root
# Modify the following line with your desired port (if not 80) and the correct path to your library.xml.  The default will work if you used the same configurations/paths specified above.
ExecStart=/usr/bin/kiwix-serve --port=80 --library /home/kiwix/kiwix-library/library.xml
Restart=always
RestartSec=3
[Install]
WantedBy=multi-user.target
```

### Configuration Shell Script
If you prefer to have all steps performed for you via script, this will do it for you.  Take note that you still must transfer your ZIM files to the correct directory and create the `kiwix` service prior to execution.

Because this does system configurations that require root access, you should execute it with `sudo`.
```bash
apt install kiwix-tools
mkdir ~/kiwix-library
kiwix-manage ~/kiwix-library/library.xml add ~/kiwix-library/*.zim
systemctl enable kiwix
# This will create an access point/hot spot with SSID "kiwix.local".  I chose this so that the SSID is the same as the hostname so it's easier to understand and find the service on the network for users.
nmcli connection add type wifi con-name "kiwix" autoconnect yes wifi.mode ap wifi.ssid "kiwix.local" ipv4.method shared ipv6.method disabled connection.autoconnect true
```