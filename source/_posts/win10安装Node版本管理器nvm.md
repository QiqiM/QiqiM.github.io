---
title:  win10安装Node版本管理器nvm
date: 2020-05-20 12:10:01
tags: [Node, nvm]
categories: Node
---

####  使用nvm的原因

开发公司项目和个人项目时，由于公司项目比较旧，Node版本比较低，但是自己做的项目安装的包，需要比较新的Node包，10以上的版本，所以就需要在同一台机器上安装多个版本的Node。经过搜索，决定使用`nvm`来做Node版本的管理。

#### 安装前需要先将之前安装的Node版本完全删除

+ Windows设置 --> 应用--> 找到Node-->点击卸载
+ 重启电脑（或者从任务管理器中杀死所有Node相关的进程）
+ 寻找以下文件夹并删除他们。根据您安装的版本，这些文件可能存在也可能不存在：
  + C:\Program Files (x86)\Nodejs
  + C:\Program Files\Nodejs
  + C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）
  + C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache）

+ 检查%PATH%环境变量，确保没有引用Nodejs和npm的存在
+ 重启（重启大法解决90%问题）

#### 下载安装

[nvm-window](https://github.com/coreybutler/nvm-windows/releases)

![20200520113014imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520113014-image.png)

nvm安装位置，看自己决定，但是安装路径不能有空格，比如`Program Files`

![20200520113215imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520113215-image.png)

安装的多版本Node放在哪里，也看个人喜好，可以修改

![20200520113257imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520113257-image.png)

#### 环境变量，安装好之后，环境变量会自动设置好

![20200520113431imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520113431-image.png)

#### 使用

在你的`nvm安装路径`下打开`cmd`或者`git bash`,在其他路径下打开，会报错`nvm： commond not found`（重启！重启！重启！）

> nvm -v    // 查看nvm版本，判断是否安装成功

![20200520113838imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520113838-image.png)

> nvm ls available        // 获取可获取的Node版本

![20200520114039imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520114039-image.png)

> nvm install 12.14.1      // 安装指定版本的Node

![20200520114149imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520114149-image.png)

> nvm use 12.14.1  // 使用指定版本Node

![20200520114316imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520114316-image.png)

> nvm list       // 列出本地已安装的Node版本

![20200520114406imagepng](https://raw.githubusercontent.com/QiqiM/yato-GitNote/master/20200520114406-image.png)

> nvm uninstall 10.15.3      // 卸载指定版本Node



#### 常用命令

| 命令                  | 功能                          |
| --------------------- | ----------------------------- |
| nvm -v                | 查看nvm版本，判断是否安装成功 |
| nvm ls available      | 获取可获取的Node版本          |
| nvm install 12.14.1   | 安装指定版本的Node            |
| nvm use 12.14.1       | 使用指定版本Node              |
| nvm uninstall 10.15.3 | 卸载指定版本Node              |

