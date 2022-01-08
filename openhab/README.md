# Run OpenHAB

Setup an OpenHAB infrastructure including Mosquitto, InfluxDB & Grafana

```
cd ~

git clone https://gitlab.bjoern-freund.de/docker/mosquitto.git

cd mosquitto
docker build -t mosquitto .

cd ../iot-server/openhab

sudo mkdir -p /media/data/openhab/openhab/userdata
sudo mkdir -p /media/data/openhab/mosquitto/data
sudo mkdir -p /media/data/openhab/mosquitto/config
sudo mkdir -p /media/data/openhab/influxdb/data
sudo mkdir -p /media/data/openhab/grafana/data

# Set the passwords by your own!
INFLUX_ADMIN_PASSWD=""
INFLUX_USER_PASSWD=""
INFLUX_USER_READ_PASSWD=""

sed -i "s/INFLUX_ADMIN_PASSWD/${INFLUX_ADMIN_PASSWD}/g" .env
sed -i "s/INFLUX_USER_PASSWD/${INFLUX_USER_PASSWD}/g" .env
sed -i "s/INFLUX_USER_READ_PASSWD/${INFLUX_USER_READ_PASSWD}/g" .env

sudo ufw allow 8080
sudo ufw reload
sudo ufw status

docker-compose up -d

$username=""
docker-compose exec mosquitto mosquitto_passwd -c /mosquitto/config/mosquitto.passwd $username
```