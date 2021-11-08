# Xdebug

[toc]

## Debugger

为什么需要 debugger，在使用 debug 扩展工具前，之前的调试大部分都是使用的诸如`echo`/`print_r`/`var_dump`/`exit` 等。

在调试一些不熟悉的框架代码、注释不清晰的源码时就会非常棘手，这时就可以通过一些扩展的 debug 工具来进行类似于 Java 一样的步进调试或更复杂的单元测试等。

> **Xdebug** 是一个**开源**的 PHP **调试器**，可以用来跟踪、调试、分析 PHP 程序的运行情况。

## 安装

//TODO

### Windows 

#### 下载

- 查看 `phpinfo` 提供的 `zendframework` 版本信息，在 [xdebug 官网](http://www.xdebug.org/download.php) 找到对应版本的 dll 文件。
- 将下载好的 `php-xdebug.dll` 文件添加到 php 目录下的 `ext` 文件夹中

#### 配置

- 配置 `php.ini` 文件：

  ```ini
   [Xdebug]
  zend_extension=(dll 文件路径)   (Win)
  xdebug.profiler_enable=on
  xdebug.trace_output_dir="../Projects/xdebug"
  xdebug.profiler_output_dir="../Projects/xdebug"
  ```
  
- 重启 Server ， 查看 `phpinfo()` 是否输出 `Xdebug` 插件的相关信息即可知晓是否安装成功。

## Xdebug 调试原理

Xdebug 的工作原理可以总结为下面几个步骤

1. IDE（比如PHPStorm，下文所述的客户端）中已经集成了一个遵循 **BGDp** 协议（一个专门用来调试的协议）的Xdebug 插件。当要 debug 的时候，点击一些 IDE 的某个按钮，启动这个插件。该插件会启动一个 9000 的端口监听远程服务器发过来的debug信息。

2. 浏览器向 Httpd 服务器发送一个带有 **XDEBUG_SESSION_START** 参数的请求，Httpd 收到这个请求之后交给后端的 PHP 进行处理（下面就忽略 Httpd ，直接把 PHP 叫做 Server）。

3. PHP 看到这个请求是带了 XDEBUG_SESSION_START 参数，就告诉 Xdebug ，“嘿，我要 debug 喔，你准备一下”。这时， Xdebug 这时会向来源 ip 客户端的 9000 端口（即客户端，也即是 IDE ）发送一个 debug 请求，然后客户端的 9000 端口响应这个请求，那么 debug 就开始了。

4. PHP 知道 Xdebug 已经准备好了，那么就开始开始一行一行的执行代码，但是**每执行一行都会让 Xdebug 过滤一下**。

5. Xdebug 开始过滤代码，Xdebug 在过滤每一行代码的时候，都会暂停代码的执行，然后向客户端的9000端口发送该行代码的执行情况，等待客户端的决策（是一句代码还是下一个断点待）。

6. 相应，客户端（IDE）收到 Xdebug 发送过来的执行情况，就可以把这些信息展示给开发者看了，包括一些变量的值等。同时向 Xdebug 发送下一步应该什么。

> 参考资料：[Xdebug断点调试的工作原理详解](http://www.softown.cn/post/117.html)

## 搭配 IDE 使用 Xdebug

### PHPStorm 

#### 1. 修改 php.ini 配置 Xdebug 

```ini
[XDebug]
;xdebug 库
zend_extension = "php_xdebug.dll"
;开启远程调试
xdebug.remote_enable = On
;客户机 ip
xdebug.remote_host = "127.0.0.1"
;客户机 xdebug 监听端口和调试协议
xdebug.remote_port = 9001
xdebug.remote_handler = dbgp 
;idekey 区分大小写
xdebug.idekey = "PHPSTORM"
xdebug.profiler_enable = off
xdebug.profiler_enable_trigger = off
xdebug.profiler_output_name = cachegrind.out.%t.%p
xdebug.profiler_output_dir = "d:\tmp"
```

#### 2. PHPStorm 中的配置

1. Language & Framework -> PHP -> debug -> Xdebug: Debug port 设置为配置的端口。
2. 在以上设置的之下的 debug -> DBGp Proxy 中配置 idekey 、Host 与 Port 。
3. 在 PHP -> Servers 中手动创建一个本地调试服务器。
4. 在 PHP -> debug 中点击 Validate 测试配置是否成功。
5. 剩下配置好 Server，创建调试配置。
6. debug 运行 Server 就可以进行调试了。

## 在 PHP 8.0 与 Xdebug 3 的情况下配置

在配置 symfony debug 的时候访问，IDE 一直在 waiting for incoming connection 。

用老配置方式折磨了半天，最后发现原来是因为 Xdebug 3 的 php.ini 写法出现了一些变化。

### 新的配置方式

```ini
[XDebug]
zend_extension=dll
xdebug.mode=debug
xdebug.start_with_request=default
xdebug.client_host=localhost
xdebug.client_port=9000
xdebug.remote_handler=dbgp
xdebug.idekey=PHPSTORM
```

## 调试

### 调试方式

#### step into 步进

单步执行，遇到子函数就进入并且继续**单步执行**（简而言之，进入子函数）；

#### step over 步跃

在单步执行时，在函数内遇到子函数时不会进入子函数内单步执行，而是将子函数整个执行完再停止，也就是把子函数整个作为一步。有一点,经过我们简单的调试,在不存在子函数的情况下是和 step into 效果一样的（简而言之，**越过子函数，但子函数会执行**）。

#### step out 步退

单步执行到子函数内时，用 step out 就可以执行完子函数余下部分，并返**回到上一层函数**。

## Xdebug Profiler



> 参考：[Xdebug profiler 中文文档](http://xdebug.org.cn/docs/profiler)

