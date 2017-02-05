# 安装ubutu后设置root密码
ubuntu的安装过程中不引导用户设置root用户的密码，刚刚安装完成后的系统，root用户的密码是随即的，而且每个开机的密码都不一样。要想使用root用户登录，可以在安装完成后进行设置：
```
sudo passwd
```
然后输入两次密码，使用root用户登录到系统。

系统默认不允许root通过ssh远程登录。要想使用root用户远程登录，需要修改/etc/ssh/sshd_config文件，找到PermitRootLogin no一行，改为PermitRootLogin yes，然后重启ssh服务
```
service ssh restart
```