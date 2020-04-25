# laradock-env

## 目录

- [简介](#简介)
- [配置](#配置)
- [实践](#实践)
  - [workspace](#workspace)
  - [php-fpm](#php-fpm)
  - [php-worker](#php-worker)

## 简介

Laradock 环境配置。

## 配置

- 切换 apt 源

  ```
  ### Environment ###########################################
  
  # If you need to change the sources (i.e. to China), set CHANGE_SOURCE to true
  CHANGE_SOURCE=true
  # Set CHANGE_SOURCE and UBUNTU_SOURCE option if you want to change the Ubuntu system sources.list file.
  UBUNTU_SOURCE=aliyun
  ```

- Workspace Composer 配置

  ```
  WORKSPACE_COMPOSER_REPO_PACKAGIST=https://mirrors.aliyun.com/composer/
  ```

- Workspace Node / NVM 配置

  ```
  WORKSPACE_NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
  WORKSPACE_INSTALL_NODE=true
  WORKSPACE_NPM_REGISTRY=https://registry.npm.taobao.org
  WORKSPACE_INSTALL_YARN=true
  ```

- 如果在安装 PPA 仓库的软件出错，则使用中科大的 PPA 加速，如编辑 `workspace/Dockerfile` 中的片段如下（第 3 行）：

  ```
  # always run apt update when start and after add new source list, then clean up at end.
  RUN set -xe; \
      find /etc/apt/sources.list.d/ -type f -name "*.list" -exec sed -i.bak -r 's#deb(-src)?\s*http(s)?://ppa.launchpad.net#deb\1 https://launchpad.proxy.ustclug.org#ig' {} \; && \
      apt-get update -yqq && \
      pecl channel-update pecl.php.net && \
      groupadd -g ${PGID} laradock && \
      useradd -u ${PUID} -g laradock -m laradock -G docker_env && \
      usermod -p "*" laradock -s /bin/bash && \
      apt-get install -yqq \
        apt-utils \
        #
        #--------------------------------------------------------------------------
        # Mandatory Software's Installation
        #--------------------------------------------------------------------------
        #
        # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
        # installed on the base image 'laradock/workspace' image. If you want
        # to add more Software's or remove existing one, you need to edit the
        # base image (https://github.com/Laradock/workspace).
        #
        # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
        libzip-dev zip unzip \
        # Install the zip extension
        php${LARADOCK_PHP_VERSION}-zip \
        # nasm
        nasm && \
        php -m | grep -q 'zip'
  ```

- 建议替换 Workspace 里面的 insecure_id_rsa、insecure_id_rsa.pub 为自己的 ssh key。

- 如果安装 `phpredis` 过程中提示 `Package "phpredis" does not have REST info xml available` 或其他错误，尝试修改 `php-fpm/Dockerfile` 中的 `PHP REDIS EXTENSION` 模块如下（去 pecl.php.net 查询包的下载地址）：

  ```
  ###########################################################################
  # PHP REDIS EXTENSION
  ###########################################################################
  
  ARG INSTALL_PHPREDIS=false
  
  RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
      # Install Php Redis Extension
      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        pecl install -o -f redis-4.3.0; \
      else \
        pecl install -o -f https://pecl.php.net/get/redis-5.2.1.tgz; \
      fi \
      && rm -rf /tmp/pear \
      && docker-php-ext-enable redis \
  ;fi
  ```

  或下载包到 php-fpm 目录下，修改 `Dockerfile` 内容如下：

  ```
  ###########################################################################
  # PHP REDIS EXTENSION
  ###########################################################################
  
  ARG INSTALL_PHPREDIS=false

  ARG REDIS_INSTALL_VERSION=5.2.1

  COPY redis-${REDIS_INSTALL_VERSION}.tgz /tmp/redis-${REDIS_INSTALL_VERSION}.tgz
  
  RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
      # Install Php Redis Extension
      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        pecl install -o -f redis-4.3.0; \
      else \
        pecl install -o -f /tmp/redis-${REDIS_INSTALL_VERSION}.tgz; \
      fi \
      && rm -rf /tmp/pear \
      && docker-php-ext-enable redis \
  ;fi
  ```

  其他 pecl 包同理。

- 构建镜像时使容器通过代理上网，例子：

  ```bash
  $ docker-compose build \
    --build-arg http_proxy=socks5://host.docker.internal:1080 \
    --build-arg https_proxy=socks5://host.docker.internal:1080 \
    workspace
  ```

  `docker-compose build` 的过程中，灵活切换是否使用代理，可增加成功的机率。`docker-compose up` 的过程中，也可以通过再次 `docker-compose build` 快速改变配置参数。

## 实践

### workspace

1. `cd ~/workspace`

2. `git clone git clone https://github.com/Laradock/laradock.git`

3. `cd laradock`

4. `cp env.example .env`

5. 编辑 `.env` 文件：

   WORKSPACE：

   `APP_CODE_PATH_HOST=../projects/`

   `PHP_VERSION=7.2`

   `CHANGE_SOURCE=true`

   `WORKSPACE_COMPOSER_REPO_PACKAGIST=https://mirrors.aliyun.com/composer/`

   `WORKSPACE_NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node`

   `WORKSPACE_NPM_REGISTRY=https://registry.npm.taobao.org`

   `WORKSPACE_INSTALL_WORKSPACE_SSH=true`

   `WORKSPACE_INSTALL_PYTHON=true`

   `WORKSPACE_INSTALL_SWOOLE=true`

   `WORKSPACE_INSTALL_MYSQL_CLIENT=true`

   `WORKSPACE_INSTALL_PING=true`

   `WORKSPACE_TIMEZONE=Asia/Shanghai`

   `WORKSPACE_INSTALL_GIT_PROMPT=true`

   `MYSQL_VERSION=5.7`

   PHP-FPM：

   `PHP_FPM_INSTALL_SWOOLE=true`

   PHP-WORKER：

   `PHP_WORKER_INSTALL_SWOOLE=true`

6. 复制 SSH key 到 workspace 下

   ```bash
   $ cp ~/.ssh/id_rsa workspace/insecure_id_rsa
   ```

   ```bash
   $ cp ~/.ssh/id_rsa.pub workspace/insecure_id_rsa.pub
   ```

7. 使用中科大的 PPA 加速，编辑 `workspace/Dockerfile` 中的片段如下（第 3 行）：

   ```
   # always run apt update when start and after add new source list, then clean up at end.
   RUN set -xe; \
       find /etc/apt/sources.list.d/ -type f -name "*.list" -exec sed -i.bak -r 's#deb(-src)?\s*http(s)?://ppa.launchpad.net#deb\1 https://launchpad.proxy.ustclug.org#ig' {} \; && \
       apt-get update -yqq && \
       pecl channel-update pecl.php.net && \
       groupadd -g ${PGID} laradock && \
       useradd -u ${PUID} -g laradock -m laradock -G docker_env && \
       usermod -p "*" laradock -s /bin/bash && \
       apt-get install -yqq \
         apt-utils \
         #
         #--------------------------------------------------------------------------
         # Mandatory Software's Installation
         #--------------------------------------------------------------------------
         #
         # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
         # installed on the base image 'laradock/workspace' image. If you want
         # to add more Software's or remove existing one, you need to edit the
         # base image (https://github.com/Laradock/workspace).
         #
         # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
         libzip-dev zip unzip \
         # Install the zip extension
         php${LARADOCK_PHP_VERSION}-zip \
         # nasm
         nasm && \
         php -m | grep -q 'zip'
   ```

8. 下载指定版本的 `Swoole` 包到 `workspace` 目录下，并编辑 `workspace/Dockerfile` 内容片段如下：

   ```
   ###########################################################################
   # Swoole EXTENSION
   ###########################################################################
   
   ARG INSTALL_SWOOLE=false
   
   ARG SWOOLE_INSTALL_VERSION_1=2.0.10
   ARG SWOOLE_INSTALL_VERSION_2=2.2.0
   ARG SWOOLE_INSTALL_VERSION_3=4.4.18

   COPY swoole-${SWOOLE_INSTALL_VERSION_1}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_1}.tgz
   COPY swoole-${SWOOLE_INSTALL_VERSION_2}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_2}.tgz
   # 替换成当前最新版本
   COPY swoole-${SWOOLE_INSTALL_VERSION_3}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_3}.tgz
   
   RUN if [ ${INSTALL_SWOOLE} = true ]; then \
       # Install Php Swoole Extension
       if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
         pecl -q install /tmp/swoole-${SWOOLE_INSTALL_VERSION_1}.tgz; \
       else \
         if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
           pecl install /tmp/swoole-${SWOOLE_INSTALL_VERSION_2}.tgz; \
         else \
           pecl install /tmp/swoole-${SWOOLE_INSTALL_VERSION_3}.tgz; \
         fi \
       fi && \
       echo "extension=swoole.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini && \
       ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-swoole.ini \
       && php -m | grep -q 'swoole' \
   ;fi
   ```

9. 下载 `WORKSPACE_AST_VERSION` 指定的 `AST` 包到 `workspace` 目录下，并编辑 `workspace/Dockerfile` 内容片段如下：

   ```
   ###########################################################################
   # AST EXTENSION
   ###########################################################################
   
   ARG INSTALL_AST=false
   ARG AST_VERSION=1.0.3
   ENV AST_VERSION ${AST_VERSION}
   
   COPY ast-${AST_VERSION}.tgz /tmp/ast-${AST_VERSION}.tgz
   
   RUN if [ ${INSTALL_AST} = true ]; then \
       # AST extension requires PHP 7.0.0 or newer
       if [ $(php -r "echo PHP_MAJOR_VERSION;") != "5" ]; then \
           # Install AST extension
           printf "\n" | pecl -q install /tmp/ast-${AST_VERSION}.tgz && \
           echo "extension=ast.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/ast.ini && \
           phpenmod -v ${LARADOCK_PHP_VERSION} -s cli ast \
       ;fi \
   ;fi
   ```

### php-fpm

1. 使用中科大的 PPA 加速，编辑 `php-fpm/Dockerfile` 中的片段如下（第 3 行）：

   ```
   # always run apt update when start and after add new source list, then clean up at end.
   RUN set -xe; \
       find /etc/apt/sources.list.d/ -type f -name "*.list" -exec sed -i.bak -r 's#deb(-src)?\s*http(s)?://ppa.launchpad.net#deb\1 https://launchpad.proxy.ustclug.org#ig' {} \; && \
       apt-get update -yqq && \
       pecl channel-update pecl.php.net && \
       apt-get install -yqq \
         apt-utils \
         #
         #--------------------------------------------------------------------------
         # Mandatory Software's Installation
         #--------------------------------------------------------------------------
         #
         # Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
         # are installed on the base image 'laradock/php-fpm' image. If you want
         # to add more Software's or remove existing one, you need to edit the
         # base image (https://github.com/Laradock/php-fpm).
         #
         # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
         libzip-dev zip unzip && \
         if [ ${LARADOCK_PHP_VERSION} = "7.3" ] || [ ${LARADOCK_PHP_VERSION} = "7.4" ]; then \
           docker-php-ext-configure zip; \
         else \
           docker-php-ext-configure zip --with-libzip; \
         fi && \
         # Install the zip extension
         docker-php-ext-install zip && \
         php -m | grep -q 'zip'
   ```

2. 到 pecl 下载指定版本 `redis` 包到 php-fpm 目录下，修改 `Dockerfile` 内容如下：

   ```
   ###########################################################################
   # PHP REDIS EXTENSION
   ###########################################################################
   
   ARG INSTALL_PHPREDIS=false

   ARG REDIS_INSTALL_VERSION_1=4.3.0
   ARG REDIS_INSTALL_VERSION_2=5.2.1

   COPY redis-${REDIS_INSTALL_VERSION_1}.tgz /tmp/redis-${REDIS_INSTALL_VERSION_1}.tgz
   COPY redis-${REDIS_INSTALL_VERSION_2}.tgz /tmp/redis-${REDIS_INSTALL_VERSION_2}.tgz
   
   RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
       # Install Php Redis Extension
       if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
         pecl install -o -f /tmp/redis-${REDIS_INSTALL_VERSION_1}.tgz; \
       else \
         pecl install -o -f /tmp/redis-${REDIS_INSTALL_VERSION_2}.tgz; \
       fi \
       && rm -rf /tmp/pear \
       && docker-php-ext-enable redis \
   ;fi
   ```

3. 下载指定版本的 `Swoole` 包到 `php-fpm` 目录下，并编辑 `php-fpm/Dockerfile` 内容片段如下：

   ```
   ###########################################################################
   # Swoole EXTENSION
   ###########################################################################
   
   ARG INSTALL_SWOOLE=false
   
   ARG SWOOLE_INSTALL_VERSION_1=2.0.10
   ARG SWOOLE_INSTALL_VERSION_2=2.2.0
   ARG SWOOLE_INSTALL_VERSION_3=4.4.18
   
   COPY swoole-${SWOOLE_INSTALL_VERSION_1}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_1}.tgz
   COPY swoole-${SWOOLE_INSTALL_VERSION_2}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_2}.tgz
   # 替换成当前最新版本
   COPY swoole-${SWOOLE_INSTALL_VERSION_3}.tgz /tmp/swoole-${SWOOLE_INSTALL_VERSION_3}.tgz
   
   RUN if [ ${INSTALL_SWOOLE} = true ]; then \
       # Install Php Swoole Extension
       if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
         pecl install /tmp/swoole-${SWOOLE_INSTALL_VERSION_1}.tgz; \
       else \
         if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
           pecl install /tmp/swoole-${SWOOLE_INSTALL_VERSION_2}.tgz; \
         else \
           pecl install /tmp/swoole-${SWOOLE_INSTALL_VERSION_3}.tgz; \
         fi \
       fi && \
       docker-php-ext-enable swoole \
       && php -m | grep -q 'swoole' \
   ;fi
   ```

4. 下载最新版本的 `imagick` 包到 `php-fpm` 目录下，并编辑 `php-fpm/Dockerfile` 内容片段如下：

   ```
   ###########################################################################
   # ImageMagick:
   ###########################################################################
   
   USER root
   
   ARG INSTALL_IMAGEMAGICK=false
   
   ARG IMAGICK_INSTALL_VERSION=3.4.4
   
   COPY imagick-${IMAGICK_INSTALL_VERSION}.tgz /tmp
   
   RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
       apt-get install -y libmagickwand-dev imagemagick && \
       pecl install /tmp/imagick-${IMAGICK_INSTALL_VERSION}.tgz && \
       docker-php-ext-enable imagick \
   ;fi
   ```

### php-worker

1. 下载指定版本的 `memcached`、`mcrypt`、`mongodb` 包到 `php-worker` 目录下，并编辑 `php-worker/Dockerfile` 内容片段如下：

   ```
   ARG MEMCACHED_INSTALL_VERSION=3.1.5
   ARG MCRYPT_INSTALL_VERSION=1.0.1
   ARG MONGODB_INSTALL_VERSION=1.7.4
   
   COPY memcached-${MEMCACHED_INSTALL_VERSION}.tgz /tmp/memcached-${MEMCACHED_INSTALL_VERSION}.tgz
   COPY mcrypt-${MCRYPT_INSTALL_VERSION}.tgz /tmp/mcrypt-${MCRYPT_INSTALL_VERSION}.tgz
   COPY mongodb-${MONGODB_INSTALL_VERSION}.tgz /tmp/mongodb-${MONGODB_INSTALL_VERSION}.tgz
   
   RUN pecl channel-update pecl.php.net \
     && pecl install /tmp/memcached-${MEMCACHED_INSTALL_VERSION}.tgz /tmp/mcrypt-${MCRYPT_INSTALL_VERSION}.tgz /tmp/mongodb-${MONGODB_INSTALL_VERSION}.tgz \
     && docker-php-ext-enable memcached mongodb
   ```

2. 下载指定版本的 `Swoole` 包到 `php-worker` 目录下，并编辑 `php-worker/Dockerfile` 内容片段如下：

   ```
   ###########################################################################
   # Swoole EXTENSION
   ###########################################################################
   
   ARG INSTALL_SWOOLE=false
   
   ARG SWOOLE_LATEST_VERSION=4.4.18
   
   COPY swoole-2.0.10.tgz /tmp/swoole-2.0.10.tgz
   COPY swoole-2.2.0.tgz /tmp/swoole-2.2.0.tgz
   # 替换成当前最新版本
   COPY swoole-${SWOOLE_LATEST_VERSION}.tgz /tmp/swoole-${SWOOLE_LATEST_VERSION}.tgz
   
   RUN if [ ${INSTALL_SWOOLE} = true ]; then \
       # Install Php Swoole Extension
       if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
         pecl -q install /tmp/swoole-2.0.10.tgz; \
       else \
         if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
           pecl install /tmp/swoole-2.2.0.tgz; \
         else \
           pecl install /tmp/swoole-${SWOOLE_LATEST_VERSION}.tgz; \
         fi \
       fi \
       && docker-php-ext-enable swoole \
   ;fi
   ```
