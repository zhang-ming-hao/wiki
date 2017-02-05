## 安装vsftpd
```
yum -y install vsftpd
```
## 启动停止服务
- 启动ftp服务
```
service vsftpd start
```
- 查看ftp服务状态
```
service vsftpd status 
```
- 重启ftp服务
```
service vsftpd restart
```
- 关闭ftp服务
```
service vsftpd stop
```
## 修改配置文件
配置文件的路径为：/etc/vsftpd/vsftpd.conf，以下是常用的配置项：  
```
# 禁止匿名用户anonymous登录
anonymous_enable=NO
# 允许本地用户登录
local_enable=YES
# 让登录的用户有写权限(上传，删除)
write_enable=YES
# 默认umask
local_umask=022
# 把传输记录的日志保存到/var/log/vsftpd.log
xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=NO
# 允许ASCII模式上传
ascii_upload_enable=YES 
# 允许ASCII模式下载
ascii_download_enable=YES
# 使用20号端口传输数据
connect_from_port_20=YES
# 欢迎标语
ftpd_banner=Welcome to use my test ftp server.
# 接下来的三条配置很重要
# chroot_local_user设置了YES，那么所有的用户默认将被chroot，
# 也就用户目录被限制在了自己的home下，无法向上改变目录。
# chroot_list_enable设置了YES，即让chroot用户列表有效。
# ★超重要：如果chroot_local_user设置了YES，那么chroot_list_file
# 设置的文件里，是不被chroot的用户(可以向上改变目录)
# ★超重要：如果chroot_local_user设置了NO，那么chroot_list_file
# 设置的文件里，是被chroot的用户(无法向上改变目录)
chroot_list_enable=YES
# touch /etc/vsftpd/chroot_list 新建
chroot_list_file=/etc/vsftpd/chroot_list
use_localtime=YES
# 以standalone模式在ipv4上运行
listen=YES
# PAM认证服务名，这里默认是vsftpd，在安装vsftpd的时候已经创建了这个pam文件，
# 在/etc/pam.d/vsftpd，根据这个pam文件里的设置，/etc/vsftpd/ftpusers
# 文件里的用户将禁止登录ftp服务器，比如root这样敏感的用户，所以你要禁止别的用户
# 登录的时候，也可以把该用户追加到/etc/vsftpd/ftpusers里。
pam_service_name=vsftpd
```

## 创建FTP用户
vsftpd使用linux系统用户。  
- 创建用户
```
useradd -d 用户主目录 -s /sbin/nologin -M 用户名
```
- 添加密码
```
passwd 用户名
密码
确认密码
```