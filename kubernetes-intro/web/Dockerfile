# Origin - https://github.com/nginxinc/NGINX-Demos/tree/master/nginx-hello-nonroot

FROM nginx:1.19.3-alpine

RUN chmod -R a+w /var/cache/nginx/ \
        && touch /var/run/nginx.pid \
        && chmod a+w /var/run/nginx.pid \
        && rm /etc/nginx/conf.d/* \
        && mkdir /app \
        && chmod -R a+rx /app

COPY hw.conf /etc/nginx/conf.d/
COPY homework.html /app/
USER 1001
EXPOSE 8000/tcp
