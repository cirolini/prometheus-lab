FROM debian:buster-slim

RUN apt-get update && apt-get install -y nginx mtail procps

# forward request and error logs to docker log collector
#RUN ln -sf /dev/stdout /var/log/nginx/access.log \
#    && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80
EXPOSE 3903

STOPSIGNAL SIGTERM

COPY run.sh /run.sh
COPY rule.mtail /rule.mtail
COPY nginx.conf /etc/nginx/nginx.conf

CMD ./run.sh
