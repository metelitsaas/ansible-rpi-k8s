# ansible-rpi-k8s
Kubernetes installation on RaspberryPi by Ansible.

Master:
- RPI 4 2G

## 1. OS installation
Image writing to SD card
```
# Find out SD card device, /dev/disk2 in my case
diskutil list

# Unmount SD card
diskutil unmountdisk /dev/disk2

# Write image to SD card
sudo dd if=2020-12-02-raspios-buster-armhf-lite.img of=/dev/disk2 bs=2m
```

Wi-Fi configuration, set your own Network Name as ssid and Password as psk
```
cat <<EOF | sudo tee /Volumes/boot/wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=RU
network={
	ssid="network_name"
	psk="network_password"
	key_mgmt=WPA-PSK
}
EOF
```

Enable SSH at first boot
```
touch /Volumes/boot/ssh
```

## 2. OS configuration
After first boot find out RPI's IP in your network
```
arp -a
```
It is 192.168.0.18 in my case.

Connect to RPI by SSH, login/pass by default pi/raspberry
```
ssh pi@192.168.0.18
```

Enable SSH and add to autostart
```
sudo systemctl start ssh
sudo systemctl enable ssh
```

Set static IP
```
echo \
'interface wlan0
static ip_address=192.168.0.18
static routers=192.168.0.1
static domain_name_servers=8.8.8.8' | sudo tee -a /etc/dhcpcd.conf
```

Create new user
```
sudo adduser admin
sudo passwd admin
sudo usermod -G sudo admin
```

Enable SFTP server and SSH access only for admin user
```
echo "Subsystem sftp internal-sftp"  | sudo tee -a /etc/ssh/sshd_config
echo "AllowUsers admin"  | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart sshd
```

Set SSH-key for RPI
```
# Generate key
ssh-keygen -t rsa

# Copy key to PI
ssh-copy-id admin@192.168.0.18
```

## 3. Run playbook
```
ansible-playbook -i hosts site.yml --ask-become-pass
```