### 微信登陆代理

微信桌面版做的很弱，并且没有Linux版。

虽然在Linux上可以使用微信web版，有时很难在一堆浏览器标签中找到。

像这样的应用，应该有属于自己的程序窗口与桌面空间。



##### 微信登陆代理实现目标

提供后台服务，管理微信登陆会话，负责与服务器通信。

提供双向的接收消息与发送消息功能。

以后台服务的方式运行，占用资源少，并且能够长时间运行，避免了需要经常手机扫描登陆的麻烦。

不过，由于这个代理提供的消息服务不再有认证等安全功能，登陆代理最好安装在安全的机器上，像本机上，或者是内网的私有服务器上。



##### 微信登陆代理原理

提到实现，不免要涉及到微信通信协议，好在已经有人对微信web版协议做了分析，虽然还有不完善的地方，基本上可以实现简单登陆与消息收发功能了。

微信登陆代理根据现有资料，使用PyQt5实现了微信web版协议，通过测试，把相关API更新到了最新的微信的wx2版本。

并且把二维码以dbus服务方式提供出来，这样该服务就不需要有UI的支持了，这也是与现有几个Linux版微信不同的地方。

除了以DBus方式提供API，还可以使用socket方式，可以更灵活了。

微信登陆代理实际上非常像ssh-agent的工作模式，长时间保持住会话，而不受UI交互客户端的启动或者退出状态的影响。

因此，把wxagent以后台服务的方式运行，即使Linux桌面偶尔出现崩溃也不用怕了，登陆进桌面后不需要再扫描QR重新登陆。

另外wxagent在启动机制上，使用了systemd --user方式，即把wxagent当作一个用户级服务。

这样有两个好处，启动不再需要root权限，并且能够很好的实现多用户多实例隔离支持。



##### 微信登陆代理程序及客户端

wxagent: 使用PyQt5实现的wxagent后台服务程序。

``` 
cd /path/to/wxagent
python setup.py install
/usr/bin/wxagent

在有包管理的系统上，可以使用systemd启动服务：
systemd --user start wxagent.service
```

wxaui: 使用PyQt5实现的简易wxagent客户端，显示微信登陆二维码，接收消息（消息未分类），处理部分接收的图片。选中用户，发送消息。

``` 
  /usr/bin/wxaui
```

wx2tox: 使用PyQt5实现的wxagent客户端，并将收到的消息分类转发到qTox IM客户端，以qTox(toxcore)群形式表示微信聊天会话。

``` 
  /usr/bin/wx2tox
```

##### 微信登陆代理模块



- 服务端
- 客户端
- dbus服务方法
- dbus总线事件

##### 微信登陆代理提供的服务API

wxagent提供的dbus服务方法： dbus://io.qtc.wxagent



islogined: sync方法, 参数表，无

``` 
返回值：bool
```

getqrpic: sync方法，参数表，无

``` 
返回值：image/jpeg data, base64 encoded
```

refresh: sync方法，参数表，无

``` 
返回值：bool
```

logout: sync方法，参数表，无

``` 
返回值：bool
```

getinitdata: sync方法，参数表，无

``` 
返回值：json string
```

getcontact: sync方法，参数表，无

``` 
返回值：json string
```

sendmessage: sync方法，

``` 
参数表：(from_username str, to_username str, content str, msgtype int)
返回值：json string
```

dumpinfo: sync方法，参数表，无

geturl: async方法，参数表，(url str)



wxagent提供的dbus事件信号： dbus://io.qtc.wxagent.signals

logined: 参数，bool

logouted: 参数，bool

newmessage: 参数，json string



##### 微信web版协议

[weixin web protocol.md(v2)](https://github.com/kitech/wxagent/blob/master/doc/protocolv2.md)

[weixin web protocol.md(v1)](https://github.com/kitech/wxagent/blob/master/doc/protocol.md)

ubuntu 12.04 64 位安装介绍 doc/build.md