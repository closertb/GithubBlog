---
title: '基于EC2, 配置一个全栈服务实例（nginx + tomcat + mysql）'
date: 2019-06-24 21:38:54
tags:
---

最近世道动荡，在前往高级的路上走出了车到山前必有路，睁眼一看是绝路的感觉。所以就索性瞎折腾一下。领了一个服务器，开启了一个伪全栈的运维之路，各种服务线上部署。
## 服务器申请与实例连接接
腾讯免费七天，阿里要钱，山里娃就在亚马逊AWS申请了一个可免费使用一年的EC2云服务器，[申请链接][1]，步骤很简单，跟着提示一步一步整就是，唯一要提醒的就是，需要准备一张信用卡，一张能支持外汇（$）结算的最好。
申请到资格后，选择你的云服务，选择对应的区域，你需要给服务实例选择一个操作系统，linux,windows常用的都可选（注意观察，我们只选免费的，很重要，很重要， 很重要）。然后配置安全组，bla,bla,....,然后启动实例。保存好你的密钥，然后打开ssh终端连接实例。操作步骤可以打开管理面板，选择实例-》选择实例-》连接-》根据面板提示连接。

![clipboard.png][7]

## 安装与配置
### 基础组件安装与配置  
登录进服务后，就可以开启一段服务器配置之旅了。如果你和我一样，对Linux常用的命令行还不熟悉，你可能需要这样一份手册：[Linux常用命令大全][2]。我选择的镜像是Ubuntu，如果你和我选择的一样，那么下面的命令你可以直接用，如果是redhat或者centos，有些命令，你需要自己去探索。先把一些常用的工具安装上：
```cmd
    sudo apt-get install unzip // 解压工具
    sudo apt-get install git   // git工具
    sudo apt-get install wget // 下载工具  
    sudo apt-get install nginx // 下载nginx   
```
### node服务安装与配置
node安装是一个相对简单的过程，你可以直接[查看官网][3]，然后按照提示一步一步进行。非常重要的一步就，你需要建立你命令的软链接。在这里我列出自己的操作步骤：
 -下载：sudo wget https://nodejs.org/download/release/latest-v8.x/node-v8.16.0-linux-x64.tar.xz
 - 建一个文件夹：sudo mkdir -p /usr/local/lib/nodejs
 - 解压到上面新建文件夹：sudo tar -xJvf node-v8.16.0-linux-x64.tar.xz -C /usr/local/lib/nodejs 
 - 建立node可执行命令链接：sudo ln -snf /usr/local/lib/nodejs/node-v8.16.0-linux-x64/bin/node /usr/bin/node
 - 重复上述步骤，建立npm可执行链接
 - 测试有效性：node -v // node-v8.16.0
### jdk的安装与配置
centos可[参考链接][4]，同时也适用于ubuntu，现在使用wget下载jdk有点麻烦（需要鉴权），所以我是本地下载，然后scp上传上去的，以下是我的操作：
 - 上传：scp -i "big.pem" jdk-8u211-linux-x64.tar.gz ubuntu@ec2-13-114-140-94.ap-northeast-1.compute.amazonaws.com:/home
 - 解压并重命名为tomcat：tar xzf jdk-8u211-linux-x64.tar.gz
 - 建立链接java，javac，jar：sudo ln -snf /home/tomcat/bin/java（你解压后的目录） /usr/bin/java，     其他两个照做
 - 测试： java -version
### tomcat服务的安装与配置
 - 下载 wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.41/bin/apache-tomcat-8.5.41.tar.gz
 - 解压
 - 进入到bin目录，然后执行 ./startup.sh
 - 实例ip查看服务运行情况（前提是在安全策略允许了8080端口的连接）
