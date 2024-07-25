---
layout: mypost
title: php-fpm以root身份运行
categories: [Linux]
extMath: true

---

# php-fpm以root身份运行

#### 我们执行下面命令看一下其当前运行状态：

```bash
ps -aux | grep php-fpm
```

其运行用户为 www

![image-20231026151451052](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202310261515329.png)

我们通过修改php-fpm.conf来修改其运行身份

1. 我们首先找到php-fpm.conf

```bash
sudo find / -name php-fpm.conf  
```

![image-20231027093502979](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202310270935165.png)

2. 打开该文件

```bash
vim pathtophp-fpm.conf
```

3. 修改其中的user和group为root

4. 重新启动php-fpm

```bash
killall php-fpm
php-fpm -R
```