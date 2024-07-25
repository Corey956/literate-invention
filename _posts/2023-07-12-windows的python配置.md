---
layout: mypost
title: Windows的python配置
categories: [Windows]
extMath: true

---

# 下载安装包

Python下载官网：https://www.python.org/

如需下载 python3.8.5 可直接跳转：https://www.python.org/downloads/release/python-385/

根据自己的版本选择下载

## 安装 Python 服务（目录需要英文目录，不能出现中文）

下载完成之后双击安装，注意：安装时自定义目录需要英文，不能包含中文，可能出现问题

双击安装exe文件

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091151090.png)

- 直接下一步

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152278.png)

- 选择安装路径

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152353.png)

- 点击`install`安装即可；等待安装完成点击`Close`关闭，然后配置环境变量即可；

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152462.png)

## 配置 Python 环境变量

> python环境变量的配置方法：首先鼠标右键此电脑–>选择属性–>然后点击高级系统设置–>点击环境变量；接着点击path进行编辑，在path中添加上python的安装路径；最后点击确定

![python环境变量如何配置](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152499.jpeg)

> 本教程操作环境：windows10/11系统、python3.8.5，Lenovo笔记本电脑。

**1、第一步：在我们的电脑上鼠标右键此电脑–>选择属性–>进去之后，点击高级系统设置，如下图所示：**

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152538.png)

**2、第二步：进去之后，点击环境变量，如下图所示：**

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152570.png)

**3、第三步：在系统变量中有Path则点击Path选择编辑，没有Path，选择新建，变量名写为Path，变量值写为python.exe的路径；**

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152690.png)

- 新建一个环境变量

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152705.png)

如果有，就点击Path，点击编辑：如果在一行上，就在变量值后面紧跟`;python`路径，如果是一行一行的那种就添加一行就行；

- 追加一个python的环境变量

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152761.png)

> 这样就配置完成了，配置完成之后都点击确定；

## 校验是否配置成功

在windows上，使用快捷键 `win+r`，调出cmd窗口；

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152802.png)

执行python，如果报错大概是配置路径没有写对

正确的是这样的

![在这里插入图片描述](https://bear-iot-c-test.oss-cn-shenzhen.aliyuncs.com/biji/202401091152827.png)

这样，就可以确认Python环境配置正确了