### 数据库的安装与配置
mysql的安装复杂一点，折腾了自己大量时间，在redhat8上没有安装成功mysql5，也迫使我把镜像换成了ubuntu，曲折的路就不多说了，直接说顺利的。如果直接使用apt-get install mysql安装，默认是安装mysql8，所以在开启安装前，需要借助mysql-apt-config增加一段配置，具体安装步骤，请查考前人栽下的树：[Ubuntu 16.04安装MySQL][5]：通过APT方式安装。
安装好之后，开启mysql，并登录
 - 开启：sudo service mysql start
 - 查看端口：sudo netstat -anp | grep mysql
 - 登录：sudo mysql -u root -p
 - 显示数据库：show databases
至此，本地链接已经ok，但是mysql远程链接数据库仍然报无发连接。原因很多，这里提及两个我遇到的。
 - 实例安全策略：实例安全策略默认只开启了22端口，如果你像我一样，你还需要开启3306端口（和mysql相关），80端口，443端口
 - mysql自己的安全策略，默认只对127.0.0.1开启，即本地开启
第一种很简单，去EC2面板上修改你正在用的安全策略，加入3306端口，并启用。
第二种稍微麻烦一点，你需要如下操作：
 - 打开mysql配置文件vim /etc/mysql/mysql.conf.d/mysqld.cnf，字段如下图所示

![clipboard.png][8]

 - 将bind-address = 127.0.0.1注销

 - 登录mysql，并执行grant all privileges on *.* to 'root'@'%' identified by 'yourpassword';在有些版本上，不能直接这样操作root用户，所以，你需要新建一个用户，然后对这个用户执行grant all privileges操作
 - flush privileges;​
 - 重启mysql，或则你的实例
