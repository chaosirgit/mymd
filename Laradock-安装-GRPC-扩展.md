---
title: Laradock 安装 GRPC 扩展
tags:
  - laravel
  - docker
  - laradock
  - 扩展
  - extension
  - grpc
keywords:
  - laravel
  - docker
  - laradock
  - 扩展
  - extension
  - grpc
categories:
  - 技术
abbrlink: fc38307b
date: 2021-06-30 23:31:29
---
### 前言

在 Laradock 环境下安装不常用的 PHP 扩展，不止用与 GRPC 扩展，其他依此类推即可

### 步骤

#### 修改配置文件:

`laradock/docker-compose.yml` 

在 `workspace` 配置参数段

````
workspace:
      build:
        context: ./workspace
        args:
````

添加

```
- INSTALL_PHPGRPC=${WORKSPACE_INSTALL_PHPGRPC}
```

#### 修改启动配置文件

`laradock/.env`

增加两个配置项

```
PHP_FPM_INSTALL_PHPGRPC=true
WORKSPACE_INSTALL_PHPGRPC=true
```

#### 修改 PHP-FPM 配置文件

`laradock/php-fpm/Dockerfile`

```
###########################################################################
# php grpc extension
###########################################################################
ARG INSTALL_PHPGRPC=false

RUN if [ ${INSTALL_PHPGRPC} = true ]; then \
  printf "\n" | pecl install -o -f grpc \
  && rm -rf /tmp/pear \
  && docker-php-ext-enable grpc \
;fi
```

#### 修改 WORKSPACE 配置文件

`laradock/workspace/Dockerfile`

```
###########################################################################
# PHP GRPC EXTENSION
###########################################################################

ARG INSTALL_PHPGRPC=false

RUN if [ ${INSTALL_PHPGRPC} = true ]; then \
    apt-get install zlib1g-dev && \
    pecl install grpc && \
    echo "extension=grpc.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/grpc.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/grpc.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-grpc.ini \
    && php -m | grep -q 'grpc' \
;fi
```

#### 重新构建

`docker-compose build workspace php-fpm`
