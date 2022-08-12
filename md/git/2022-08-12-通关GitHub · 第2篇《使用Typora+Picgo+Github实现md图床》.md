# 使用Typora+Picgo+Github实现md图床



## 前言

这一篇对于我来说是一个很重要的一篇文章。

原因是，解决了困扰我多时的问题。

对于一个从事嵌入式这个高门槛行业的我来说，琐碎的知识点、繁多的代码、众多的问题、XX的任务（以我贫瘠的语文水准，真的找不到形容词了），对于我这个记忆力差，脑容量小的人来说，好记性不如烂笔头，就需要去记笔记。

用什么去记呢？

纸？呵呵，我一个程序员还手撸代码呢...

除了纸，那就剩正在被按下的电脑了。对于一个程序员来说，当然用电子去记笔记啦。通过网上搜寻，找到了这一款软件。

没错他叫**Typora**，一个感觉未来要完全收费的软件。不过说实话，它对于markdown语法的支持，哪怕它以后真要花钱了，我真得会花钱去使用它。因为！实在是很方便！

这里给大家一个链接去看看它到底有多好用 [6 年，为什么 Typora 是最好的 Markdown 编辑器？](https://baijiahao.baidu.com/s?id=1718539845829585696&wfr=spider&for=pc)

当然实际用的话没有百分百完美的软件。Typora文档内部无法保存图片，每次插入的图片，Typora内部也是在本地的一个路径下的文件夹做缓存，打开那个文件夹发现，之前插入的图片都在那里面，而markdown文档里只剩下了一个长长的链接。

如果这个时候只把这个文档给别人的话，图片多半就没了（人是什么时候没的都不晓得），总不能再把那些图片放在一个文件夹内，一起打包给别人，太不方便了吧——那和做ppt时往里面插入一段音频有什么区别？

我在使用时就一直困惑于typora怎样让远端的人当我把文档给他时，他也能够看到里面的图片呢?

别忘了，Typora里有图片上传功能，用Typora链接一个图片库里不就行了。说干就干。

------



## GitHub

### 1.创建GitHub图库

![image-20220812235138235](https://raw.githubusercontent.com/Tschome/image/main/img/git/202208122351309.png)

### 2.获取令牌

大致步骤如下：

* 点击自己的 GitHub 头像
* Settings
* Developer settings
* Personal access tokens
* Generate new token

之后会有这个界面，自己填选，Note注明给谁用，全部勾选

![img](https://raw.githubusercontent.com/Tschome/image/main/img/git/202208122357045.png)

生成token

此时可以保存备用，也可以在下边用到的时候，再按照上述步骤生成 token 。注意 token 是私密的，需要做好安全保护！

## Picgo

### 1.安装PicGo

GitHub下载地址：[Picgo](https://github.com/Molunerfinn/PicGo/releases)

### 2.配置

安装好picgo后，打开。

<img src="https://raw.githubusercontent.com/Tschome/image/main/img/git/202208130004553.png" alt="image-20220813000433494" style="zoom:67%;" />

接下来配置 GitHub 作为图床，在左侧找到图床设置，找到GitHub图床。前边有星号的为必填项，依次填入之前创建的

* 仓库名，注意是：账户名/仓库名；
* 分支名（创建仓库时如果没有创建其他分支，默认就是 master 分支）——我的是main
* 最后填入之前生成的 token 令牌，
* 指定路径（最好加上'/'，否则可能会设置失败哦）
* 点击确定。

配置就如此简单。接下来就可以使用啦



## Typora

### 1.软件下载

- Typora 官网：[Typora](http://typora.io/)
- PicGo 蓝奏云下载地址：[PicGo-Setup-2.2.2.exe](https://www.lanzous.com/ia49ojg)
- PicGo 在 GitHub 上的地址：[GitHub - PicGo](https://github.com/Molunerfinn/PicGo/releases)

### 2.配置图片上传

#### 1.打开Typora，点击菜单栏-文件-偏好设置

<img src="https://raw.githubusercontent.com/Tschome/image/main/img/git/202208130011070.png" alt="image-20220813001124975" style="zoom: 50%;" />

#### 2.配置插入图片和上传服务设置

##### 插入图片

选择为上传图片，且下面按需要选中。

##### 上传服务设定

上传服务选择PicGo（app）

PicGo路径，就设置为你电脑里安装PicGo的绝对路径，

设置完之后，不要忘记点击验证上传功能哦。



## 后叙

使用一段时间后，发现图床真的好用，不过对于写多个知识点博客的我来说，存在几个问题，与此同时，大家或多或少可能也会存在这个问题：

* 所有图片都放在一个库里
* 如果修改文档，删除图片，文档里的图片是删除了，但是**图床里（也就是GitHub里库）的图片是不会主动删除的！！！**

这几个问题导致，如果随意的插入图片的话，后续维护的成本非常的大。

我经过思考，我有下面比较折中的办法，使得图床的图片更好管理些

##### 1.插入图片前，打开PicGo配置GitHub的界面

将指定存储路径改为索要设定的文件夹内。如下图，当前所编写的文档是和git有关的内容，那我就把图片放到/image/git/路径下。

<img src="https://raw.githubusercontent.com/Tschome/image/main/img/git/202208130004553.png" alt="image-20220813000433494" style="zoom:67%;" />

如果是其他路径的话，直接在这里修改（库里没有这个文件夹，它会自动创建），

##### 2.点击确定，保存设置。

##### 3.点击PicGo设置，打开上传前重命名功能

<img src="https://raw.githubusercontent.com/Tschome/image/main/img/git/202208130031015.png" alt="image-20220813003136941" style="zoom: 67%;" />

##### 3.再在Typora中插入图片

==注意：针对不同类的文档类型选择不同的路径是每次插入图片前的必要操作，其他操作设置一次就行==

##### 4.修改文章，修改或图片时

首先去库里把相对应的图片删除，再将文档里的链接删除。这一步能够确保库里的图片都是有用的。

------

> 那么到这里，虽然操作麻烦了点，不过我也是会用图床的人啦，相信你看了这篇文章，对你有了更好的启发。
>
> 或许你有更好的想法，欢迎联系我，大家一起讨论~