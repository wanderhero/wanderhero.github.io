---
title: IntelliJ IDEA For Mac 导入导出快捷键配置  
categories: [IDE,IntelliJ IDEA]  
tags: [IDE,IntelliJ IDEA,快捷键,配置]  
date: 2017-05-05 14:03:18  
---
- 点击顶部的 **File**，选择 **Export Settings...**；
[![](http://wander.u.qiniudn.com/20170505-1403-0.0.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-0.0.png?imageslim)

- 默认选择的应该就是**当前 IntelliJ IDEA 版本所在的配置路径**；  
[![](http://wander.u.qiniudn.com/20170505-1403-1.0.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-1.0.png?imageslim)  
[不对看这里]如果不是，就选择 **...** 即更多文件，选择 **Recent Flaces** 即最近使用，选择你所使用的 IntelliJIdea 版本；  
[![](http://wander.u.qiniudn.com/20170505-1403-1.1.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-1.1.png?imageslim)
- 点击 **OK**，如果是第一次 Export，会提示 **Export Complete**，此时不要急着 **Cancel**，而是选择 Reveal in Finder；  
[![](http://wander.u.qiniudn.com/20170505-1403-2.1.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-2.1.png?imageslim)  
[不对看这里]如果不是第一次，则会提示 **File Already Exists**，其实就是该文件已存在，无脑点击 **OK** 即可，后面就是老司机上路了；
[![](http://wander.u.qiniudn.com/20170505-1403-2.0.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-2.0.png?imageslim)

- 在打开的文件里找到 **keymaps** 目录，你想要的 **快捷键配置** 文件就静静地躺在那里，enjoy it 吧~
[![](http://wander.u.qiniudn.com/20170505-1403-3.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-3.png?imageslim)
<!-- more -->
***
### 前言  
&emsp;&emsp;虽然用的最习惯的还是 eclipse，尤其是 MyEclipse，但由于 JetBrains 成功的商业模式，把各个工具的操作都统一了，大势所趋，IntelliJ IDEA 成了新宠。我还不是很熟悉这个新伙伴，今天遇到要把快捷键设置导出的问题就一下子难到了。本以为谷歌在手，天下我有，结果搜了一通，要么是 Windows 版的，要么就是风马牛不相及的快捷键使用说明，竟然没有我大 Mac 的文章，那就让我来献个丑吧~

### 正文
&emsp;&emsp;前面提到，我已经找到了 [Windows](#参考1) 的方法，那举一反三，Mac 上的位置呼之欲出。  
&emsp;&emsp;Windows 下IDEA默认快捷键的配置文件所在地（目录前缀自己改）：
C:\Program Files (x86)\JetBrains\IntelliJ IDEA xxx\lib\resources.jar\idea\KeyMap_***.xml，显然得，Mac 下肯定在 **IntelliJ IDEA.app** 的 **包内容** 里，**resources.jar** 包是顺利找到了，但通过 **JD-GUI** 查看了下，可能是版本的问题， **idea** 目录下并没有和 **KeyMap** 有关的文件，而是在同级的 **keymaps** 目录下，和 **偏好设置** 里的 **Keymap** 的设置是一样的，说明 默认快捷键的配置 找对了。  
[![](http://wander.u.qiniudn.com/20170505-1403-4.0.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-4.0.png?imageslim)  
[![](http://wander.u.qiniudn.com/20170505-1403-4.1.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-4.1.png?imageslim)  
[![](http://wander.u.qiniudn.com/20170505-1403-4.2.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-4.2.png?imageslim)  
[![](http://wander.u.qiniudn.com/20170505-1403-4.3.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170505-1403-4.3.png?imageslim)

问题来了，我自定义的在哪里
### 喜欢的快捷键要好好珍惜
快捷键修改
自己的别人的
### 吐槽一下

### 参考 
- <span id="参考1">[jiangfuqiang的专栏 - 完整导出IntelliJ IDEA的快捷键](http://blog.csdn.net/jiangfuqiang/article/details/38553011)，其他几篇跟博主一模一样的文章应该都是 copy 的</span>  
- [极客学院的IntelliJ IDEA 相关核心文件和目录介绍之Mac 的配置文件保存路径](http://wiki.jikexueyuan.com/project/intellij-idea-tutorial/installation-directory-introduce.html)  
- [Color Themes - 多彩主题](http://color-themes.com/?view=index)

快捷键修改
{% note danger %} **版权声明：** 本文章转载请注明出处！ {% endnote %}