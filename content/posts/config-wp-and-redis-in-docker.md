---
title: Configure Wordpress and Redis in Docker
date: 2023-02-09 16:40:01
tags:
    - linux
    - play
---

## Foreword

The station has been transferred, and the original 4g small device has been transferred to the current 32g single-use top surface. php+redis+mysql), a caddy under external service. All in all，Containerized incense is complete! 😀

## Transfer plan

My transfer plan is very simple, so it's a problem, and it's finally on the new equipment.

This job demand blow one down `UpdraftPlus-Reward/Recovery` This issue has been completed, serious damage, support automatic release of contents to various long-distance service top, comprehensive and unlimited (Amazon S3, Dropbox, Google Cloud hard drive, rackspace, (S) FTP, WebDAV Japanese electronic mail), the calculation is the most powerful data I've used in front of me, and the data I'm talking about is true. Fudo Hashihara Retreat

This song is ready to use, it's just a matter of demand, and it's ready for the new environment.

## Environmental Department

Former corporate department plan: Docker containerization (wordpress official image, redis official image, mysql official image)

Precondition:

-   docker
-   docker-compose

Directly use docker-compose unit immediately, the following content:

```yml
version: "3.1"

services:
    wordpress:
        image: wordpress
        container_name: wordpress
        restart: always
        ports:
            # This is the end of the movie
            - 127.0.0.1:8080:80
        environment:
            WORDPRESS_DB_HOST: db
            WORDPRESS_DB_USER: wpuser
            WORDPRESS_DB_PASSWORD: wppassword
            WORDPRESS_DB_NAME: wpdb
        volumes:
            - wordpress:/var/www/html

    db:
        image: mysql:5.7
        container_name: wordpress_mysql
        restart: always
        environment:
            MYSQL_DATABASE: wpdb
            MYSQL_USER: wpuser
            MYSQL_PASSWORD: wppassword
            MYSQL_RANDOM_ROOT_PASSWORD: "1"
        volumes: -wp_db:/var/lib/mysql

    redis:
        image: redis
        container_name: wordpress_redis
        restart: always

volumes:
    wordpress:
    wp_db:
```

Use the following commands

```bash
docker-compose -p wp up -d
```

After the execution is completed, it will be placed on the opposite side of the agent.

My web apparel use caddy, specific combination possible, please refer to my previous text, this layout format is as follows:

```bash
https://jinzh.me {
         tls jinzhongjia@qq.com
# Arrangement mail box
         reverse_proxy 127.0.0.1:8080
#Placement anti-replacement site
         encode gzip
#opengzip
}
```

After that, deploy wordpress redis, redis uplink department completed, but not include wordpress image php redsi exhibition, and update wordpress redis information

For this reason, in the advanced container:

```bash
docker exec -it wordpress bash
#wordpress is container name
```

Wordpress container package management usage is apt, first update:

```bash
apt update
apt install vim
# Anso vim
pecl install redis-5.1.1
# Pass through pecl and redis redis
docker-php-ext-enable redis
# Opening Exhibition
```

Add the following contents to `wp-config.php`:

```php
define("WP_REDIS_HOST","redis");//Main machine name (docker container name)
define("WP_REDIS_PORT",6379);
define("WP_REDIS_PASSWORD","yourpassword");
# Other parameters Reference: https://github.com/rhubarbgroup/redis-cache/wiki/Connection-Parameters
```

The deployment is complete, and the wp has been opened (recommended Redis Object Cache).
