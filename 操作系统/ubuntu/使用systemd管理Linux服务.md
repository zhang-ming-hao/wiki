# 使用systemd管理Linux服务
在众多Linux系统中，都是采用init脚本的方式进行管理，但是每种Linux都不太一致，很难在每个Liunx发行版中共用。  
目前，许多Linux厂商都采用systemd作为系统服务管理程序，自Ubuntu15.04、Centos7后，都放弃了原来的init脚本方式。  
在Linux系统中，守护进程一般都以字母d结尾，取自daemon。而systemd以system命名，意味着他要做整个系统的守护进程。

## systemd与init脚本的差异
init脚本本质上就是shell脚本，可以被一些标准参数如 start，stop，restart 等调用。shell 脚本常见的缺点就是，慢、可读性不强、太详细又很傲娇。虽然它们很灵活（毕竟那就是代码呀），但是有些事只用脚本做还是显得太困难了，比如安排并列执行、正确监视进程，或者配置详细执行环境。init脚本还有一个硬伤就是，臃肿，重复代码太多。因为上述的“标准参数”必须要靠各个脚本来实现，而且各个脚本之间的实现都差不多。  
而 Systemd 则进行了统一实现，也就是说在 Systemd service 中完全就不需要、也看不到这部分内容。这使得 Systemd 服务非常简明易读。  
systemd兼容init脚本，但是根据以上理由，最好针对所有您安装的守护精灵都使用原生 Systemd 服务来启动。

## systemd语法
systemd不需要写脚本实现服务的管理，它采用配置文件的方式。它的配置文件与windows中ini文件的格式基本相同，只是多了一些约定：
* “#” 开头的行后面的内容被认为是注释
* systemd中的布尔值，1、yes、on、true 都是开启，0、no、off、false 都是关闭
* 当一个选项设置多个值时，可以多次设置，也可以一次设置并用空格分隔
* 时间单位默认是秒，如果想使用毫秒或分钟，需要进行特殊说明
* 一个配置文件分为三个部分，分别是：
    + Unit：单元
    + Service：服务
    + Install：安装

以ubuntu为例，其配置文件的路径为/lib/systemd/system。

## systemd的单元
systemd的单元有以下几种，它们的名称及扩展名如下：
* 系统服务（.service）
* 挂载点（.mount）
* sockets（.sockets）
* 系统设备（.device）
* 交换分区（.swap）
* 文件路径（.path）
* 启动目标（.target）
* 由 systemd 管理的计时器（.timer）  

一般的，我们做的都是系统服务，其他几种都是随着系统的发行版内置的，不需要也不能由我们进行改动。因此，下面只介绍系统服务的相关内容。  

systemd的单元（Unit）的主要作用是描述该单元与其他单元的依赖关系，有如下选项：
* Description：单元描述，描述单元的文字
* Requires：本单元依赖的单元，这个单元启动了，那么它依赖的单元也会被启动; 它依赖的单元被停止了，该单元也会被停止。  
`注：这个设定并不能控制某单元与它依赖的单元的启动顺序。Systemd不是先启动Requires再启动本单元，而是在本单元被激活时，并行启动两者。因此，需要使用After或Before来控制单元的启动顺序。`
* Wants：此选项是Requires的弱化版。当此单元被启动时，所有这里列出的其他单元只是尽可能被启动。 但是，即使某些单元不存在或者未能启动成功， 也不会影响此单元的启动。 推荐使用此选项来设置单元之间的依赖关系。
* Requisite：此选项是Requires的强化版。当单元启动时，这里列出的单元必须已经全部处于启动成功的状态， 否则，将不会启动此单元(也就是直接返回此单元启动失败的消息)， 并且同时也不会启动那些尚未成功启动的单元。
* BindsTo：与Requires类似，不同之处在于：如果这里列出的单元停止运行或者崩溃，那么也会连带导致该单元自身被停止。 这就意味着该单元可能因为某个单元的主动退出、某个设备被拔出、某个挂载点被卸载， 而被强行停止。
* PartOf：与Requires类似，不同之处在于：仅作用于单元的停止或重启。其含义是，当停止或重启这里列出的某个单元时，也会同时停止或重启该单元自身。注意，这个依赖是单向的，该单元自身的停止或重启并不影响这里列出的单元。
* Before/After：强制指定单元之间启动的先后顺序。停止顺序与启动顺序正好相反。
* Conflicts：一个单元的启动会停止与它“冲突”的单元，反之亦然。
* OnFailure：当该单元进入失败("failed")状态时， 将会启动列表中的单元。

