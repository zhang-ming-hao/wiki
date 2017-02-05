# Python第三方包的制作
我们在开发过程中，为了便于使用，经常会封装一些库。过去的做法会把封装好的文件复制到多个项目中使用，这样做会使库出现多个版本，不便于维护。  
我们可以把这些库制作为第三方的包，这样修改这些包并发布，各项目就可以同步公共的代码。  
以IBMMQClient为例，说明第三方的包的制作要点。

## 目录结构
IBMMQClient项目源码的目录结构如下：
```
IBMMQClient
├─ setup.py
│  
└─IBMMQClient
    ├─ IBMMQClient.py
    ├─ readme.md
    ├─ __init__.py
    │  
    └─pymqi
        ├─ CMQC.py
        ├─ CMQCFC.py
        ├─ CMQXC.py
        ├─ CMQZC.py
        ├─ mqic.dll
        ├─ pymqe.py
        ├─ pymqe.pyd
        └─ __init__.py
```

## setup.py的写法
先看代码：
```python
#!/usr/bin/env python
# coding:utf-8

"""
# IBMMQClient库的安装脚本
"""

from distutils.core import setup

setup(
    name='IBMMQClient',
    version='0.1',
    packages=['IBMMQClient', 'IBMMQClient/pymqi'],
    url='http://www.sdysit.com/',
    license='',
    author='Shandong Youth IT LTD.',
    author_email='',
    description='IBM WebSphere MQ Client',
    data_files=[
        ('Lib/site-packages/IBMMQClient', ['IBMMQClient/readme.md']),
        ('Lib/site-packages/pymqi', ['IBMMQClient/pymqi/mqic.dll', 'IBMMQClient/pymqi/pymqe.pyd'])
    ]
)
```
代码中调用了setup函数，各参数的含义为：
* name：包的名称
* version：包的版本
* packages：要处理的包，即包的根目录
* url：该包的作者的url
* license：许可证
* author：作者
* author_email：作者邮箱
* description：包的描述
* data_files：除py文件外，还需要的文件。  
    该参数是一个元组的列表，一个元组处理一个文件夹。元组的第一项为目标文件夹，如果使用相对路径，则为sys.prefix或sys.exec_prefix的相对路径；元组的第二项为一个列表，列表中的每一项是要放入目标文架的文件的路径。