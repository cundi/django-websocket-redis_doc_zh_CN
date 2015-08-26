# django-websocket-redis-
目前翻译的文档是django-websocket-redis 0.4.4  

djang-websocket-redis 0.4.4
# Introduction 引言
Django和Ruby－on－Rails这样的已经开发出来的应用服务器并没有打算支持长连接。因此，这些框架不能够很好的适应web应用，这些应用可以由服务器构造的异步事件来重构。对于客户端期望的提醒来说，这里一个可行的解决方案
是持续的使用XMLHttpRquest（即Ajax）向服务器轮询。可是这样做会产生很多的流量，而且其做法也以依赖于内部轮训的间隔，对于聊天室这样的实时事件或者基于浏览器对多用户游戏来说这是不可行的解决方案。  

用Python编写的Web应用通畅使用WSGI做为自己和web服务器之间的通讯层。而WSGI是一个无状态协议，它定义了如何使用一种抽象自HTTP协议的简单方法来处理请求，并作出响应，然而设计的这个协议并不支持非阻塞请求。  

## The WSGI protocal can not support websockets WSGI协议不支持websoket
Django中，web服务器接受进来的请求，然后设置一个之后会被传送到应用服务器的WSGI字典。HTTP头部及其所装载的内容只要被创建，稍后请求就会立刻完成，并流送到客服端。这个处理过程通常需要大约12毫秒。吞吐量是一个服务器可以处理的平均响应时间，乘以并发工作的数量。每个工作的都有自己的线程／进程，

对于要准备处理这个流程的来说，在WSGI协议的技术规范顶层添加webscoket这样的长周期连接几乎是不可能的。因此大多数对webscoket实现都是是走的另外一条路。websocket的连接通过默认的应用服务器的端到端运行服务来控制。这里，web服务器支持通过来自客户端的派送请求达到对长周期连接的支持。  

能够过派送websoket请求的web服务器是NGiNX服务器。常规的请求被发送到使用WSGI协议的Django，而长活动的websocket连接则被传送到了一个只负责响应websocket请求的特殊服务。  

典型的实现提议是使用运行在一个NoDEJS内部的`socket.io`循环。  

图片：略

这里，**Django**使用RESTful API与**Node.JS**通信。可是这样操作却难以维护，因此它将两种不同的技术放在了一起。可供选择的建议是使用其他的基于Python的异步时间框架，比如Torando。不过这些方法看上去都像是临时替代性的解决方案，因为人们不得不同时使用Django和第二个框架。这样做导致项目依赖于另外的基础应用，而且难以维护。此外，你去运行两个同时运行的框架在应用开发中会显得很尴尬，特别是调试代码的时候。  

## uWSGI
在搜索简单一点儿的方法时，我发现了开箱即用的`uWSGI offers websockets`。利用`Redis`做为消息队列，再加少许几行Python代码，就可以让任何一个基于WSGI的框架做到双向通信，比如，Django。当然，这里也是禁止对每个打开的websocket连接创建一个新的线程。因此，在协作并发模式中使用出色的gevent和greenlet库，对于所有处开放的连接来说，部分代码是运行在一个独立的线程或者进程当中的。  

该方法的一些优点如下：  

- 易于实现。
- 异步I/O循环处理的websocket可以运行在下类情况： 
    * 在Django中使用 `./manage.py runserver`可以获得完整的调试控制。
    * 使用uWSGI，做为一个独立的HTTP服务器。
    * 在两个分离的循环中使用NGiNX活着Apache（版本大于等于2.4）做为前置代理，

## 使用Redis作为消息队列
这里可能的一个争议就是这些操作实现即来一点也不简单，因为你得有一个额外的服务——即Redis数据服务器——而且必须同时和Django一起运行。Websocket是双向的但是它们一般的应用场景是在客户端上触发服务器初始化事件。尽管，其他的方向也是可用的，不过使用Ajax来处理会特别容易一些——即添加一个额外的TCP／IP握手。  

这里，仅有的“与客户端保持联系”是附件到websocket的文件描述器。而且因为我们要说上几千次的打开连接，内存周期中的所占的空间和CPU资源都必须缩减到最小。就该实现操作来说，仅有一个打开的文件处理每次开放都websocket连接。  

生产服务器是无论如何都要求使用某种会话存储的。它可以是memcached活着一个Redis数据服务器。因此，这类服务是无论如何都必须运行的，假如我们选择了他们的其中的一个，我们应该使用一个支持集成消息队列的会话存储。而在使用Redis作为缓存和会话存储时，我们几乎可以不受限制的获取消息队列。  

### Scalability 扩展性
Redis中的一个非常好的功能就是它的无限扩展性。假如一个Redis服务器不能扣处理自身的作业，