## systemd的服务
systemd的服务（Service）的主要作用是描述该单元运行的环境、权限、以及启动/停止/重启时使用的命令等。有如下选项：
* Type：设置进程的启动类型， 必须设为 simple, forking, oneshot, dbus, notify, idle 之一。如果不设置，则默认为simple。  
    + 如果设为 simple (设置了 ExecStart= 但未设置 BusName= 时的默认值)， 那么表示 ExecStart= 进程就是该服务的主进程。 如果此进程需要为其他进程提供服务， 那么必须在该进程启动之前先建立好通信渠道(例如套接字)， 以加快后继单元的启动速度。
    + 如果设为 forking ， 那么表示 ExecStart= 进程将会在启动过程中使用 fork() 系统调用。 这是传统UNIX守护进程的经典做法。 也就是当所有的通信渠道都已建好、启动亦已成功之后， 父进程将会退出，而子进程将作为该服务的主进程继续运行。 对于此种进程， 建议同时设置 PIDFile= 选项， 以帮助 systemd 准确定位该服务的主进程， 进而加快后继单元的启动速度。
    + oneshot (未设置 ExecStart= 时的默认值) 与 simple 类似， 不同之处在于该进程必须在 systemd 启动后继单元之前退出。 此种类型通常需要设置 RemainAfterExit= 选项。
    + dbus (设置了 ExecStart= 也设置了 BusName= 时的默认值) 与 simple 类似， 不同之处在于该进程需要在 D-Bus 上获得一个由 BusName= 指定的名称。 systemd 将会在启动后继单元之前， 首先确保该进程已经成功的获取了指定的 D-Bus 名称。 设为此类型相当于隐含的依赖于 dbus.socket 单元。
    + notify 与 simple 类似， 不同之处在于该进程将会在启动完成之后通过 sd_notify(3) 之类的接口发送一个通知消息。 systemd 将会在启动后继单元之前， 首先确保该进程已经成功的发送了这个消息。 如果设为此类型， 那么 NotifyAccess= 将只能设为 all 或 main 之一， 并且 NotifyAccess= 的默认值 也会变为 main 。 注意，目前 Type=notify 尚不能在 PrivateNetwork=yes 的情况下正常工作。
    + idle 与 simple 类似， 不同之处在于该进程将会被延迟到所有的操作都完成之后再执行。 这样可以避免控制台上的状态信息 与shell脚本的输出混杂在一起。
* RemainAfterExit：当该服务的所有进程全部退出之后， 是否依然将此服务视为活动(active)状态。 默认值为 no
* GuessMainPID：在无法明确定位该服务主进程的情况下，systemd是否应该猜测主进程的PID(可能不正确)。该选项仅在设置了Type=forking但未设置PIDFile=的情况下有意义。如果PID猜测错误，那么该服务的失败检测与自动重启功能将失效。 默认值为 yes
* PIDFile：守护进程的PID文件，必须是绝对路径。强烈建议在 Type=forking 的情况下明确设置此选项。 systemd将会在此服务启动后从此文件中读取主守护进程的PID。systemd不会写入此文件，但会在此服务停止后删除它(若存在)。 
* BusName：设置与此服务通信所使用的 D-Bus 名称。在 Type=dbus 的情况下必须明确设置此选项。
* ExecStart：在启动该服务时需要执行的命令行(命令+参数)。仅在设置了 Type=oneshot 的情况下， 才可以设置任意个命令行(包括零个)，否则必须且只能设置一个命令行。如果不设置任何ExecStart指令， 那么必须确保设置了RemainAfterExit=yes指令。

    `注：命令行必须以一个绝对路径表示的可执行文件开始，并且其后的那些参数将依次作为"argv[1] argv[2] ..."传递给被执行的进程。如果在绝对路径前加上可选的"@"前缀，那么其后的那些参数将依次作为"argv[0] argv[1] argv[2]..."传递给被执行的进程。如果在绝对路径前加上可选的"-"前缀，那么即使该进程以失败状态(例如非零的返回值或者出现异常)退出，也会被视为成功退出。如果在绝对路径前加上可选的 "+" 前缀，那么进程将拥有完全的权限(超级用户的特权)。可以同时使用"-","@","+"前缀，且顺序任意。`  
    `如果设置了多个命令行，那么这些命令行将以其在单元文件中出现的顺序依次执行。如果某个无 "-" 前缀的命令行执行失败，那么剩余的命令行将不会被继续执行，同时该单元将变为失败(failed)状态。`  
    `当未设置 Type=forking 时，这里设置的命令行所启动的进程 将被视为该服务的主守护进程。`
