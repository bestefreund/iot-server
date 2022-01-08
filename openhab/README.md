# Run OpenHAB

```
cd ~

sudo mkdir -p /media/data/openhab/openhab/userdata
sudo mkdir -p /media/data/openhab/mosquitto/data
sudo mkdir -p /media/data/openhab/mosquitto/config
sudo mkdir -p /media/data/openhab/influxdb/data
sudo mkdir -p /media/data/openhab/grafana/data

cd iot-server/openhab

# Set the passwords by your own!
INFLUX_ADMIN_PASSWD=""
INFLUX_USER_PASSWD=""
INFLUX_USER_READ_PASSWD=""

sed -i "s/INFLUX_ADMIN_PASSWD/${INFLUX_ADMIN_PASSWD}/g" .env
sed -i "s/INFLUX_USER_PASSWD/${INFLUX_USER_PASSWD}/g" .env
sed -i "s/INFLUX_USER_READ_PASSWD/${INFLUX_USER_READ_PASSWD}/g" .env

sudo cp ./mosquitto_conf/mosquitto.conf /media/data/openhab/mosquitto/config/
sudo touch /media/data/openhab/mosquitto/config/mosquitto.passwd

sudo ufw allow 8080
sudo ufw reload
sudo ufw status

docker-compose up -d
```
