title: (转)搭建Git服务器及设置public key
date: 2015-02-26 10:03:16
tags: ツールインストール
---

<h2>搭建Git服务器</h2>

在远程仓库一节中，我们讲了远程仓库实际上和本地仓库没啥不同，纯粹为了7x24小时开机并交换大家的修改。

GitHub就是一个免费托管开源代码的远程仓库。但是对于某些视源代码如生命的商业公司来说，既不想公开源代码，又舍不得给GitHub交保护费，那就只能自己搭建一台Git服务器作为私有仓库使用。

搭建Git服务器需要准备一台运行Linux的机器，强烈推荐用Ubuntu或Debian，这样，通过几条简单的apt命令就可以完成安装。

假设你已经有sudo权限的用户账号，下面，正式开始安装。

第一步，安装git：

$ sudo apt-get install git
第二步，创建一个git用户，用来运行git服务：

$ sudo adduser git
第三步，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。

第四步，初始化Git仓库：

先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：

$ sudo git init --bare sample.git
Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：

$ sudo chown -R git:git sample.git
第五步，禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：

git:x:1001:1001:,,,:/home/git:/bin/bash
改为：

git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

第六步，克隆远程仓库：

现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：

$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
剩下的推送就简单了。

管理公钥
如果团队很小，把每个人的公钥收集起来放到服务器的/home/git/.ssh/authorized_keys文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用Gitosis来管理公钥。

这里我们不介绍怎么玩Gitosis了，几百号人的团队基本都在500强了，相信找个高水平的Linux管理员问题不大。

管理权限
有很多不但视源代码如生命，而且视员工为窃贼的公司，会在版本控制系统里设置一套完善的权限控制，每个人是否有读写权限会精确到每个分支甚至每个目录下。因为Git是为Linux源代码托管而开发的，所以Git也继承了开源社区的精神，不支持权限控制。不过，因为Git支持钩子（hook），所以，可以在服务器端编写一系列脚本来控制提交等操作，达到权限控制的目的。Gitolite就是这个工具。

这里我们也不介绍Gitolite了，不要把有限的生命浪费到权限斗争中。

小结
搭建Git服务器非常简单，通常10分钟即可完成；

要方便管理公钥，用Gitosis；

要像SVN那样变态地控制权限，用Gitolite。

### windows下git库的ssh连接，使用public key的方法
在windows下进行项目开发，使用git，通过ssh方式与git库连接，而ssh方式用public key实现连接。

首先需要下载mygit，安装后使用git bash。git bash（有GUI界面，如安装TortoiseGit后就可以使用）。我主要使用命令行，其命令行系统类似linux环境的基本操作命令，可以直接看到c:，如cd /d/mygitwork，进入我的D盘下的工程目录，放置开发的工程。

 

我的操作：在windows的git bash中用ssh -keygen ssh -keygen -t rsa生成了密钥对，cp .ssh/id_rsa.pub authorized_keys 改名。 将这个key交给同事，他作为github上的库创建者，添加到工程git库中，我clone该git库后，就可以使提交代码了，具体步骤：

如果已经用自己邮箱生成过ssh key，可以省去步骤1和2
1、生成ssh-key
ssh-keygen -t rsa -C "XXX@company.com"

2、重命名ssh-key
cp ~/.ssh/id_rsa.pub ~/.ssh/XXX@company.com.pub

3、发送邮件给git库负责人，由其添加到git库中，获得使用权限，将~/.ssh/xxx@company.com.pub放入邮件附件


 

与git库连接：ssh方式要利用public key实现写功能。

（一般公司会搭建自己的git服务器，如果是自己，可以使用免费的git 服务器github。具体的使用，在后面介绍）

git库建好后，用git clone连接，但这样的库，只有读功能，没有写功能。如果想写，必须用ssh方式，连接后，提交自己的public key，由该库的管理员将该public key添加到git库，产生访问权限。

public key 的原理在这里有介绍http://hi.baidu.com/beijiqieys/item/4643900f6ae51223a0312dc8

具体的命令是用ssh -keygen -t rsa生成密钥对，在客户端上创建一对公私钥 （公钥文件：~/.ssh/id_rsa.pub； 私钥文件：~/.ssh/id_rsa）
然后把公钥放到服务器上（~/.ssh/authorized_keys）, 自己保留好私钥.在使用ssh登录时,ssh程序会发送私钥去和服务器上的公钥做匹配.如果匹配成功就可以登录了。

将公钥文件复制到远程服务器上：

$ scp ~/.ssh/id_dsa.pub user@remote.host:pubkey.txt
$ ssh user@remote.host
$ mkdir ~/.ssh
$ chmod 700 .ssh
$ cat pubkey.txt >> ~/.ssh/authorized_keys
$ rm ~/pubkey.txt
$ chmod 600 ~/.ssh/*
$ exit

//权限的设置非常重要,因为不安全的设置安全设置,会让你不能使用RSA功能。

cat .ssh/id_rsa.pub | ssh user_B@your_ip "cat - >> /root/.ssh/authorized_keys"

也可以用

           ssh-keygen     #生成证书。

           ssh-copy-id -i id-rsa.pub 用户@ip    #把证书传到远程的那个机器上 并 生成authorized_keys文件。

 

github上的库创建：

创建Github Repository，注册Github账户(https://github.com/)，在GitHub，一个项目对应唯一的Git版本库，创建一个新的版本库就是创建一个新的项目。访问仪表板（Dashboard）页面，如下图所示，可以看到关注的版本库中已经有一个，但自己的版本库为零。在显示为零的版本库列表面板中有一个按钮“New Repository”，点击该按钮开始创建新版本库。

我们为新建立的版本库命名为“kxt-example”，相应的项目名亦为“ kxt-example ”，创建完毕后访问项目页，提示版本库尚未初始化，并给出如何初始化版本库的帮助，如下图所示(由于我的kxt-example已经初始化过了，所以下面的图片是截另一个未初始化的项目)。务必要 set up git，这个官网已经讲的很清楚了，这里不再介绍。

注意任何GitHub用户均可使用该URL访问此公开版本库，但只有版本库建立者luffyke具有读写权限，其他人只有只读权限。在初始化版本库之前，最好先确认是否是用正确的公钥进行认证。  