* ExecStartPre/ExecStartPost：设置在执行ExecStart之前/后执行的命令行。语法规则与ExecStart完全相同。  
    `注：只有当ExecStartPre中所有的不以"-"为前缀的命令全部执行成功后，才会执行ExecStart中的命令。只有ExecStart中所有的不以"-"为前缀的命令全部执行成功后，才会执行ExecStartPost中的命令。`
    `不可将ExecStartPre用于需要长时间执行的进程。因为所有由ExecStartPre派生的子进程都会在启动ExecStart服务进程之前被杀死。`
    `如果在服务启动完成之前，任意一个ExecStartPre,ExecStart,ExecStartPost中无"-"前缀的命令执行失败或超时，那么，ExecStopPost将会被继续执行，而ExecStop则会被跳过。`
* ExecStop：这是一个可选的选项，用于设置当该服务被要求停止时所执行的命令行。语法规则与ExecStart完全相同。执行完此处设置的命令行之后，该服务所有剩余的进程将会根据KillMode的设置被杀死如果未设置此选项，那么当此服务被停止时，该服务的所有进程都将会根据KillSignal的设置被立即全部杀死。
* ExecStopPost：这是一个可选的指令，用于设置在该服务停止之后所执行的命令行。语法规则与ExecStart完全相同。 此选项中设置的命令都会在服务停止后被无条件的执行。因此，应该将此选项用于设置那些无论服务是否启动成功都必须在服务停止后无条件执行的清理操作。此选项设置的命令必须能够正确处理由于服务启动失败而造成的各种残缺不全以及数据不一致的场景。由于此选项设置的命令在执行时，整个服务的所有进程都已经全部结束，所以无法与服务进行任何通信。
* RestartSec：设置在重启服务(Restart)前暂停多长时间。默认值是100毫秒(100ms)。
* TimeoutStartSec：设置该服务允许的最大启动时长。如果守护进程未能在限定的时长内发出"启动完毕"的信号，那么该服务将被视为启动失败，并会被关闭。设为 "infinity" 则表示永不超时。
* TimeoutStopSec：设置该服务允许的最大停止时长。如果该服务未能在限定的时长内成功停止，那么将会被强制使用 SIGTERM 信号关闭，如果依然未能在相同的时长内成功停止， 那么将会被强制使用 SIGKILL 信号关闭。设为 "infinity" 则表示永不超时。
* TimeoutSec：一个同时设置TimeoutStartSec与TimeoutStopSec的快捷方式。
* WorkingDirectory：设置进程的工作目录。
* User/Group：设置进程在执行时使用的用户与组。既可以设为数字形式的ID也可以设为字符串形式的名称。如果没有明确设置Group选项，则使用User所属的默认组。此选项不影响带有 "+" 前缀的命令。
* Environment：设置进程的环境变量。
* EnvironmentFile：与Environment类似，不同之处在于此选项是从文本文件中读取环境变量的设置。文件中的空行以及以分号或井号开头的行会被忽略，其他行的格式必须符合VAR=VALUE的shell变量赋值语法。行尾的反斜杠将被视为续行符。

## systemd的安装
systemd的安装（Install）包含单元的启用信息。事实上，systemd在运行时并不使用此小节。只有systemctlenable与disable命令在启用/停用单元时才会使用此小节。它有以下选项：
* Alias：启用时使用的别名，可以设为一个空格分隔的别名列表。
* WantedBy/RequiredBy：接受一个空格分隔的单元列表，表示当列表中的任意一个单元启动时，该单元都会被启动。可用作开机启动使用。

