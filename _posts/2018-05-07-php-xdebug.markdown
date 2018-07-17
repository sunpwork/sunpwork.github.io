---
layout:       post
title:        "PHP调试 使用PHPstorm xdebug调试"
subtitle:     ""
date:         2018-05-07 19:50:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - PHP
    - PHPstorm
    - xdebug
---

>在开发过程中，如果不能进行断点调试，那真的是很痛苦，今天我们就来介绍一下如何使用PHPstorm进行本地调试。

想要PHP进行调试，必须要安装php的xdebug插件，高版本的wampServer已经集成了这个插件，插件路径`C:\wamp\bin\php\php7.0.10\zend_ext\php_xdebug-2.4.1-7.0-vc14.dll`

如果没有，在网上下载一个放到里面就可以了。

然后我们只要在php.ini中配置一下就ok了，php.ini的所在路径如下：`C:\wamp\bin\php\php7.0.10\phpForApache.ini`
这个目录因wamp安装的路径和PHP的版本而异，我们打开PHP.ini，在文档的最后插入以下内容(新的版本已经加入)：
```ini
[xdebug]
zend_extension="C:\wamp\bin\php\php7.0.10\zend_ext\php_xdebug-2.4.1-7.0-vc14.dll" #本地php xdebug扩展包的路径
xdebug.remote_enable = On #开启远程调试
xdebug.profiler_enable = On
xdebug.remote_mode="req" 
xdebug.profiler_enable_trigger = On
xdebug.profiler_output_name =cachegrind.out.%t.%p
xdebug.profiler_output_dir="C:\wamp\tmp" 
xdebug.remote_host=127.0.0.1 #远程调试host，这里为本机
xdebug.show_local_vars=0
xdebug.idekey="PHPSTORM"
xdebug.remote_port=9000 #调试监听端口
xdebug.remote_handler=dbgp
xdebug.collect_vars = On
xdebug.collect_return = On
xdebug.collect_params = On
```

设置完毕之后点击右下角的wamp按钮，选择 Restart All Services 这样配置就生效了

这时候我们在index.php中写入如下：
```php
<?php

phpinfo();
```

运行后看到如下内容就说明xdebug已经配置成功了

![phpinfo](/img/in-post/phpstorm_xdebug/phpinfo.jpg)

下面我们在PHPstorm中配置xdebug环境

打开 File-Setting，首先我们先将本地的PHP配置同步到PHPstorm

![phpsetting](/img/in-post/phpstorm_xdebug/phpsetting.jpg)

然后我们来配置xdebug相关参数

![phpsettingDebug](/img/in-post/phpstorm_xdebug/phpsettingDebug.jpg)

然后我们来配置xdebug相关参数

![DBGp Proxy](/img/in-post/phpstorm_xdebug/DBGp Proxy.jpg)

这里面配置的内容应该和php.ini配置的相对应
```ini
[xdebug]
zend_extension="C:\wamp\bin\php\php7.0.10\zend_ext\php_xdebug-2.4.1-7.0-vc14.dll" #本地php xdebug扩展包的路径
xdebug.remote_enable = On #开启远程调试
xdebug.profiler_enable = On
xdebug.remote_mode="req" 
xdebug.profiler_enable_trigger = On
xdebug.profiler_output_name =cachegrind.out.%t.%p
xdebug.profiler_output_dir="C:\wamp\tmp" 
xdebug.remote_host=127.0.0.1 #远程调试host，这里为本机
xdebug.show_local_vars=0
xdebug.idekey="PHPSTORM"
xdebug.remote_port=9000 #调试监听端口
xdebug.remote_handler=dbgp
xdebug.collect_vars = On
xdebug.collect_return = On
xdebug.collect_params = On
```

配置本地Servers

![settingServers](/img/in-post/phpstorm_xdebug/settingServers.jpg)

最后，我们来配置本地 PHP Web Application，页面右上角编辑配置

![edit configurations](/img/in-post/phpstorm_xdebug/editconfigurations.jpg)

添加新的PHP Web Application

![pagewebapplication](/img/in-post/phpstorm_xdebug/pagewebapplication.jpg)

选择之前设置的Servers

![chooseServers](/img/in-post/phpstorm_xdebug/chooseServers.jpg)

最后我们在index.php中随意写一段代码：

![test](/img/in-post/phpstorm_xdebug/test.jpg)

点击右上角的debug按钮就可以看到效果了：

![debug-success](/img/in-post/phpstorm_xdebug/debug-success.jpg)