## Run Home Assistant

Setup a Home Assistant infrastructure including Mosquitto & Node-Red

```
cd ~

git clone https://gitlab.bjoern-freund.de/docker/mosquitto.git

cd mosquitto
docker build -t mosquitto .

cd ../iot-server/homeassistant

sudo mkdir -p /media/data/homeassistant/homeassistant/config
sudo mkdir -p /media/data/homeassistant/mosquitto/data
sudo mkdir -p /media/data/homeassistant/mosquitto/config
sudo mkdir -p /media/data/homeassistant/nodered/data

# Home Assistant config
localIP="$(ifconfig eth0 | awk '$1 == "inet" {print $2}')"

sed -i "s/<hostip>/${localIP}/g" hass-config/configuration.yaml

sudo cp ./hass-config/configuration.yaml /media/data/homeassistant/homeassistant/config/

sudo ufw allow 8080
sudo ufw reload
sudo ufw status

docker-compose up -d

$username=""
docker-compose exec mosquitto mosquitto_passwd -c /mosquitto/config/mosquitto.passwd $username
```
