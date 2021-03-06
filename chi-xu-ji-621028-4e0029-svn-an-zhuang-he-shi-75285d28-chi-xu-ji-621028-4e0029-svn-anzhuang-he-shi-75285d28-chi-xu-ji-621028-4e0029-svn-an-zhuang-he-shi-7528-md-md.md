## 持续集成(一)--svn的安装和使用
> 环境为:Centos+Subversion+Apache+Jsvnadmin

### 1.CI服务器
> 172.16.88.142 使用用户:root

### 2.安装前建议更新操作系统
```shell
    yum update  # 更新
    reboot      # 重启
```

### 3.安装apache
```shell
    yum install httpd httpd-devel
    service httpd start
    chkconfig httpd on
    
    # 修改配置
    vi /etc/httpd/conf/httpd.conf
    # 找到ServerName  并修改为
    ServerName localhost:80
    
    # 开启防火墙，打开80端口
    vi /etc/sysconfig/iptables
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
    service iptables restart
```
`验证是否安装:`输入 http://172.16.88.142  出现如下界面
![](http://ojw4jti3h.bkt.clouddn.com/18-3-27/81811013.jpg)

### 4.安装SVN服务
```shell
    yum install mod_dav_svn subversion # 必须安装mod_dav_svn
    # 安装完了后在/etc/httpd/conf.d/中就会多一个subversion.conf文件
    # 重启apache
    service httpd restart
```
`验证安装:ls /etc/httpd/modules/ | grep svn` 出现如下两个文件
![](http://ojw4jti3h.bkt.clouddn.com/18-3-27/10049352.jpg)
`使用命令:svn --version`查看svn的版本

### 5.如何使用svn

#### 5.1 创建svn库主目录（多库模式，一份配置w恩建管理多个库）
```shell
    mkdir /svn/
    cd /etc/httpd/conf.d
    ls  # 其中有一个subversion.conf文件
    vi subversion.conf 
```
添加如下内容
```conf
    #Include /svn/httpd.conf
    <Location /svn/>
    DAV svn
    SVNListParentPath on
    SVNParentPath /svn
    AuthType Basic
    AuthName "Subversion repositories"
    AuthUserFile /svn/passwd.http
    AuthzSVNAccessFile /svn/authz
    Require valid-user
    </Location>
    RedirectMatch ^(/svn)$ $1/
```
创建两个文件
```shell
    touch /svn/passwd.http
    touch /svn/authz
```
重启apache
```shell
    service httpd restart
```

### 6.安装jsvnadmin
    java开发的svn管控台
    
#### 6.1.安装mysql
```shell
1.检查mysql是否安装
	rpm -qa | grep mysql
2.如果安装过，如果想卸载
	yum remove mysql mysql-server mysqllibs mysql-common
	rm -rf /var/lib/mysql
	rm -rf /etc/my.conf
3.安装mysql
	#使用yum安装，在安装之前需要安装mysql的下载源。需要从oracle上下载
	1)下载mysql源包
	我们使用的是centOS6.4，对应的rpm包为:mysql-community-release-el6-5.noarch.rpm
	2)安装mysql的下载源
	yum localinstall mysql-community-release-el6-5.noarch.rpm
	3)安装mysql
	yum install mysql-community-server
4.启动mysql服务
	service mysqld start
5.检查mysql
	chkconfig --list | grep mysqld
6.设置开启启动
	chkconfig mysqld on
7.为了方便远程调用，需要打开防火墙
    vi /etc/sysconfig/iptables
    #添加  打开3306
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
    #重启防火墙
    service iptables restart
8.设置mysql的root密码
	mysqladmin -u root password 'root'
9.mysql授权远程访问
	# 登陆mysql
	mysql -uroot -proot
	mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
    mysql>FLUSH PRIVILEGES;
```

#### 6.2 安装jdk和tomcat
安装jdk
```shell
    #1.解压
    tar -zxvf jdk-8u151-linux-x64.tar.gz
    #2.配置环境变量
    vi ~/.bash_profile
    #输入
    export JAVA_HOME=/root/soft/java_1.7
    export PATH=$PATH:$JAVA_HOME/bin
    source ~/.bash_profile
```
安装tomcat
```shell
    # 修改tomcat端口为9000 和 添加编码为UTF-8
    <Connector port="9000" ... URIEncoding="UTF-8"/>
    
    
    # 开启9000的防火墙
```
安装jsvnadmin
```shell
    # 上传svnadmin-3.0.5.zip 
    # 解压后将svnadmin.war拷贝到webapps中
    # 解压svnadmin.war 再删除svnadmin.war
    unzip svnadmin.war -d svnadmin
    # 修改配置文件
    vi svnadmin/WEB-INF/jdbc.properties
    修改以下内容
    db=MySQL
    # MySQL
    MySQL.jdbc.driver=com.mysql.jdbc.Driver
    MySQL.jdbc.url=jdbc:mysql://127.0.0.1/svnadmin?characterEncoding=utf8
    MySQL.jdbc.username=root
    MySQL.jdbc.password=root
    
    
    
    # 创建svn对应的数据库
    # 创建一个数据库svnadmin
    导入svnadmin.zip的db中的mysql5.sql和lang/en.sql
    
    启动tomcat
    
    http://172.16.88.142:9000/svnadmin
    第一次使用需要设置账号和密码root/root
```

#### 6.4 使用管控台
1. 创建一个库
![](http://ojw4jti3h.bkt.clouddn.com/18-3-27/64586723.jpg)

2. 单击URL会提示认证失败
```
    1.设置用户组，将root用户添加到manager中
    2.因为现在认证为apache
    chown -R apache.apache edu # 将edu设置为apche用户组
    chmod -R 777 edu
    3.关闭SELinux（Linux访问控制）
    修改/etc/selinux/config 文件
    vi /etc/selinux/config
    将SELINUX=enforcing修改为SELINUX=disabled
    重启机器即可
    reboot
```























