# laradock-env

## 目录

[简介](#简介)

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

- 安装 Oh My ZSH

  ```
  ### Install Oh My ZSH! ####################################
  
  # If you want to use "Oh My ZSH!" with Laravel autocomplete plugin, set SHELL_OH_MY_ZSH to true.
  
  SHELL_OH_MY_ZSH=true
  ```

- Workspace Composer 配置

  ```
  WORKSPACE_COMPOSER_REPO_PACKAGIST="https://mirrors.aliyun.com/composer/"
  ```

- Workspace Node / NVM 配置

  ```
  WORKSPACE_NVM_NODEJS_ORG_MIRROR="http://npm.taobao.org/mirrors/node"
  WORKSPACE_NPM_REGISTRY="https://registry.npm.taobao.org"
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
