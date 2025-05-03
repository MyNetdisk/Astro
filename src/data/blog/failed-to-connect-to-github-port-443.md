---
author: Axton
pubDatetime: 2024-01-27T15:22:00Z
modDatetime: 2024-01-27T15:22:00Z
title: Failed to connect to github.com port 443
featured: false
draft: false
tags:
  - Git
  - GitHub
description:
   unable to access 'http://github.com/******'
---

今天想把自己有关文件格式转换的Python脚本上传到github上，但是无奈遇到报错：

>fatal: unable to access 'http://github.com/******': Fai
led to connect to github.com port 443 after 21051 ms: Couldn't connect to server

这是由于本机系统代理端口和git端口不一致导致的。

解决办法：
一、查看自己本机系统代理：

>设置---网络和Internet---代理---地址:端口

![默认样式截图](@/assets/images/eaa20bb54eeb43da97e35f04370ea36f.png)

二、修改git配置：（其中的10809改为你电脑的端口号）
```sh
git config --global http.proxy http://127.0.0.1:10809
git config --global https.proxy http://127.0.0.1:10809
```

三、再次push就可以成功上传。

![默认样式截图](@/assets/images/8ad0182d08594888840880c92794338f.png)

>版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。                     
原文链接：https://blog.csdn.net/m0_64007201/article/details/129628363
