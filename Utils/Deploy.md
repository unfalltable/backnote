---
title: 部署
toc: content
keywords: [util]
---

## 云服务器配置Java环境

- 下载JavaSe，maven安装包
- 传到云服务器中解压安装
- 配置环境变量
  - `vim /etc/profile`
  - 写入
    - `export JAVA_HOME=/data/java/jdk`
    - `export PATH=$PATH:$JAVA_HOME/bin`
    - `export MAVEN_HOME=/data/maven/...`
    - `export PATH=$PATH:$MAVEN_HOME/bin`
  - 使环境生效
    - `source /etc/profile`
  - 检查java是否安装成功
    - `java -version`
    - `mvn -v`
- 安装git
  - `yum -y install git`
  - `git --version`
  - 添加公钥
    - `ssh-keygen -t rsa -C "xx@xx.com"`
    - `cat ~/.ssh/id_rsa.pub`
- 安装docker（官网）yum安装
  - 查找mysql
    - `docker search mysql`
    - `docker pull mysql`
  - `mkdir -p /data/docker/mysql.conf`
  - `docker run -p 3306:3306 --name mysql -v /data/docker/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -d mysql --lower-case-table-names=1`
  - `docker run -p 3306:3306 --name mysql -v /data/docker/mysql/conf:/etc/mysql/conf.d -v /data/docker/mysql/conf/my.conf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0 --lower-case-table-names=1`
  - `docker exec -it mysql env LANG=C.UTF-8 bash`
  - 登录mysql
    - `mysql -uroot -p`
    - 创建需要的数据库
- 把项目拉到云服务器中
  - 修改配置信息，端口路径啥的
- 打包
  - `mvn install`
- 用Java启动项目
  - `nohup java -jar {jar名} &`    能使java一直运行

## Blog(Hexo)部署到云服务器		

##### 服务器端

- 安装gcc，gcc-c++，PCRE库，openssl，zlib，nginx，Git，Node.js

  - `sudo apt-get install gcc`

  - `sudo apt-get install zlib1g-dev`

  - `sudo apt-get install libpcre3 libpcre3-dev  `

  - `sudo apt-get install openssl libssl-dev  `

  - `sudo apt install zlib1g`

  - nginx最好是下载包后在安装,下载安装在 /usr/local/ 下

    - `cd /usr/local/`

    - `wget http://nginx.org/download/nginx-1.17.9.tar.gz`

    - `tar -xvf nginx-1.17.9.tar.gz`

    - `cd nginx-1.17.9`

    - `./configure`

    - `make && make install`

    - `/usr/local/nginx/sbin/nginx`

    - 在 /etc/init.d/下创建一个nginx写入以下内容

      - ```yaml
        #!/bin/bash
        #Startup script for the nginx Web Server
        #chkconfig: 2345 85 15
        nginx=/usr/local/nginx/sbin/nginx
        conf=/usr/local/nginx/conf/nginx.conf
        case $1 in 
        start)
        echo -n "Starting Nginx"
        $nginx -c $conf
        echo " done."
        ;;
        stop)
        echo -n "Stopping Nginx"
        killall -9 nginx
        echo " done."
        ;;
        test)
        $nginx -t -c $conf
        echo "Success."
        ;;
        reload)
        echo -n "Reloading Nginx"
        ps auxww | grep nginx | grep master | awk '{print $2}' | xargs kill -HUP
        echo " done."
        ;;
        restart)
        $nginx -s reload
        echo "reload done."
        ;;
        *)
        echo "Usage: $0 {start|restart|reload|stop|test|show}"
        ;;
        esac
        ```

        - `chmod +x nginx`

- 服务器创建git用户

  - `sudo useradd git -m`
  - `sudo passwd git `

- 给git用户增加权限

  - `chmod 740 /etc/sudoers`
  - `vim /etc/sudoers`
    - `在root ALL=(ALL) ALL 下添加 git ALL=(ALL) ALL`
  - `chmod 400 /etc/sudoers`

- 切换到git用户

  - `su git `
  - `cd ~`
  - `mkdir .ssh`
  - `cd .ssh`
  - `vi authorized_keys`
    - 将本地ssh复制到里面去
      - `ssh-keygen -t rsa -C "1563077843@qq.com" `
  - `chmod 600 ~/.ssh/authorized_keys`
  - `chmod 700 ~/.ssh`

- 创建git仓库

  - `cd ~`
  - `git init --bare blog.git`
  - `vi ~/blog.git/hooks/post-receive`
    - 输入`git --work-tree=/home/www/website --git-dir=/home/git/Blog.git  checkout -f`
  - `chmod +x ~/blog.git/hooks/post-receive`

- 本地与服务器连接

  - ssh -v  git@服务器ip

- 修改nginx.conf 中的locat的root

  - root home/www/website

- 修改hexo的配置文件

  - repo : git@服务器ip: home/git/B.git

- 重启nginx

  - service nginx restart

## Hexo博客部署到GitHub

1. ### 前期工作

   1. 生成ssh密钥添加到GitHub

      - 配置Github账号

        - ```
          git config --global user.name "Name"		
          git config --global user.email "Email"
          ```

      - 获取ssh，添加到GitHub中

        - ```
          ssh-keygen -t rsa -C "Email"			
          ```

   2. 修改_config.yml

      - ```
        deploy:
          type: git
          repository: ssh仓库地址
          branch: master
        ```

### 添加评论插件

- valine

## 解决Failed to download metadata for repo 'appstream'

**Step 1:** Go to the `/etc/yum.repos.d/` directory.

```
[root@autocontroller ~]# cd /etc/yum.repos.d/
```

**Step 2:** Run the below commands

```
[root@autocontroller ~]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@autocontroller ~]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```

**Step 3:** Now run the yum update

```
[root@autocontroller ~]# yum update -y
```

## Public Key Retrieval is not allowed

**在连接url上加 allowPublicKeyRetrieval=true**

## Docker部署



