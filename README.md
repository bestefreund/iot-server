# IoT-Server: Raspberry Pi 3 with Docker (& an USB-Stick for data)

## Descritption

This Repo is intended for testing & tools for related to IoT-devices.
You can choose one of the platforms or all of them, but be aware of hardware & resource limitations of your system.
In every folder you'll find a docker-compose file representing an independent infratructure.

## Folder structure

```
tree -I ".git*" -a

.
├── docker_cleanup
│   ├── daemon.json
│   ├── docker-cleanup.service
│   ├── docker-cleanup.timer
│   └── docker-compose@.service
├── homeassistant
│   ├── configurator-config
│   │   └── settings.conf
│   ├── docker-compose.yml
│   ├── .env
│   ├── hass-config
│   │   └── configuration.yaml
│   ├── mosquitto-config
│   │   ├── mosquitto.conf
│   │   └── mosquitto.passwd
│   └── README.md
├── openhab
│   ├── docker-compose.yml
│   ├── .env
│   ├── mosquitto-config
│   │   ├── mosquitto.conf
│   │   └── mosquitto.passwd
│   └── README.md
├── portainer
│   ├── docker-compose.yml
│   └── README.md
└── README.md
```

## Referneces

Raspberry Docker:
 * HowTo:  https://withblue.ink/2020/06/24/docker-and-docker-compose-on-raspberry-pi-os.html

RasperryOS 64-Bit:
 * Download: https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-11-08/

OpenHab:
 * Docker: https://www.laub-home.de/wiki/OpenHAB_3_Docker_Installation
 * Tasmota: https://community.openhab.org/t/control-tasmota-flashed-devices-via-mqtt/104099

Home Assistant:
 * Docker: https://www.dahlen.org/2018/03/23/home-assistant-mit-docker-auf-raspberry-pi-betreiben/
 * Docker-Compose: https://iotechonline.com/home-assistant-install-with-docker-compose/?cn-reloaded=1
 * Home Assistant using venv in Docker: https://github.com/tribut/homeassistant-docker-venv
 * Tasmota: https://www.home-assistant.io/integrations/tasmota/

FHEM:
 * Docker: https://blog.krannich.de/fhem-auf-rpi-mit-docker-und-hypriot/
 * Tasmota: https://gettoweb.de/haus/tasmota-device-in-fhem-einbinden/

## Config

After RaspberryOS installation login to the Raspberry & configure ssh
```
ssh-keygen

sudo systemctl enable ssh
sudo systemctl start ssh

exit
```

Login via ssh & start your journey
```
sudo raspi-config
```

### Basic

```
# Enable 64-bit
sudo bash -c 'echo "arm_64bit=1" >> /boot/config.txt'
sudo reboot

# SSH-Key auth
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Configure System
sudo apt-get update
sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y && sudo apt-get clean && sudo apt-get install -y net-tools nmap tcpdump git dialog ufw htop

sudo update-alternatives --set iptables /usr/sbin/iptables-legacy

# USB-Stick as file store
sudo mkdir -p /media/data

sudo blkid | grep 'LABEL="data"' | awk -F '"' '{print $4}'

sudo bash -c "cat >> /etc/fstab" <<'EOF'
#Backup
UUID="<SOME UUID>"     /media/data   ext2    defaults        0       0
EOF

sudo reboot



### Insert bashrc-Aliases
bash -c "cat >> ~/.bashrc" <<'EOF'

### Useful Ailases
alias sourcebash="source ~/.bashrc"
alias bashrc="nano ~/.bashrc"
alias pinge="ping 8.8.8.8"

alias apt-upgrade="sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get autoremove -y && sudo apt-get clean -y"
alias apt-update="sudo apt-get update"

alias temperature="watch -n 1 vcgencmd measure_temp"

EOF

source ~/.bashrc

# Persist logs
sudo sed -i 's/#Storage.*/Storage=persistent/' /etc/systemd/journald.conf
sudo systemctl restart systemd-journald.service

apt-upgrade

# Enable firewall & allow ssh
sudo ufw allow 22
sudo ufw enable
sudo ufw reload
sudo ufw status

# Reboot automatic
sudo -i

# Script for reboot
cat > /root/reboot_automatic.sh << EOF
#!/bin/sh
# Reboot the host
echo "Rebooting...\n"
reboot

EOF
chmod +x /root/reboot_automatic.sh

# Systemd-unit for reboot automatic
cat > /etc/systemd/system/reboot_automatic.service << EOF
[Unit]
Description=Reboot automatically
After=multi-user.target

[Service]
Type=simple
ExecStart=/root/reboot_automatic.sh
EOF

# Systemd-timer every day of month at 4 o'clock
bash -c "cat > /etc/systemd/system/reboot_automatic.timer" << EOF
[Unit]
Description=Run reboot every day at 4 o'clock

[Timer]
OnCalendar=*-*-* 4:00:00

[Install]
WantedBy=timers.target
EOF

systemctl daemon-reload
systemctl enable reboot_automatic.timer
systemctl start reboot_automatic.timer 
systemctl status reboot_automatic.timer reboot_automatic.service

exit
```

## Docker

```
# Install some required packages first
sudo apt-get install -y haveged apt-transport-https ca-certificates curl gnupg2 software-properties-common

# Get the Docker signing key for packages
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add the Docker official repos
echo "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Install Docker
sudo apt-get update
sudo apt-get install -y --no-install-recommends docker-ce cgroupfs-mount
sudo usermod -aG docker $USER
```

## Docker-Compose

```
# Install required packages
sudo apt install -y python3-pip libffi-dev

# Install Docker Compose from pip (using Python3)
# This might take a while
sudo pip3 install docker-compose

docker info
docker-compose --version

# To get rid of WARNINGS from 'docker info' modify the /boot/cmdline.txt, add these parameters and reboot
sudo nano /boot/cmdline.txt
----------------------------------------------------
 cgroup_memory=1 cgroup_enable=memory swapaccount=1
----------------------------------------------------
sudo reboot

docker info
```

## Docker CleanUp routines

### Start-stop docker-compose with systemd

```
sudo cp ./docker_cleanup/docker-compose@.service /etc/systemd/system/docker-compose@.service
sudo systemctl daemon-reload
sudo systemctl enable docker-compose@homeassistant
sudo reboot
```

### Docker cleanup timer with system

```
sudo cp ./docker_cleanup/docker-cleanup.timer ./docker_cleanup/docker-cleanup.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable docker-cleanup.timer
sudo reboot
```