** 关于sql批量导入 **
 - 导出：mysqldump -u root -p -d targetDatabaseName > targetFileName.sql, 根据提示，输入密码，导出ok，当前文件价应该就存在一个targetFileName.sql，里面包含了表数据与结构；
 - 远程导入mysqldump -h 132.72.192.432 -P 3306 -u root -p targetDatabaseName < targetFileName.sql(导入之前你得已经创建了targetDatabaseName库）
 - 上面的远程操作不一定好使，下面的这种操作，成功率更高。ssh远程登录，然后mysql登录，创建你的targetDatabase库，然后use targetDatabase，采用source命令：source /targetFile/targetFileName.sql；回车，即可导入成功，在这些操作之前，你需要将你本地的sql文件通过scp传送到远端服务器；

## nginx配置与域名的解析
安装nginx，无非就想解决静态资源访问，反向代理，gzip，负载均衡问题，由于我这是初级使用，没有涉及到负载均衡的情形。
### 运行
前面已经安装了nginx，看是否安装成功，可使用命令运行：
```cmd
   sudo systemctl start nginx.service
   // 其他常用命令
   sudo systemctl stop nginx.service
   sudo systemctl reload nginx.service
   sudo systemctl status nginx.service
```
然后通过ip访问80端口，如果顺利可以看到，但是一般都是不顺利，显示403。
![clipboard.png][9]

### 解决nginx权限不足造成的访问问题
可参考连接：[nginx权限不足造成的访问问题][6]，我遇到的是第四项，即SELinux。

### 开启静态资源服务
我要代理的静态资源是一个react框架打包的网站，话不多说，直接列出我的配置：

```config
    server {
        listen       80;
        listen       [::]:80;
        server_name  h5.closertb.site;
        index index.html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        location / {
            root   /home/static;
            autoindex on;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    } 
```
### 服务的代理
我的后端服务是基于tomcat，端口为8080。
```config    
   server {
        listen       80;
        listen       [::]:80;
        server_name  server.closertb.site;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
            proxy_pass       http://127.0.0.1:8080;
            proxy_set_header Host      $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }  
```
门户网站是一个基于nextJs的ssr渲染，所以需要代理一个node服务，这个服务运行在8500端口 
```config    
    server {
        listen       80;
        listen       [::]:80;
        server_name  client.closertb.site;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
            proxy_pass       http://127.0.0.1:8500;
            proxy_set_header Host      $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```  
关于nginx代理的一些常识，从上面三段配置，我们可以看到，我们都是对80端口进行了代理，但可以设置不同类别，指向不同资源，区分点就在于server_name，通过此来决定返回代理的内容。代理上面也列出了两种，静态资源的代理（root）和服务的代理（proxy_pass）。
为什么都要用80端口？
 - 首先，对于一个成熟的网站，他的域名一般是不带端口号的；
 - 另外，对于一个服务器，对外开放的端口越多，被攻击的可能性更高；
 - 最后，在域名解析时，是无法将一个域名解析到一个带端口的ip地址上的，至少阿里云是这样的

![clipboard.png][10]

### 开启gzip
为什么要开启gzip？因为虽然前端框架的不断侵蚀和资源的丰富，虽然网络更快了，打包压缩策略也用了，但100K以上的资源加载确实很慢，所以我们还得借助gzip来加快资源的获取。话不多说，上图自己感受：
开启gzip前

![clipboard.png][12]

![clipboard.png][11]

开启gzip后

![clipboard.png][13]

![clipboard.png][14]
看完上面两幅图，你就会觉得这是肉眼可见的差别，提速增效太明显了
```config
        # Gzip Settings

        gzip on;
        gzip_disable "msie6";
        gzip_min_length 10k;
        gzip_vary on;
        # gzip_proxied any;
        gzip_comp_level 3;
        gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
```
## 启动服务
 - 启动mysql，上面已经说过了；
 - 上传前端构建包，并解压静态资源；
 - 上传jar包，运行 java -jar target-version.jar,启动后端服务；
 - git download，然后npm i，npm run prod运行node服务；
至此，一个纯前端服务，纯后端服务，SSR渲染服务就启动完成，访问相应的域名即可查看。

## 一些杂七杂八的冷门命令整理
其实，这整篇都是一些个人杂七杂八的知识整理，做个笔记，方便以后翻阅。但下面的碎片知识，才是真的杂，包括一些不常用的命令
 - 修改文件权限命令chmod (-R) xxx targetpath, eg: chmod -R 777 /home 将home目录及其子目录的属性修改为可读，可写，可执行；
 - 查看文件夹下子文件夹及文件信息ls -l；相比与ls，-l就显得特别有用，可查看文件的读写权限，如果建立了ln链接的，可查看链接信息；
 - 查看运行端口对应的pid；netstat -tunlp |grep prot；eg: netstat -tunlp |grep 8080
 - 查看相应应用运行的服务列表：ps -ax | grep applicationName；eg: ps -ax | grep node
 - 查看和远程链接网络状态，相当于ping，telnet ip/domain port;telnet www.baidu.com 80
暂时就列出这么多吧，公司快要挂了，是时候开始准备面试了，Fighting。  

  [1]: https://aws.amazon.com/cn/ec2/?nc2=h_m1
  [2]: https://blog.csdn.net/gyshun/article/details/82112883
  [3]: https://github.com/nodejs/help/wiki/Installation
  [4]: https://tecadmin.net/install-java-8-on-centos-rhel-and-fedora/#
  [5]: https://www.cnblogs.com/jpfss/p/7944622.html
  [6]: https://www.cnblogs.com/williamjie/p/9604594.html
  [7]: https://image-static.segmentfault.com/252/119/2521198261-5d0ed4a991a38_articlex
  [8]: https://image-static.segmentfault.com/104/549/1045494038-5d0f2c75d2432_articlex
  [9]: https://image-static.segmentfault.com/411/661/4116610423-5d0f448b34519_articlex
  [10]: https://image-static.segmentfault.com/346/466/3464662076-5d0f9b80b4bf9_articlex
  [11]: https://image-static.segmentfault.com/247/158/2471584855-5d0f9f70d1006_articlex
  [12]: https://image-static.segmentfault.com/593/909/593909200-5d0f9f597677f_articlex
  [13]: https://image-static.segmentfault.com/370/520/3705207070-5d0f9dcb53581_articlex
  [14]: https://image-static.segmentfault.com/364/663/3646631339-5d0f9f11e906b_articlex