## 理论结合实践
### 简单的例子
使用python写一个死循环的小程序，作为服务程序，代码如下：
```python
#!/usr/bin/env python
# coding:utf-8

import time

f = open('test.log', 'w')
while True:
        f.write('test\n')
        f.flush()
        time.sleep(1)
f.close()

```
为该服务程序编写配置文件如下：
```
[Unit]
Description=Test service

[Service]
WorkingDirectory=/home/zhangmh/test/ser
ExecStart=/home/zhangmh/test/ser/test.py

[Install]
WantedBy=default.target
```
将文件保存在/lib/systemd/system/test.service中，即可使用旧的service命令来启动该服务，
也可以使用systemd的新命令来启动该服务：
```
systemctl start test
```
如果配置文件被修改过，需要使用如下命令systemd重新读取配置文件：
```
systemctl daemon-reload
```
启动后，可以查看服务的状态：
```
systemctl status test
```
显示的状态如下：
```
● test.service - Test service
   Loaded: loaded (/lib/systemd/system/test.service; disabled; vendor preset: enabled)
   Active: active (running) since 日 2017-02-05 10:29:58 CST; 4s ago
 Main PID: 4849 (python)
   CGroup: /system.slice/test.service
           └─4849 python /home/zhangmh/test/ser/test.py

2月 05 10:29:58 zhangmh-virtual-machine systemd[1]: Started Test service.
```
可以重启该服务：
```
systemctl restart test
```
再次查看状态如下：
```
● test.service - Test service
   Loaded: loaded (/lib/systemd/system/test.service; disabled; vendor preset: enabled)
   Active: active (running) since 日 2017-02-05 10:31:31 CST; 3s ago
 Main PID: 4871 (python)
   CGroup: /system.slice/test.service
           └─4871 python /home/zhangmh/test/ser/test.py

2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopped Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Started Test service.
```
停止该服务：
```
systemctl stop test
```
再次查看状态如下：
```
● test.service - Test service
   Loaded: loaded (/lib/systemd/system/test.service; disabled; vendor preset: enabled)
   Active: inactive (dead)

2月 05 10:29:58 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopped Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopped Test service.
```
还可以查看该服务的日志
```
journalctl -u test
```
显示的日志为：
```
2月 05 10:29:58 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopped Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopped Test service.
```
程序print的内容以及出错后的异常信息，都可以在日志中显示。如，将test.py改为如下代码：
```python
#!/usr/bin/env python
# coding:utf-8

import time

print 'test'
a = 1 / 0
f = open('test.log', 'w')
while True:
        f.write('test\n')
        f.flush()
        time.sleep(1)
f.close()

```
启动该服务，没有任何输出，查看服务的状态，显示：
```
● test.service - Test service
   Loaded: loaded (/lib/systemd/system/test.service; disabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since 日 2017-02-05 10:41:26 CST; 9s ago
  Process: 5002 ExecStart=/home/zhangmh/test/ser/test.py (code=exited, status=1/FAILURE)
 Main PID: 5002 (code=exited, status=1/FAILURE)

2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: test
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: Traceback (most recent call last):
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]:   File "/home/zhangmh/test/ser/test.py", line 8, in <module>
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]:     a = 1 / 0
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: ZeroDivisionError: integer division or modulo by zero
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Main process exited, code=exited, status=1/FAILURE
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Unit entered failed state.
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Failed with result 'exit-code'.
```
查看服务日志，显示：
```
2月 05 10:29:58 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Stopped Test service.
2月 05 10:31:31 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopping Test service...
2月 05 10:33:27 zhangmh-virtual-machine systemd[1]: Stopped Test service.
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: Started Test service.
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: test
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: Traceback (most recent call last):
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]:   File "/home/zhangmh/test/ser/test.py", line 8, in <module>
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]:     a = 1 / 0
2月 05 10:41:26 zhangmh-virtual-machine test.py[5002]: ZeroDivisionError: integer division or modulo by zero
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Main process exited, code=exited, status=1/FAILURE
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Unit entered failed state.
2月 05 10:41:26 zhangmh-virtual-machine systemd[1]: test.service: Failed with result 'exit-code'.
```