-----------------------------------------
# Installation and Configuration 安装和配置
## Installation 安装
如果还没有完成**Redis server**的安装，可以使用系统提供的安装工具来安装，比如aptitude，yum，port或者从[从源安装Redis](http://redis.io.download)。  

在你的主机上启动Redis服务  

```python
$ sudo service redis-server start
```

接下来检查Redis是否启动，并能够接受连接  

```python
$ redis-cli ping
PONG
```

### Dependencies 依赖

- [Django](http://djangoproject.com/)>=1.5
- redis >=2.10.3([Redis的Python客户端](https://pypi.python.org./pypi/redis))
- [uWSGI](http://projects.unbit.it/uwsgi/)>=1.9.20
- [gevent](https://pypi.python.org/pypi/gevent)>=1.0.1
- [greenlet](https://pypi.python.org./pypi/greenlet)>=0.4.5
- 建议的可选的应用:[wsaccel](https://pypi.python.org/pypi/wsaccel)>=0.6

## Configuration 配置
添加“ws4redis”到项目设置中的INSTALLED_APPS：

```python
INSTALLED_APPS = (
    ...
    'ws4redis',
    ...
)
```

指定websocket连接URL和普通请求URL之间的区别。  

```python
WEBSOCKET_URL = '/ws/'
```

假如Redis数据存储使用了连接设置而不是默认的设置，那么可以使用下面这个字典来重写这些值。  

```python
WS4REDIS_CONNECTION = {
    'host': 'redis.example.com',
    'port': 16379,
    'db': 17,
    'password': 'verysecret',
}
```

### Note 注释
仅需指定与默认值只不同的值。  

**Websocket for Redis**能够用WS4PREDIS_EXPIRE，此外还能够固定消息队列所发布的消息。这一点，有利于客户端应该能够在重新连接webscoket之后访问发布过的信息的情况，例如页面重新载入之后。  

此命令在几秒钟之内设置数字，每个收到的消息都通过Redis来保存，以及发布消息队列。  

```python
WS4REDIS_EXPIRE = 7200
```

**Webscoket for Redis**能够使用字符串对数据存储中的每个条目假如前缀。默认，这个前缀是空的。假如相同的Redis连接用来存储欠他类型的数据，为了避免名称的冲突，我们鼓励你对这些条目使用一个唯一字符串。  

```python
WS4REDIS_PREFIX = 'ws'
```

你可以利用一个定制的类来重写 `ws4redis.store.RedisStore`，当你需要的该类另外一种实现时。  

```python
WS4REDIS_SUBSCRIBER = 'myapp.redis_store.Redis-Subscriber'
```

该命令在开发和忽略生产环境的情况下要求使用。它重写了Django的内部主循环，并在request处理器的前面添加一个URL调度程序。  

```python
WSGI_APPLICATION = 'ws4redis.django_runserver.application'
```

确保你的模板上下文中至少包含下面这些处理器：  

```python
TEMPLATE_CONTEXT_PROCESSORS = (
    ...
    'django.contrib.auth.context_processors.auth',
    'django.core.context_processors.static',
    'ws4redis.context_processors.default',
    ...
)
```

### Check your Installation 安装的检查

## Replace memcached with Redis 使用Redis替换memcached

### Warning 警告

# Running WebSocket for Redis

## Django在开发模式中使用WebSocket for Redis

### Note 注释

## Django使用WebSocket for Redis作为独立的uWSGI服务器

在此配置中，**uWSGI**有自己的主循环。为了区别常规请求与WebSocket，可以修改Python启动器模块，wsgi.py为：  

```python
import os
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myapp.settings')
from django.conf import settings
from django.core.wsgi import get_wsgi_application
from ws4redis.uwsgi_runserver import uWSGIWebsocketServer

_django_app = get_wsgi_application()
_websocket_app = uWSGIWebsocketServer()

def application(environ, start_response):
    if environ.get('PATH_INFO').startswith(settings.WEBSOCKET_URL):
            return _websocket_app(environ, start_response)
    return _django_app(environ, start_response)
```

你可以利用下面的命令运行uWSGI作为一个独立的服务器：  

```python
uwsgi --virtualenv /path/to/virtaulenv --http :80 --gevent 100 --http-websockets --module wsgi
```

服务器会应答在80端口的Django和WebSocket的HTTP请求。这里，修改过的应用所调度动进入请求依赖于URL和Django处理器两者的任意一个，要么是进入到WebScoket的主循环中。  

该配置配置适用于测试uWSGI和低流量的网站。因为uWSGI运行在一个线程／进程中，它会阻塞诸如访问数据库这类的调用，而且也会阻塞所有其他的HTTP请求。。。。

### 提供静态文件服务
在这个配置中，你不能够提供静态文件服务，因为Django不能够运行在调试模式，而且uWSGI也不知道如何服务于你所发布的静态文件。因此，在urls.py中添加模式staticfiles_urlpatterns：  

```python
from django.conf.urls import url, patterns, include
from django.contrib.staticfiles.urls import staticfiles_urlpatterns

urlpatterns = patterns('',
    ....
) + staticfiles_urlpatterns()
```

### Note 注释
，一如下结所诉，不要忘了在升级为更具有扩展能力的配置时移除staticfiels_urlpatterns。  
