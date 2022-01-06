FROM alpine/git
WORKDIR /tmp
RUN git clone https://github.com/hivemq/hivemq-mqtt-web-client.git

FROM secanis/hivemq-mqtt-web-client
COPY --from=0 /tmp/hivemq-mqtt-web-client/* /usr/local/apache2/htdocs/
