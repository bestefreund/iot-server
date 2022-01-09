## Run Home Assistant

Setup a Home Assistant infrastructure including Mosquitto & Node-Red

```
cd ../iot-server/homeassistant

sudo mkdir -p /media/data/homeassistant/homeassistant/config
sudo mkdir -p /media/data/homeassistant/homeassistant/configurator-config
sudo mkdir -p /media/data/homeassistant/mosquitto/config
sudo mkdir -p /media/data/homeassistant/mosquitto/data
sudo mkdir -p /media/data/homeassistant/nodered/data

# Home Assistant config
sudo cp ./hass-config/configuration.yaml /media/data/homeassistant/homeassistant/config/
sudo cp ./configurator-config/settings.conf /media/data/homeassistant/homeassistant/configurator-config/

localIP="$(ifconfig docker0 | awk '$1 == "inet" {print $2}')"
sed -i "s/<hostip>/${localIP}/g" /media/data/homeassistant/homeassistant/config/configuration.yaml

# Mosquitto config
sudo cp ./mosquitto-config/* /media/data/homeassistant/mosquitto/config/

sudo chown -R $USER /media/data/homeassistant

# Firewall
sudo ufw allow 8080
sudo ufw reload
sudo ufw status

docker-compose up -d

username=""
docker-compose exec mosquitto mosquitto_passwd -c /mosquitto/config/mosquitto.passwd $username

docker restart mosquitto

docker system prune -f --volumes
```

Workaround "Can't compare unknown and SimpleVer"

```
docker exec -it home bash

cat /config/.HA_VERSION
exit

docker rm -f home

ha_version="2021.12.8"
sudo bash -c "echo \"${ha_version}\" > /media/data/homeassistant/homeassistant/config/.HA_VERSION"
```
