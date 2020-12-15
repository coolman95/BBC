# DOCKER nginx, php-fpm, samba

> LOCAL TEST
```

docker run -d --name mysql -v /usr/local/mysql:/var/lib/mysql -v /usr/local/mysql/conf:/etc/mysql/ -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql:5.6


docker run -it --name samba -p 139:139 -p 445:445 -v /var/www/:/mount -d dperson/samba -u "samba;123456" -s "share;/mount/;yes;no;no;all;none"


docker run -d --name php-fpm --privileged=true -p 9000:9000 -v /var/www/:/usr/share/nginx/html php:7.2-fpm
docker run -d --name nginx --privileged=true -p 80:80 -v /var/www/:/usr/share/nginx/html --link php-fpm:php nginx

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

---

docker cp ./md.conf nginx:/etc/nginx/conf.d/md.conf

```

> docker GD ZIP 扩展
```

#容器中
#echo "deb http://mirrors.163.com/debian/ stretch main contrib non-free\ndeb http://mirrors.163.com/debian/ stretch-updates main contrib non-free\ndeb http://mirrors.163.com/debian/ stretch-backports main contrib non-free" > /etc/apt/sources.list  #软件源修改为网易镜像站源

apt update  #更新软件源
apt install -y libwebp-dev libjpeg-dev libpng-dev libfreetype6-dev #安装各种库
docker-php-source extract #解压源码
cd /usr/src/php/ext/gd  #gd源码文件夹
docker-php-ext-configure gd --with-webp-dir=/usr/include/webp --with-jpeg-dir=/usr/include --with-png-dir=/usr/include --with-freetype-dir=/usr/include/freetype2   #准备编译
docker-php-ext-install gd   #编译安装
php -m | grep gd

#重启容器

```

```
获取扩展文件
wget http://pecl.php.net/get/zip-1.13.5.tgz

复制到容器当中
docker cp ./zip-1.13.5.tgz php7:/usr/src/php/ext/
加压文件
tar -zvxf zip-1.13.5.tgz
重命名
mv zip-1.13.5 zip
进入zip 文件
cd /usr/src/php/ext/zip
运行安装文件
docker-php-ext-install zip

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

