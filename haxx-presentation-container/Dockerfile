FROM nginx:1.19.2-alpine

RUN mkdir -p /templates

ADD before.html /templates/before.html
ADD after.html /templates/after.html
ADD entrypoint.sh /opt/entrypoint.sh

RUN chmod +x /opt/entrypoint.sh

ENTRYPOINT ["/bin/sh", "-c", "/opt/entrypoint.sh"]
EXPOSE 80
