# 安装dokuwiki

1. 安装apache2服务器
    ```
    apt install apache2
    ```
1. 安装php5
    ```
    apt install php5
    ```
1. 安装libapache2-mod-php5
    ```
    apt install libapache2-mod-php5
    ```
1. 将dokuwiki复制到/var/www/html目录下，并将dokuwiki的目录权限设置为755
1. 在浏览器中访问install.php，在页面中填写所需的信息。

# 安装插件

使用管理员用户登录，选择->管理->扩展管理器->搜索和安装，打开插件安装页面
1. 安装markdown插件  
    搜索markdown，安装markdown扩展

2. 显示侧边栏  
    搜索simpleindex，安装扩展。完成后编辑sidebar页面，添加如下内容
    ```
    <simpleindex sidebar>
    ```
    simpleindex为调用的插件，sidebar是需要隐藏的命名空间
    
# 创建页面
在url中输入不存在的id，会出现“该链接不存在”的提示，此时点击右侧的创建按扭，可以创建该id相应的页面。  
id中使用如下规则：
```
命名空间:页面名
```
命名空间相当于一个分类，页面名为相应页面的名称。

# 删除页面
要删除页面，只需要清空页面的内容并保存即可。  
要删除命名空间，只需要清空该命名空间内的所有页面即可。