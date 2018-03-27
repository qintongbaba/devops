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
    RedirectMath ^(/svn)$ $1/
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



















