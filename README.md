# IoT-Server: Raspberry Pi 3 with Docker / openHub

## Referneces

Raspberry Docker: https://withblue.ink/2020/06/24/docker-and-docker-compose-on-raspberry-pi-os.html

RasperryOS 64-Bit: https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-11-08/

OpenHAB 3 Docker Installation: https://www.laub-home.de/wiki/OpenHAB_3_Docker_Installation

Control Tasmota flashed devices via MQTT: https://community.openhab.org/t/control-tasmota-flashed-devices-via-mqtt/104099

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

## Basic

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

## Run Portainer

```
sudo mkdir -p /media/data/portainer/data

cd ~/iot-server/portainer

docker-compose up -d
```

## Run OpenHAB

```
cd ~

# Build MQTT-WebClient
git clone https://gitlab.bjoern-freund.de/shared/hivemq-mqtt-web-client.git
cd hivemq-mqtt-web-client
docker build -t mqtt-webclient .

sudo mkdir -p /media/data/openhab/userdata
sudo mkdir -p /media/data/mosquitto/data
sudo mkdir -p /media/data/config
sudo mkdir -p /media/influxdb/data
sudo mkdir -p /media/grafana/data

cd ../iot-server/openhab

# Set the passwords by your own!
INFLUX_ADMIN_PASSWD=""
INFLUX_USER_PASSWD=""
INFLUX_USER_READ_PASSWD=""

sed -i "s/INFLUX_ADMIN_PASSWD/${INFLUX_ADMIN_PASSWD}/g" .env
sed -i "s/INFLUX_USER_PASSWD/${INFLUX_USER_PASSWD}/g" .env
sed -i "s/INFLUX_USER_READ_PASSWD/${INFLUX_USER_READ_PASSWD}/g" .env

sudo cp ./mosquitto_conf/mosquitto.conf /media/data/mosquitto/config/
sudo touch /media/data/mosquitto/config/mosquitto.passwd

sudo ufw allow 8080
sudo ufw reload
sudo ufw status

docker-compose up -d
```
