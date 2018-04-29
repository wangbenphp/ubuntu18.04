#一.LAMP
##（一）安装配置
    1.tasksel install lamp-server
    2.1.vi /etc/mysql/mysql.conf.d/mysqld.cnf 添加'#'注释掉其中的"bind-address = 127.0.0.1"
    2.2./etc/init.d/mysql restart
    3.1.登录mysql
    3.2.GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' IDENTIFIED BY 'test' WITH GRANT OPTION;
    3.3.FLUSH PRIVILEGES;
##（二）配置虚拟机
    1.创建项目
    mkdir /var/www/wwwroot/test
    2.添加文件
    vi /var/www/wwwroot/test/index.php
    3.创建config配置文件
    cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/test.conf
    4.修改config配置文件
    vi /etc/apache2/sites-available/test.conf 编辑： 
    ServerName www.test.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/wwwroot/test
    5.启动新的虚拟机配置
    a2ensite test.conf
    6.重启Apache
    /etc/init.d/apache2 restart

#二.LNMP
##（一）安装配置
    1.1.若已安装Apache，需要先停止Apache
    1.2./etc/init.d/apache2 stop
    2.1.若未安装MySQL
    2.2.apt-get install mysql-server php-mysql
    3.apt-get install nginx
    4.apt-get install php-cli php-cgi
    5.apt-get install php-fpm
    6.vi /etc/php/7.2/fpm/pool.d/www.conf
    注释掉 listen = listen = /run/php/php7.2-fpm.sock
    后添加 listen = 127.0.0.1:9000

##（二）配置虚拟机
    1.vi /etc/nginx/sites-enabled/default
    server {
        listen 8080;
        root /var/www/wwwroot/test/public;
        index index.php index.html index.htm;
        server_name www.test.com;
        if (!-e $request_filename) {
            rewrite ^/index.php(.*)$ /index.php?s=$1 last;
            rewrite ^(.*)$ /index.php?s=$1 last;
            break;
        }
        location / {
            try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param PATH_INFO       $fastcgi_path_info;
            fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }
    2.重启服务器
    /etc/init.d/nginx restart
    /etc/init.d/php7.2-fpm restart

#三.Redis&php-redis
##（一）安装配置php-redis
    1.git clone https://github.com/nicolasff/phpredis.git
         注：可能没有预先安装git，只需要按照提示安装即可。
    2.mv phpredis/ /etc/
    3. cd /etc/phpredis
    4.phpize
    5.
    ./configure
    6.make && make install
    7.vi /etc/php/7.2/fpm/conf.d/redis.ini  中 写入（extension=/etc/phpredis/modules/redis.so）退出保存。此操作需要先创建fpm/conf.d/文件夹。
    8.apache配置：
    vi /etc/php/7.2/apache2/php.ini 中写入 （extension=/etc/phpredis/modules/redis.so）
    9.nginx配置：无需配置
    10.重启服务器
     /etc/init.d/apache2 restart 
    /etc/init.d/nginx restart
    /etc/init.d/php7.2-fpm restart
##（二）安装Redis服务器
    1.apt-get install redis-server
    2.vi /etc/redis/redis.conf  注释掉 bind 127.0.0.1 ::1
    3./etc/init.d/redis-server restart

#四.安装CURL
    1.apt-get install curl libcurl3 libcurl3-dev php7.2-curl
    2.重启服务器
    /etc/init.d/nginx restart
    /etc/init.d/apache2 restart
    /etc/init.d/php7.2-fpm restart

#五.安装Composer
    1.进入安装目录：
    cd /usr/local/bin
    2.下载并安装：
    curl -s https://getcomposer.org/installer | php
    3.添加执行权限：
    chmod a+x composer.phar
    4.加入全局命令：
    mv composer.phar /usr/local/bin/composer
    5.更新：
    sudo composer.phar self-update
    6.查看版本号
    composer --version
    7.composer安装后一定要更改国内镜像
    composer config -g repositories.packagist composer https://packagist.phpcomposer.com
    8.apt-get install phpunit