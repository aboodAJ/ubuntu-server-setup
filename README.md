# Ubuntu Server Setup

Commands and configuration files from my video on installing Ubuntu Server on a laptop.

I used Ubuntu Server 24.04 LTS.

## 1. Download Ubuntu Server

Download the ISO from https://ubuntu.com/download/server.

## 2. Create a bootable USB

You'll need a USB drive (at least 4 GB). Everything on it will be erased.

On Ubuntu, open **Startup Disk Creator**. If it isn't installed:

```bash
sudo apt install usb-creator-gtk
```

Select the ISO, choose your USB drive, then click **Make Startup Disk**. Boot the laptop from the USB and follow the Ubuntu Server installer (all steps shown in the video).

## 3. Connect using SSH (optional)

On the server, get its IP address:

```bash
hostname -I
```

From another computer, connect with:

```bash
ssh username@192.168.1.50
```

Replace the username and IP with yours.

## 4. Keep the server running with the lid closed

On laptops, closing the lid can put the system to sleep. Edit:

```bash
sudo nano /etc/systemd/logind.conf
```

Add or uncomment these settings:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Save (`Ctrl+O`, Enter) and exit (`Ctrl+X`), then run:

```bash
sudo systemctl restart systemd-logind
```

Check that the server stays accessible after closing the lid.

## 5. Set a static IP (optional)

First, find your network interface and router IP:

```bash
ip -br a
ip route | grep default
```

For example, `wlp1s0` might be your Wi-Fi interface and `192.168.1.1` your router.

Check your existing Netplan configuration:

```bash
ls /etc/netplan/
sudo cat /etc/netplan/50-cloud-init.yaml
```

The filename may differ. If Wi-Fi is already configured there, leave its access point and password settings in place.

Create a separate config:

```bash
sudo nano /etc/netplan/99-static.yaml
```

Example for Wi-Fi:

```yaml
network:
  version: 2
  wifis:
    wlp1s0:
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
```

Change the interface, IP and gateway to match your network. Choose an unused IP outside your router's DHCP pool (or reserve it on the router). For Ethernet, use `ethernets:` instead of `wifis:`. YAML indentation uses spaces, not tabs.

Set permissions and check the merged configuration:

```bash
sudo chmod 600 /etc/netplan/99-static.yaml
sudo netplan get
```

Apply the changes:

```bash
sudo netplan try
```

Confirm if the network still works. Otherwise, Netplan will roll back after the confirmation timeout. If you're connected over SSH, reconnect using the new IP.
