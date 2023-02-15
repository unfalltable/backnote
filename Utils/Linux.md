---
title: Linux
toc: content
keywords: [util]
---

# 基础

- Linux的目录结构是级层式的树状目录结构
  - 根目录： "/"
- 一切皆为文件

### 远程登录Linux

- Linux需要开启SSHD服务监听
  - dpkg -l | grep ssh         查看当前的ubuntu是否安装了ssh-server服务
  - sudo apt-get install openssh-server  安装ssh-server服务
  - ps -e | grep ssh            确认ssh-server是否启动了
  - sudo service ssh start  如果没有则可以这样启动

### 开启root用户

1. 在普通用户下输入 `sudo passwd root`

2. 修改50-ubuntu.conf

   - `sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf`

     ```
     [Seat:*]
     user-session=ubuntu
     greeter-show-manual-login= true
     all-guest=false #这个可以 不用配置
     ```

3. 修改gdm-autologin

   - `sudo vim /etc/pam.d/gdm-autologin`

     - 注释auth  required  pam_succeed_if.so user != root quiet_success
     - 注释%PAM-1.0

     ```
     #%PAM-1.0
     auth    requisite       pam_nologin.so
     #auth   required        pam_succeed_if.so user != root quiet_success
     auth    optional        pam_gdm.so
     auth    optional        pam_gnome_keyring.so
     auth    required        pam_permit.so
     ```

4. 修改gdm-password

   - `sudo vim /etc/pam.d/gdm-password`

     - 注释掉 auth required pam_succeed_if.so user != root quiet_success

     ```
     #%PAM-1.0
     auth    requisite       pam_nologin.so
     #auth   required        pam_succeed_if.so user != root quiet_success
     @include common-auth
     auth    optional        pam_gnome_keyring.so
     @include common-account
     ```

5. 修改/root/.profile文件

   - `sudo vim/root/.profile`


# 常用命令

## Linux

| 效果                         | 指令                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 重启                         | reboot                                                       |
| 解压                         | tar -zxvf                                                    |
| 创建文件                     | touch {文件名}<br />gedit {文件名}<br />echo {内容} > {文件名} |
| 创建文件夹                   | mkdir                                                        |
| 删除                         | rm -rf                                                       |
| 重命名文件                   | mv 原文件名 新文件名                                         |
| 查找文件                     | find                                                         |
| rz                           | 上传文件                                                     |
| 查找端口                     | netstat -tunlp \| grep {port}                                |
| 根据名称查看进程             | ps -ef \| grep {进程名}                                      |
| 杀死端口                     | kill -9 PID                                                  |
| 查看内存占用                 | top                                                          |
| 查看进程中线程的内存占用情况 | ps H -eo pid, tid, %cpu \| grep 进程id                       |
| 查看某PID进程状态            | ps -aux \| grep PID                                          |
| 开放端口                     | firewall-cmd --zone=public --add-port=3306/tcp-permanent<br />firewal-cmd -reload |
| 关闭防火墙                   | systemctl stop firewalld<br />systemctl disable firewalld    |
| 创建并编辑文件               | vim 文件名                                                   |
| 后台运行                     | nohup                                                        |
| 改变权限                     | chmod                                                        |
| 启动 / 重启 / 停止进程       | service 进程名 start / reload / stop                         |
|                              |                                                              |

## Mysql

- 进入mysql /log
  - tail -f catalina.out      动态查看运行消息

## Tomcat

- 查看实时日志
  - `tail -f catalina.out ` 

## Redis

| 效果      | 指令（redis目录下）          |
| --------- | ---------------------------- |
| 后台启动  | redis-server redis.conf      |
| 停止      | redis-cli -u 123321 shutdown |
| 进入redis | redis-cli -a 密码            |

## mariadb

| 效果     | 指令                     |
| -------- | ------------------------ |
| 开机自起 | systemctl enable mariadb |
|          |                          |
|          |                          |



# 常见问题

- 使用不了vim命令
  - 安装vim
    - `sudo apt-get install vim-gtk`
- 无法定位软件包
  - 更新源
    - `sudo apt-get update`
- 缺少git依赖
  - `npm install `--`save hexo-deployer-git`
- ssh连接时 出现Host key verification failed.
  - ssh-keygen -R 想要访问的ip
- bash: nginx: command not found
  - 配置环境变量


- nginx 403 forbidden
  - 检查hooks 下的 post-receive

- 服务器拒绝访问
  - 服务器端重启nginx

### SSH服务器拒绝了密码

- cd /etc/ssh/

- vim sshd_config

- ```
  # Authentication:
  LoginGraceTime 120
  PermitRootLogin yes
  StrictModes yes
  ```

# 安装

## 安装vscode

```shell
sudo apt update
sudo apt install software-properties-common apt-transport-https wget
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt install code
sudo apt update
sudo apt upgrade
```

## 安装wordpress

- https://devpress.csdn.net/centos/62ecebbe19c509286f417a0f.html

## 安装mariadb

```shell
yum install mariadb-server
```

## 安装apache

```shell
yum -y install httpd
mysql_secure_installation #初始化数据库，全选y
```

## 安装php

```shell
yum install -y epel-release
rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum --enablerepo=remi install -y php74-php
yum --enablerepo=remi install -y php74-php php74-php-gd php74-php-xml php74-php-sockets php74-php-session php74-php-snmp php74-php-mysql
yum install -y php74-php-fpm
systemctl enable php74-php-fpm
ln -s /opt/remi/php74/root/usr/bin/php /usr/bin/php
```



# 常见工具

## VIM的使用

- `ctrl s` 		停止向终端传输数据
  - `ctrl q`    退出vim
- 按EXC跳到命令模式
  - :w 					保存文件但不退出vi
  - :w file           将修改另外保存到File中，不退出vi
  - :w!                强制保存，不退出vi
  - :wq               保存并退出vim
  - :q                   不保存文件，退出vi
  - :q!                 不保存文件，强制退出vi
  - :e!                 放弃所有修改，从上次保存文件开始再编辑

# 升级

## Glibc升级至2.31

1. 下载包

```shell
wget https://mirrors.aliyun.com/gnu/glibc/glibc-2.31.tar.gz
```

2. 解压

```shell
tar -zxvf glibc-2.31.tar.gz
```

3. 升级python到3.4之后
4. 升级gcc到9.3.0
5. 升级make到4.3













