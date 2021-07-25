FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf
COPY ./files/ /usr/local/apache2/htdocs/
COPY files /var/www/public
RUN chown -R www-data:www-data /var/www/public
EXPOSE 80
