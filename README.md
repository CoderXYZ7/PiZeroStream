# PiZeroStream

Here is a complete step-by-step guide to setting up a Raspberry Pi Zero W as a private offline streaming server that you can access via Wi-Fi and manage via SSH.

### **Step-by-Step Guide to Setting Up a Raspberry Pi Zero W as a Private Offline Streaming Server**

#### **1. Hardware Requirements:**
- Raspberry Pi Zero W (with Wi-Fi)
- MicroSD Card (16GB or larger)
- External Storage Device (USB flash drive or hard drive)
- Power Supply (5V)
- USB OTG Cable (to connect external storage)
- MicroSD Card Reader (for initial setup)

#### **2. Software Requirements:**
- Raspberry Pi OS Lite
- Jellyfin Media Server
- Hostapd and Dnsmasq (for Wi-Fi access point)
- SSH Client (e.g., PuTTY for Windows, Terminal for macOS/Linux)

#### **3. Preparing the SD Card:**

**Download Raspberry Pi OS Lite:**
1. Visit the [Raspberry Pi website](https://www.raspberrypi.org/software/operating-systems/).
2. Download Raspberry Pi OS Lite.

**Flash the OS to the SD Card:**
1. Use the [Raspberry Pi Imager](https://www.raspberrypi.org/software/) or [Balena Etcher](https://www.balena.io/etcher/) to flash the OS onto the SD card.

**Enable SSH and Configure Wi-Fi:**
1. After flashing, create an empty file named `ssh` (no extension) in the `boot` partition of the SD card to enable SSH.
2. Create a file named `wpa_supplicant.conf` in the `boot` partition with the following content to configure Wi-Fi (replace `Your_SSID` and `Your_PASSWORD` with your Wi-Fi credentials):
   ```plaintext
   country=US
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   update_config=1

   network={
       ssid="Your_SSID"
       psk="Your_PASSWORD"
       key_mgmt=WPA-PSK
   }
   ```

#### **4. Initial Raspberry Pi Configuration:**

**Boot the Raspberry Pi:**
1. Insert the SD card into the Raspberry Pi and power it on.

**Connect via SSH:**
1. Use an SSH client like PuTTY (Windows) or Terminal (macOS/Linux) to connect to the Pi:
   ```bash
   ssh pi@raspberrypi.local
   ```
   Default username: `pi`
   Default password: `raspberry`

**Update the System:**
1. Update and upgrade the system:
   ```bash
   sudo apt-get update
   sudo apt-get upgrade -y
   ```

#### **5. Set Up the Wi-Fi Access Point:**

**Install Hostapd and Dnsmasq:**
1. Install the necessary packages:
   ```bash
   sudo apt-get install hostapd dnsmasq -y
   ```

**Configure Hostapd:**
1. Edit the Hostapd configuration file:
   ```bash
   sudo nano /etc/hostapd/hostapd.conf
   ```
   Add the following content:
   ```plaintext
   interface=wlan0
   driver=nl80211
   ssid=YourNetworkName
   hw_mode=g
   channel=6
   wmm_enabled=0
   macaddr_acl=0
   auth_algs=1
   ignore_broadcast_ssid=0
   wpa=2
   wpa_passphrase=YourPassword
   wpa_key_mgmt=WPA-PSK
   wpa_pairwise=TKIP
   rsn_pairwise=CCMP
   ```

2. Edit the default Hostapd file to point to the configuration file:
   ```bash
   sudo nano /etc/default/hostapd
   ```
   Set:
   ```plaintext
   DAEMON_CONF="/etc/hostapd/hostapd.conf"
   ```

**Configure Dnsmasq:**
1. Backup the original configuration and create a new one:
   ```bash
   sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
   sudo nano /etc/dnsmasq.conf
   ```
   Add the following:
   ```plaintext
   interface=wlan0
   dhcp-range=192.168.50.2,192.168.50.20,255.255.255.0,24h
   ```

**Configure Network Interfaces:**
1. Edit the DHCP configuration:
   ```bash
   sudo nano /etc/dhcpcd.conf
   ```
   Add:
   ```plaintext
   interface wlan0
   static ip_address=192.168.50.1/24
   nohook wpa_supplicant
   ```

2. Ensure the network interfaces are correctly configured:
   ```bash
   sudo nano /etc/network/interfaces
   ```
   Add or modify:
   ```plaintext
   allow-hotplug wlan0
   iface wlan0 inet static
   address 192.168.50.1
   netmask 255.255.255.0
   ```

**Start and Enable Services:**
1. Start and enable Hostapd and Dnsmasq:
   ```bash
   sudo systemctl start hostapd
   sudo systemctl start dnsmasq
   sudo systemctl enable hostapd
   sudo systemctl enable dnsmasq
   ```

#### **6. Install and Configure Jellyfin:**

**Install Jellyfin:**
1. Install Jellyfin:
   ```bash
   sudo apt install apt-transport-https -y
   sudo apt update
   sudo apt install jellyfin -y
   ```

**Configure Jellyfin:**
1. Access Jellyfin’s web interface at `http://192.168.50.1:8096` from a browser connected to the Pi's Wi-Fi network.
2. Follow the setup wizard to configure your media library, pointing it to the folder on your external storage where you have your media files.

#### **7. Accessing the Pi and Streaming Content:**

**Connect to the Pi’s Wi-Fi Network:**
1. Use your device (laptop, smartphone, etc.) to connect to the Pi’s Wi-Fi network using the SSID and password you configured.

**SSH into the Pi:**
1. Use an SSH client to connect to the Pi using its static IP address:
   ```bash
   ssh pi@192.168.50.1
   ```

**Access Jellyfin Interface:**
1. Open a browser and navigate to `http://192.168.50.1:8096`.

**Stream Movies:**
1. Use the Jellyfin interface to browse and stream your movies.

By following these steps, you will have a Raspberry Pi Zero W set up as a private offline streaming server, accessible via its own Wi-Fi network and manageable via SSH.
