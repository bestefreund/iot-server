FROM alpine/git
WORKDIR /tmp
RUN git clone https://github.com/hivemq/hivemq-mqtt-web-client.git

FROM hypriot/rpi-busybox-httpd
COPY --from=0 /tmp/hivemq-mqtt-web-client/* /www
