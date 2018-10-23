FROM debian:jessie
ARG WEBAPP_DOMAIN

RUN apt-get update && apt-get install -y \
    nginx

ADD nginx.conf /etc/nginx/
ADD atelier.conf /etc/nginx/sites-available/

RUN sed -i "s/HOSTNAME/${WEBAPP_DOMAIN}/g" /etc/nginx/sites-available/atelier.conf

RUN ln -s /etc/nginx/sites-available/atelier.conf /etc/nginx/sites-enabled/atelier
RUN rm /etc/nginx/sites-enabled/default

RUN echo "upstream php-upstream { server php:9000; }" > /etc/nginx/conf.d/upstream.conf

RUN usermod -u 1000 www-data
RUN echo 'alias ll="ls -alh"' >> ~/.bashrc

CMD ["nginx"]

EXPOSE 80
EXPOSE 443
