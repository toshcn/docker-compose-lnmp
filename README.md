# docker-compose-lnmp
A L(Alpine Linux ) + N(Nginx) + M(MariaDB) + P(PHP) Docker compose images
基于PHP 7.1版本，构建干净、轻量级PHP依赖环境。

## Get Docke and Docker Compose
- [Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos)
- [Get Docker CE for Debian](https://docs.docker.com/install/linux/docker-ce/debian)
- [Get Docker CE for Fedora](https://docs.docker.com/install/linux/docker-ce/fedora)
- [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu)
- [Get Docker CE for Fedora](https://docs.docker.com/install/linux/docker-ce/fedora)
- [Install Docker CE from binaries](https://docs.docker.com/install/linux/docker-ce/binaries)
- [Post-installation steps for Linux](https://docs.docker.com/install/linux/linux-postinstall)

## 推荐Github官网安装Docker Compose。

```shell
curl -L https://github.com/docker/compose/releases/download/1.13.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

## 国内环境，推荐使用阿里云Docker Hub加速器服务。

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://muehonsf.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 主要特性
 - 基于PHP 7.1版本，构建干净、轻量级PHP依赖环境。
 - 基于Alpine Linux 最小化Linux环境加速构建镜像。
 - 支持PHP CLI/FPM两种运行模式。Nginx容器与PHP-FPM采用Socket方式连接，提供PHP Web应用环境。
 - 可独立配置容器运行时环境参数，支持容器运行日志、数据与宿主机分离，方便调试与维护。
 - 支持Nginx虚拟站点、SSL证书服务。配置参考Nginx中conf.d目录文件。
 - 支持多个虚拟站点内部程序互通。
 - 使用Docker Compose 编排容器，支持在开发、测试、生产环境中快速完成服务器搭建任务。

## Mariadb 初始密码，docker-compose.yml中配置
  用户：密码
  developer：123456
  ROOT：201314

## Install安装
 1. 克隆Git仓库。需要提前安装好Git。
```shell
git clone https://github.com/toshcn/docker-compose-lnmp && cd docker-compose-lnmp
```
 2. 配置.env环境参数，一般无需修改默认参数。配置PHP_FPM_DOMAIN 支持Nginx容器虚拟主机互通。
```shell
# 生成.env文件
cp .env.example .env
```
 3. 配置Nginx虚拟站点。参考rootfs/etc/nginx/conf.d 目录下配置，复制一份新的站点配置文件，修改域名与主机目录。
```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# default-php server configuration
#
server {
	listen 80;
	listen [::]:80;
	server_name www.default-php.my default-php.my;

	# Add index.php to the list if you are using PHP
    index index.php;

	root         /usr/local/nginx/html/default-php;

	location / {
		# 去除index.php美化url
		try_files $uri $uri/ /index.php$is_args$args;
	}

	location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        #fastcgi_pass 127.0.0.1:9000;
        fastcgi_pass unix:/var/run/php/7.1/php-fpm.sock;
        try_files $uri =404;
    }

    location ~* /\. {
        deny all;
    }
}

### ssl conf ####
server {
	listen 443 ssl;
	listen [::]:443 ssl;
	server_name www.default-php.my default-php.my;

	# Add index.php to the list if you are using PHP
    index index.php;

	root         /usr/local/nginx/html/default-php;

	ssl_certificate /etc/nginx/conf.d/ssl/ssl.crt;
	ssl_certificate_key /etc/nginx/conf.d/ssl/ssl.key;

	ssl_session_cache shared:SSL:1m;
	ssl_session_timeout  10m;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;

	location / {
		# 去除index.php美化url
		try_files $uri $uri/ /index.php$is_args$args;
	}

	location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        #fastcgi_pass 127.0.0.1:9000;
        fastcgi_pass unix:/var/run/php/7.1/php-fpm.sock;
        try_files $uri =404;
    }

    location ~* /\. {
        deny all;
    }
}
```
 4. 使用Docker Compose 快速启动容器。
```shell
sudo docker-compose up -d
```

## 其他说明
docker 常用命令
```shell
# 查看所有运行和者退出的容器
sudo docker ps -a

# 删除停止的容器
sudo docker rm -f contianer_name ...

# 快速停止与删除容器组
sudo docker-compose down

# 删除本地docker镜像
suudo docker rmi -f image_name ....

# 清除所有已经停止运行的容器
sudo docker container prune
```
所有的容器基于Alpine Linux ，默认使用sh shell，进入容器时使用该命令：sudo docker exec -it container_name sh
```shell
docker exec -it alpine-php sh
```

安装 [ctop](https://github.com/bcicen/ctop) 工具可以帮助查看容器在主机的使用情况。

