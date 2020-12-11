# DOCKER nginx, php-fpm, samba

> LOCAL TEST
```

docker run -d --name mysql -v /usr/local/mysql:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/ -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql:5.6


docker run -it --name samba -p 139:139 -p 445:445 -v /var/www/:/mount -d dperson/samba -u "samba;123456" -s "share;/mount/;yes;no;no;all;none"


docker run -d --name php-fpm --privileged=true -p 9000:9000 -v /var/www/:/usr/share/nginx/html php:7.2-fpm
docker run -d --name nginx --privileged=true -p 80:80 -v /var/www/:/usr/share/nginx/html --link php-fpm:php nginx

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

```
