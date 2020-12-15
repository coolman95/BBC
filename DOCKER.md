# DOCKER nginx, php-fpm, samba

> LOCAL TEST
```

docker run -d --name mysql -v /usr/local/mysql:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/ -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql:5.6


docker run -it --name samba -p 139:139 -p 445:445 -v /var/www/:/mount -d dperson/samba -u "samba;123456" -s "share;/mount/;yes;no;no;all;none"


docker run -d --name php-fpm --privileged=true -p 9000:9000 -v /var/www/:/usr/share/nginx/html php:7.2-fpm
docker run -d --name nginx --privileged=true -p 80:80 -v /var/www/:/usr/share/nginx/html --link php-fpm:php nginx

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

```

> docker nginx.conf default.conf
```
server {
    listen       80;
    listen  [::]:80;
    server_name  95.md.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
    root   /usr/share/nginx/html/bbc;

    location / {
        index  index.html index.htm index.php;

        if (!-e $request_filename) {
            rewrite  ^(.*)$  /index.php?s=$1  last;
            break;
        }
    }

    location /api/ {
        if (!-e $request_filename){
            rewrite  ^/api/(.*)$  /api.php/$1  last;
        }
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        fastcgi_pass   php-fpm:9000;
        fastcgi_index  index.php;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    # location ~ /\.ht {
    #    deny  all;
    # }
}
```

> docker cp
` shell
docker cp ./md.conf nginx:/etc/nginx/conf.d/md.conf
`
