# laradock-env

## 目录

- [简介](#简介)
- [配置](#配置)

## 简介

Laradock 环境配置。

## 配置

- 修改宿主机上的项目路径

  ```
  # Point to the path of your applications code on your host
  APP_CODE_PATH_HOST=../projects/
  ```

- 切换合适的 PHP 版本

  ```
  # Select a PHP version of the Workspace and PHP-FPM containers (Does not apply to HHVM).
  # Accepted values: 7.4 - 7.3 - 7.2 - 7.1 - 7.0 - 5.6
  PHP_VERSION=7.2
  ```

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

- Workspace SSH

  ```
  WORKSPACE_INSTALL_WORKSPACE_SSH=true
  ```

- Workspace MySQL 客户端

  ```
  WORKSPACE_INSTALL_MYSQL_CLIENT=true
  ```

- Workspace Ping

  ```
  WORKSPACE_INSTALL_PING=true
  ```

- Workspace 设置时区

  ```
  WORKSPACE_TIMEZONE=Asia/Shanghai
  ```

- Workspace Bash Git Prompt

  ```
  WORKSPACE_INSTALL_GIT_PROMPT=true
  ```

- MySQL

  ```
  MYSQL_VERSION=5.7
  ```

如果在安装 PPA 仓库的软件出错，则使用中科大的 PPA 加速，编辑 `workspace/Dockerfile` 中的片段如下（第 3 行）：

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

如果无法安装 PHP AST 拓展，如不安装：

```
WORKSPACE_INSTALL_AST=false
```

## 其他

- 构建镜像时使容器通过代理上网，例子：

  ```bash
  $ docker-compose build \
    --build-arg http_proxy=socks5://host.docker.internal:1080 \
    --build-arg https_proxy=socks5://host.docker.internal:1080 \
    workspace
  ```

  `docker-compose build` 的过程中，灵活切换是否使用代理，可增加成功的机率。`docker-compose up` 的过程中，也可以通过再次 `docker-compose build` 快速改变配置参数。

- 建议替换 Workspace 里面的 insecure_id_rsa、insecure_id_rsa.pub 为自己的 ssh key。
