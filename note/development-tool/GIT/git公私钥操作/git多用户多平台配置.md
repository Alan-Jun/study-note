在Git使用中经常会碰到多用户问题，例如：你在公司里有一个git账户，在github上有一个账户，并且你想在一台电脑上同时对这两个git账户进行操作，此时就需要进行git多用户配置。
首先配置不同的SSH KEY，使用ssh-keygen命令产生两个不同的SSH KEY，进入.ssh目录：

```properties
#切换到.ssh目录
cd ~/.ssh  
#使用自己的企业邮箱产生SSH KEY
ssh-keygen -t rsa -C 你的企业邮箱
#企业的可以使用id_rsa，也可以自己起名，例如：id_rsa_work
Enter file in which to save the key (/root/.ssh/id_rsa): id_rsa
#将ssh key添加到SSH agent中，该命令如果报错：Could not open a connection to your authentication agent.无法连接到ssh agent，可执行ssh-agent bash命令后再执行ssh-add命令。
ssh-add ~/.ssh/id_rsa 
```

同理，配置自己的github账户，再有其他账户类似:

```properties
#切换到.ssh目录
cd ~/.ssh  
#使用自己github的注册邮箱产生SSH KEY
ssh-keygen -t rsa -C 你的个人邮箱  
#github的SSH KEY
Enter file in which to save the key (/root/.ssh/id_rsa): id_rsa_github
#将ssh key添加到SSH agent中 ，该命令如果报错：Could not open a connection to your authentication agent.无法连接到ssh agent，可执行ssh-agent bash命令后再执行ssh-add命令。
ssh-add ~/.ssh/id_rsa_github
```

在生成ssh key之后，需要**分别**在github的profile中和公司git的profile中编辑SSH KEY，以github为例：

```properties
1. Title，可以随便写： 
2. 将.ssh目录下对应的id_rsa_github.pub中的内容拷到Key中，点击Add SSH key按钮即可。公司的git类似。 
```

然后在.ssh目录下配置config文件：

```properties
#创建并编辑config文件
#公司的git地址
Host git.***.com  #自定义名称，最好和hostName 得公司git地址相同
   User ALen  #自定义name 随便写 
   Hostname git.***.com  #公司的git地址
   IdentityFile ~/.ssh/id_rsa  #访问公司git的SSH KEY
   Port   ***  #公司的git端口

Host github.com
   User ymj
   Hostname github.com #github的地址
   IdentityFile ~/.ssh/id_rsa_github  #访问github的SSH KEY
```

测试配置是否成功

```properties
#github的地址
ssh -T git@github.com 
#出现如下内容，表示成功链接github，***为你的github账户的用户名
Hi ***! You've successfully authenticated, but GitHub does not provide shell access.
#公司的git地址
ssh -T git@git.***.com 
#出现如下内容，表示成功链接github，***为公司git账户的用户名
Hi ***! You've successfully authenticated, but GitHub does not provide shell access.
```

