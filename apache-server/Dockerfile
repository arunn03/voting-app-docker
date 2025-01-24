FROM httpd:2.4.62-alpine

WORKDIR /usr/local/apache2/

RUN rm ./htdocs/index.html

COPY ./data ./htdocs/data

CMD ["httpd-foreground"]

