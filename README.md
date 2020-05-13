#LNMP环境安装</br>
源码编译安装 LNMP 环境虽然便于自定义，但是对于小型服务器来说，漫长的编译时间让人无法等待。如果能在 10 分钟内搞定环境安装，那就很好了。</br></br>
1、配置yum源</br>
CentOS 7 的 默认 yum 源里的软件包版本可能不是最新的，如果要安装最新的软件包就得配置下 yum 源。</br>
配置 yum 源可以通过直接安装 rpm (Red Hat Package Manager) 包，或者修改 Repository，本文讲解通过安装 rpm 方式。</br>
1.1、安装epel源</br>
首先需要安装 EPEL ( Extra Packages for Enterprise Linux ) yum 源，用以解决部分依赖包不存在的问题</br>
[root@localhost ~]# yum -y install epel-release</br>
1.2、安装MySQL源</br>
官方安装MySQL源参考网址</br>
https://dev.mysql.com/doc/mysql-repo-excerpt/5.6/en/linux-installation-yum-repo.html</br>
安装 rpm 包前需要导入 rpm-GPG-KEY 文件，不然安装过程会出错。将 MySQL rpm-GPG-KEY 另存为 mysql_pubkey.asc 并导入</br>
[root@localhost ~]# rpm --import mysql_pubkey.asc</br>
导入后安装 CentOS 7 的 MySQL rpm 包</br>
[root@localhost ~]# rpm -Uvh http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm</br>
1.3、安装PHP源</br>
PHP 最新的 rpm 官方yum源包地址</br>
http://rpms.remirepo.net/</br>
导入 PHP rpm-GPG-KEY (remi)</br>
[root@localhost ~]# rpm --import http://rpms.remirepo.net/rpm-GPG-KEY-remi</br>
安装 PHP rpm (remi) 包</br>
[root@localhost ~]# rpm -Uvh http://remi.mirrors.arminco.com/enterprise/remi-release-7.rpm</br>
1.4、安装 Nginx 源</br>
官方安装Nginx源参考网址</br>
http://nginx.org/en/linux_packages.html</br>
导入 Nginx rpm-GPG-KEY</br>
[root@localhost ~]# rpm --import http://nginx.org/packages/keys/nginx_signing.key</br>
安装 Nginx rpm 包</br>
[root@localhost ~]# rpm -Uvh http://nginx.org/packages/centos/7/noarch/rpmS/nginx-release-centos-7-0.el7.ngx.noarch.rpm</br>
到目前为止，yum 源已经安装好了 ，接着进行下一步的配置。</br></br>
2、修改相关的yum源文件</br>
MySQL yum 源默认是启用的 MySQL-5.6，PHP yum 源默认都没有启用，Nginx yum 源默认是启用的 Nginx-1.8。定位到 /etc/yum.repos.d/，对后缀为 .repo 的文件进行编辑，修改 enabled 为 1 以启用。</br>
2.1、启用 PHP-7.0的yum源</br>
1、修改 /etc/yum.repos.d/remi.repo，将 [remi] 和 [remi-test] 下面的 enabled=0 改为 enabled=1；</br>
2、修改 /etc/yum.repos.d/remi-php70.repo，将 [remi-php70] 下面的 enabled=0 改为 enabled=1；</br>
[root@localhost ~]# sed -i "/remi\/mirror/{n;s/enabled=0/enabled=1/g}" /etc/yum.repos.d/remi.repo</br>
[root@localhost ~]# sed -i "/test\/mirror/{n;n;s/enabled=0/enabled=1/g}" /etc/yum.repos.d/remi.repo</br>
[root@localhost ~]# sed -i "/php70\/mirror/{n;s/enabled=0/enabled=1/g}" /etc/yum.repos.d/remi-php70.repo</br>
到这一步 yum 配置就算完成了，清除并生成 yum缓存使之生效：</br>
[root@localhost ~]# yum clean all</br>
[root@localhost ~]# yum makecache</br></br>
3、安装 MySQL + PHP + Nginx + phpMyAdmin</br>
yum 源已经配置好了，现在直接安装 MySQL + PHP + Nginx + phpMyAdmin</br>
[root@localhost ~]# yum install -y mysql-community-server nginx php php-bcmath php-fpm php-gd php-json php-mbstring php-mcrypt php-mysqlnd php-opcache php-pdo php-pdo_dblib php-pgsql php-recode php-snmp php-soap php-xml php-pecl-zip phpMyAdmin</br>
注：上面安装的 php-* 可以根据实际使用情况选择安装</br></br>
4、修改MySQL + PHP + Nginx + phpMyAdmin的配置文件</br>
安装完成后，进行下一步的环境配置。</br>
MySQL 配置文件在 /etc/my.cnf.d/</br>
PHP 配置文件在 /etc/php-fpm.d/</br>
Nginx 配置文件在 /etc/nginx/</br>
phpMyAdmin 的配置文件在 /etc/phpMyAdmin/</br>
4.1、配置 MySQL</br>
MySQL 配置文件保持默认，运行一次安全配置即可。</br>
4.1.1、启动 MySQL</br>
[root@localhost ~]# systemctl start mysqld.service</br>
4.1.2、安全配置 MySQL</br>
设置 root 密码、删除匿名用户、禁止 root 远程登录、删除 test 数据库、重新加载权限表，一路 Y 下去</br>
[root@localhost ~]# mysql_secure_installation</br>
4.2、配置 PHP</br>
PHP 默认配置文件使用的是监听 9000 端口进行通信，针对小型单一、没有做负载均衡的服务器，可以使用 unix sock 方式通信。使用 unix sock 方式需要修改 PHP 配置文件。</br>
#更换监听方式</br>
listen = /dev/shm/php-fpm-default.sock</br>
#监听队列最大长度为不限</br>
listen.backlog = -1</br>
#指定监听用户和用户组（需存在）</br>
listen.owner = www</br>
listen.group = www</br>
启动 PHP-FPM：</br>
[root@localhost ~]# systemctl start php-fpm.service</br>
4.3、配置 Nginx</br>
让服务器默认访问显示为 400 提示页。</br>
新建名为 nginx-default.conf 的配置文件</br>
[root@localhost ~]# touch /etc/nginx/conf.d/nginx-default.conf</br>
编辑配置文件</br>
[root@localhost ~]# vim /etc/nginx/conf.d/nginx-default.conf</br>
#将以下信息输入到 nginx-default.conf</br>
server</br>
{</br>
    listen 80 default;</br>
    return 400;</br>
}</br>
防火墙放行 HTTP 端口访问：</br>
firewall-cmd --permanent --zone=public --add-service=http</br>
firewall-cmd --reload</br>
启动 Nginx</br>
[root@localhost ~]# systemctl start nginx.service</br>
这时，在浏览器地址栏输入当前服务器 IP 就会看到一个 400 的提示页面了。</br>
4.4、绑定域名+站点目录+保存日志+运行 PHP的配置文件</br>
server</br>
{</br>
    #监听80端口</br>
listen 80;</br>
#绑定域名 default.com 和 www.default.com</br>
server_name default.com www.default.com;</br>
#设置首页文件，越前优先级越高</br>
index index.html index.htm index.php;</br>
#设置网页编码</br>
charset utf-8;</br>
#设置站点根目录</br>
    root  /home/wwwroot/default;</br>
    #运行 PHP</br>
    location ~ .*\.php$</br>
    {</br>
        #默认使用9000端口和PHP通信</br>
fastcgi_pass  127.0.0.1:9000</br>
        #使用 unix sock 和PHP通信</br>
#fastcgi_pass  unix:/dev/shm/php-fpm-default.sock;</br>
        fastcgi_index index.php;</br>
        #PHP文档根目录</br>
  fastcgi_param DOCUMENT_ROOT  /home/wwwroot/default;</br>
        #PHP 脚本目录</br>
fastcgi_param SCRIPT_FILENAME  /home/wwwroot/default$fastcgi_script_name;</br>
        include fastcgi_params;</br>
        try_files $uri = 404;</br>
    }</br>
    #设置文件过期时间</br>
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp3|wma)$</br>
    {</br>
        expires      30d;</br>
    }</br>
    #设置文件过期时间</br>
    location ~ .*\.(js|css)$</br>
    {</br>
        expires      12h;</br>
    }</br>
    #设置文件访问权限</br>
    location ~* /templates(/.*)\.(bak|html|htm|ini|old|php|tpl)$ {</br>
        allow 127.0.0.1;</br>
        deny all;</br>
    }</br>
    #设置文件访问权限</br>
    location ~* \.(ftpquota|htaccess|htpasswd|asp|aspx|jsp|asa|mdb)?$ {</br>
        deny all;</br>
    }</br>
    #保存日志</br>
    access_log /var/log/nginx/default-access.log main;</br>
    error_log /var/log/nginx/default-error.log crit;</br>
}</br>
4.5、配置 phpMyAdmin</br>
[root@localhost ~]# vi etc/phpMyAdmin/config.inc.php</br>
修改以下内容：</br>
$cfg['Servers'][$i]['host'] = 'localhost';</br>
$cfg['Servers'][$i]['port'] = '3306';</br>
$cfg['Servers'][$i]['socket'] = '/var/lib/mysql/mysql.sock';</br>
$cfg['Servers'][$i]['connect_type'] = 'socket';</br>
$cfg['Servers'][$i]['extension'] = 'mysqli';</br>
$cfg['Servers'][$i]['auth_type'] = 'cookie';</br>
$cfg['UploadDir'] = '/tmp';</br>
$cfg['SaveDir'] = '/tmp';</br>
如果Nginx使用的是上面的进阶代码，那么把 phpMyAdmin 的目录 复制到 /home/wwwroot/default/phpMyAdmin/ 下面，就可通过 http://default.com/phpMyAdmin 访问了</br>
复制 phpMyAdmin 目录</br>
[root@localhost ~]# cp -a /usr/share/phpMyAdmin /home/wwwroot/default/</br>
替换连接形式为目录</br>
[root@localhost ~]# rm -rf /home/wwwroot/default/phpMyAdmin/doc/html</br>
[root@localhost ~]# cp -a /usr/share/doc/phpMyAdmin-<span>*</span>/html /home/wwwroot/default/phpMyAdmin/doc/</br></br>
5、一键脚本
上面已经讲解了如何配置和安装，但是不能每次都这么一步一步来吧？为了节省时间，写了一个一键安装管理脚本，可选择安装 Nginx 1.8/1.9、 MySQL 5.5/5.6/5.7 和 PHP 5.5/5.6/7.0</br>
安装</br>
[root@localhost ~]# yum install -y unzip</br>
[root@localhost ~]# wget https://github.com/maicong/LNMP/archive/master.zip</br>
[root@localhost ~]# unzip master.zip</br>
[root@localhost ~]# cd LNMP-master</br>
[root@localhost ~]# bash lnmp.sh</br>
输出到指定文件</br>
[root@localhost ~]# bash lnmp.sh 2>&1 | tee lnmp.log</br>
管理站点</br>
[root@localhost ~]# service vhost (start,stop,list,add,edit,del,exit) <domain> <server_name> <index_name> <rewrite_file> <host_subdirectory></br>
start 启动</br>
stop 停止</br>
list 列出</br>
add 添加</br>
edit 编辑</br>
del 删除</br>
exit 什么都不做</br>
<domain>: 配置名称，例如：domain</br>
<server_name>: 域名列表，例如：domain.com,www.domain.com</br>
<index_name>: 首页文件，例如：index.html,index.htm,index.php</br>
<rewrite_file>: 伪静态规则文件，保存在 /etc/nginx/rewrite/ 例如：nomal.conf</br>
<host_subdirectory>: 是否支持子目录绑定，on 或者 off</br>
示例：</br>
添加一个标识为 domain 的站点</br>
[root@localhost ~]# service vhost add domain domain.com,www.domain.com index.html,index.htm,index.php nomal.conf on</br>
启动标识为 domain 的站点</br>
[root@localhost ~]# service vhost start domain</br>
停止标识为 domain 的站点</br>
[root@localhost ~]# service vhost stop domain</br>
编辑标识为 domain 的站点</br>
[root@localhost ~]# service vhost edit domain</br>
删除标识为 domain 的站点</br>
[root@localhost ~]# service vhost del domain</br>
列出所有站点</br>
[root@localhost ~]# service vhost list</br>
备份数据</br>
[root@localhost ~]# service vbackup (start,list,del) <delete name.tar.gz></br>
start 添加</br>
list 列出</br>
del 删除</br>
示例：</br>
添加一个新的备份</br>
[root@localhost ~]# service vbackup start</br>
列出备份文件</br>
[root@localhost ~]# service vbackup list</br>
删除一个备份</br>
[root@localhost ~]# service vbackup del name.tar.gz</br>
Linux公社的RSS地址：https://www.linuxidc.com/rssFeed.aspx</br>
本文永久更新链接地址：https://www.linuxidc.com/Linux/2018-10/155057.htm</br>
