FROM debian

RUN apt update -y
RUN apt upgrade -y

RUN apt install -y apache2

ADD index.html /var/www/html/index.html

ADD apacheConf.conf /etc/apache2/apache2.conf

RUN mkdir /etc/apache2/logs

CMD ["apache2", "-D", "FOREGROUND"]

