---
layout: post
title: 'win10 家庭版安装 Docker 踩坑'
categories: 其他
tags: Docker
excerpt: '记录了在 win10 家庭版安装使用 Docker 的踩坑过程……'
---

这个入门过程真是有点坑……我要记录一下2333

前天想搞个 `Docker` 玩一下，官网下好了 `Docker Desktop for Windows` 的安装包，安装……

> 抱歉 `Docker` 不支持 win10家庭版……

我：……（从入门【还没】到放弃？？？）

……怎么可能呢！！
于是google之，发现是我自己的锅，官方文档其实已经写了系统要求和不满足的解决办法了，是我自己没看……= =  
![](http://120.77.171.203/assets/img/posts/2019-04/1.jpg)
https://docs.docker.com/docker-for-windows/install/

好吧，那就安装 `Docker Toolbox` 咯~  
安装文档：https://docs.docker.com/toolbox/toolbox_install_windows/  
按着文档走就好。

PS：如果电脑已经安装过 `Virtual Box`，列表就不要勾选这个。如果电脑已经装过 `Git`，也可以不用勾选。 
![](http://120.77.171.203/assets/img/posts/2019-04/2.jpg)

安装完之后就会多了两个快捷方式  
![](http://120.77.171.203/assets/img/posts/2019-04/3.jpg)

双击 `Docker Quickstart Terminal` 启动 `Docker`，这个时候我遇到了一个问题
![](http://120.77.171.203/assets/img/posts/2019-04/4.jpg)  
原因是我之前装过 `Git` 了，不是在默认路径的，所以现在它找不到 `bash.exe` 的路径。  
解决办法：[解决点击Docker出现windows 正在查找bash.exe。如果想亲自查找文件，请点击“浏览”的问题](https://blog.csdn.net/A632189007/article/details/78601213)

改完之后可以正常启动了，显示如下界面就是成功了
![](http://120.77.171.203/assets/img/posts/2019-04/5.jpg)
我们还可以执行以下 `docker version` 命令看看，有正常输出就说明没问题了。

啊好开心终于安装好了，接下来迫不及待试试第一个镜像~~  
```bash
docker run hello-world
```
`docker run` 这个命令如果你本地没有这个镜像，会先帮你 `pull` 下来，结果……  
下载速度非常的慢！！

emmm……没毛病

`Docker` 默认的镜像源 `Docker Hub` 访问比较慢，我们可以改成国内的镜像源。  
我用的是阿里云的，官方文档：[Docker 镜像加速器](https://yq.aliyun.com/articles/29941)。

```bash
# 启动Docker虚拟机【其实我们可以用这个命令而不通过 Docker Quickstart Terminal 来启动。如果已经启动了Docker，这个命令可以不用执行】
docker-machine start default
docker-machine ssh default # ssh连接docker虚拟机
sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=加速地址 |g" /var/lib/boot2docker/profile # 修改加速地址【注意要把加速地址改一下，阿里云的地址参考上面的官方文档获得】
exit # 退出docker虚拟机
docker-mechine restart default # 重启docker虚拟机
```

---

使用 `Docker Toolbox` 还有一个地方要注意的，默认的Virtual Box docker虚拟机创建在了C盘，那随着我们不断使用 `Docker` ，占用的空间就会变得很大，所以我们可以先改一下Virtual Box docker虚拟机的位置。
具体操作我直接引用另一篇文章（[在Windows中玩转Docker Toolbox(镜像加速)](https://blog.csdn.net/chengly0129/article/details/68947265)）了：

> #### Docker虚拟机文件地址修改
> 
> 默认情况下，docker-machine创建的虚拟机文件，是保存在C盘的C:\Users\用户名\.docker\machine\machines\default 目录下的，如果下载和使用的镜像过多，那么必然导致该文件夹膨胀过大，如果C盘比较吃紧，那么我们就得考虑把该虚拟机移到另一个盘上。
> 
> 具体操作如下：
> 1. 使用docker-machine stop default停掉Docker的虚拟机。
> 2. 打开VirtualBox，选择“管理”菜单下的“虚拟介质管理”，我们可以看到Docker虚拟机用的虚拟硬> 盘的文件disk。
> 3. 选中“disk”，然后点击菜单中的“复制”命令，根据向导，把当前的disk复制到另一个盘上面去。
> 4. 回到VirtualBox主界面，右键“default”这个虚拟机，选择“设置”命令，在弹出的窗口中选择“存> 储”选项。
> 5. 把disk从“控制器SATA”中删除，然后重新添加我们刚才复制到另外一个磁盘上的那个文件。
> 
> 这是我设置好后的界面，可以看到我在步骤3复制的时候，复制到E:\VirtualBox\default\dockerdisk.vdi文件去了。
> ![](http://120.77.171.203/assets/img/posts/2019-04/6.jpg)

---

最后还有一个小Tips：  
Docker容器是在VirtualBox的虚拟机里面，不是在Windows里面，所以如果要访问用Docker容器启动的网站的话要做端口映射，而且在本机【自己的windows】浏览器访问的时候要通过虚拟机ip（可以通过 `docker-machine env` 命令查看）访问。  
像这样：  
![](http://120.77.171.203/assets/img/posts/2019-04/7.jpg)
![](http://120.77.171.203/assets/img/posts/2019-04/8.jpg)

---

好啦到这里坑已经踩完了，可以愉快地使用 Docker 啦~~

---

### 参考文章：

[Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)  
[Install Docker Toolbox on Windows](https://docs.docker.com/toolbox/toolbox_install_windows/)  
[解决点击Docker出现windows 正在查找bash.exe。如果想亲自查找文件，请点击“浏览”的问题](https://blog.csdn.net/A632189007/article/details/78601213)  
[在Windows中玩转Docker Toolbox(镜像加速)](https://blog.csdn.net/chengly0129/article/details/68947265)  