### 一个程序启动多个进程
写一个使用tornado的例子，可以一次启动多个进程监听不同端口，程序代码如下：
```python
#!/usr/bin/env python
# coding:utf-8

import sys
import multiprocessing
import tornado.web
import tornado.ioloop
import tornado.httpserver
from tornado.options import parse_command_line
from sessionRH import SessionRH

class HomeHandler(SessionRH):
    def get(self):
        self.write('hello.')

class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r"/",  HomeHandler),]

        settings = dict(
            cookie_secret = 'rewqr4gfd654fdsg@$%34dfs',
            session_expiry = 0,
            login_url = "/",
            debug = 1
        )

        tornado.web.Application.__init__(self, handlers, **settings)

def main(port):
    parse_command_line()
    http_server = tornado.httpserver.HTTPServer(Application(), xheaders=True)
    http_server.listen(port)
    tornado.ioloop.IOLoop.instance().start()

if __name__ == "__main__":

    if len(sys.argv) == 1:
        ports = [80]
    else:
        try:
            ports = [int(port) for port in sys.argv[1].split(',')]
        except:
            try:
                a, b = sys.argv[1].split('-')
                ports = range(int(a), int(b)+1)
            except:
                ports = list()
                print 'Parameter error.'

    lock = multiprocessing.Lock()
    for port in ports:
        p = multiprocessing.Process(target=main, args=(port,))
        p.start()
        print 'Web server id started on port %d.'%port
```
服务配置如下：
```
[Unit]
Description=Test Http Service

[Service]
WorkingDirectory=/home/zhangmh/test/ser
ExecStart=/home/zhangmh/test/ser/http.py 9001-9008

[Install]
WantedBy=default.target
```
启动服务后，查看状态：
```
● http.service - Test Http Service
   Loaded: loaded (/lib/systemd/system/http.service; disabled; vendor preset: enabled)
   Active: active (running) since 日 2017-02-05 11:20:20 CST; 3s ago
 Main PID: 5781 (python)
   CGroup: /system.slice/http.service
           ├─5781 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5784 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5785 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5786 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5787 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5788 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5789 python /home/zhangmh/test/ser/http.py 9001-9008
           ├─5790 python /home/zhangmh/test/ser/http.py 9001-9008
           └─5791 python /home/zhangmh/test/ser/http.py 9001-9008

2月 05 11:20:20 zhangmh-virtual-machine systemd[1]: Started Test Http Service.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9001.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9002.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9003.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9004.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9005.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9006.
2月 05 11:20:20 zhangmh-virtual-machine http.py[5781]: Web server id started on port 9007.
```
可以看到systemd可以自动监测到程序启动的进程，停止服务时，这些启动的进行将全部被杀死。

### 依赖其他单元
配置好nginx，使用nginx监听80端口，并使用反向代理机制将请求跳转到9001~9008端口。配置上述http服务依赖nginx单元：
```
[Unit]
Description=Test Http Service
Wants=nginx.service

[Service]
WorkingDirectory=/home/zhangmh/test/ser
ExecStart=/home/zhangmh/test/ser/http.py 9001-9008

[Install]
WantedBy=default.target
```
将nginx和http两个服务都处于停止状态后，启动http服务，查看两个服务的状态，可以发现两个服务都被启动。

### 开机启动
上述http服务的Install中设置了被default.target依赖，使用如下命令：
```
$ systemctl enable http
Created symlink from /etc/systemd/system/default.target.wants/http.service to /lib/systemd/system/http.service.
```
在/etc/systemd/system/default.target.wants中创建http.service的链接，可以将服务设置为开机启动，本质上，该服务会在系统启动default.target单元时启动该服务。  
如果要关闭开机启动，则可以使用如下命令：
```
systemctl disable http
```

### 使用扩展
既然systemd是整个系统的守护进程，那么它肯定可以控制系统的运行状态。如：
```
systemctl reboot # 重启
systemctl poweroff # 关机
systemctl suspend # 睡眠
systemctl hibernate # 休眠
```