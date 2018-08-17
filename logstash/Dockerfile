FROM logstash:5-alpine

COPY ./conf.d /etc/conf.d

RUN mkdir -p /logs/output /logs/input

VOLUME [ "/logs/output", "/logs/input" ]

EXPOSE 5000

ENTRYPOINT [ "logstash" , "-f" , "/etc/conf.d" ]