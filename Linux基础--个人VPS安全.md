# Linux基础--个人VPS安全

1.更新软件

旧版本的软件可能会含有某些漏洞，从而成为攻击者发起攻击的基础。这个怎么说呢，其实作为个人用户没有太多可以做的。**请保持软件的最新状态吧！**我使用`OpenMediaVault`这个NAS系统时，安全相关的更新会强制自动安装的。

```bash
sudo apt-get update && sudo apt-get upgrade
```

## 2.使用非root用户

### 1)创建非root用户

从今天开始，希望你不要再直接使用root用户了！我们先马上来创建一个新的**非root用户**吧！

首先，创建一个新用户组：

```bash
sudo groupadd -g 344 test
```

344叫做gid，我是随便取的。你只要取一个不为0的整数应该都可以，因为0是root用户组专用的gid。另外，1000、100也是很常见的内置用户组。如果你打算用这些用户组，那你可以不需要运行此命令。

这时，有童鞋可能会问：**为什么用户(组)会有id和name这两个属性呢？**这个问题以后再讨论。

接着，我们为test用户组创建一个用户，名字叫做test_user。

```bash
# 创建一个新用户
sudo useradd \
    -m -d /home/test_user `#自动创建用户目录`\
    -s /bin/bash `#设置默认shell`\
    -g test `#主组`\
    -G sudo `#副组`\
    test_user
```

这个命令有点长，而且结构有点奇怪，哈哈 😄

这其实是一种shell命令的可视化写法，通过\号将一行中的多个元素分至不同的行。在shell中，这会被认为是一条命令。

这样拆分写的好处是，可以比较清晰地看到代码的功能结构。你以后运行或记录很长的shell命令时，也可以使用这个技巧。在markdown的代码框中，可以用TAB键来迅速生成前方空格，或者用Shift+TAB消除前方空格。以后有机会，你可以自己试试看。它和下面的写法是等价的：

```bash
sudo useradd -m -d /home/test_user -s /bin/bash -g test -G sudo test_user
```

```bash
# 删除用户
sudo userdel test_user

# 删除用户组
sudo groupdel test
```

注意，要将所有的用户删除后才可以删除用户组。

| user      | password |
| --------- | -------- |
| test_user | 0709     |

下面，我给这个用户设置一个密码：

```bash
# 设置密码
sudo passwd test_user # 假设我输入了testtest
```

那么，要怎么切换到`test_user`用户呢？可以这样：

```bash
su test_user # su <用户名>
```

如果你目前的用户级别不比test_user高，它会要求你输入密码。这个密码就是testtest。

如果你是root，直接就切换过去了。应该可以理解这种层次的关系以及它的合理性吧？

在切换的一瞬间，你会发现漂亮的ohmyzsh不见了。这是因为我们用了默认的/bin/bash的缘故。现在先不要管外观这些花里胡哨的东西先😜

这里，我们可以观察一下主机都有哪些用户：

```bash
sudo less /etc/passwd      #按q终止
```

### 2)观察home目录

```bash
cd ~
ls -hl
```

输出`total 0`，表示什么都没有。

这时，观察一下home目录的绝对路径

```bash
pwd
```

输出`/home/test_user`。

好吧，在Linux里创建新用户就是这么简单？对呀，我也不知道为什么这么简单呀😆

这里就是新家了，以后基本上就是在这里部署应用啦！

### 3)探索sudo

这个时候，我们试一下与`root`的交互。比如：

```bash
cd /root
```

输出bash: cd: /root: Permission denied。因为我们是普通用户嘛，没有办法进入root的目录。在大多数情况下，你甚至没有办法进入另外一个普通用户的home目录，除非你的权限比它高级。所以如果你想保护一些东西，可以用root权限保护它。

我们再试试能不能使用sudo：

```bash
sudo apt-get update
```

输入密码后就可以正常地运行了。因为创建用户的时候，`sudo`是`test_user`的副组。是不是很帅呐☺️

这时，我们再输入这个命令，观察一下我们的用户：

```bash
id
```

输出：

```bash
uid=1002(test_user) gid=344(test) groups=344(test),27(sudo)
```

uid是用户的id，gid是用户组的id。这里看到test_user属于test和sudo两个用户组。如果test_user不属于sudo用户组，它将不可以使用sudo!

至于uid和gid的具体用途，以后玩Docker的时候你会有所体会的。这里按下不表。

最后，我个人认为你可以看一下/etc/sudoers文件：

```bash
ls -hl /etc/sudoers
```

输出是

```bash
-r--r----- 1 root root 825 Apr 17 14:01 /etc/sudoers
```

我们发现即使是root权限有只能查看。

实际上，它控制了用户对于`root`权限调用的程度。

```bash
sudo chmod +600 /etc/sudoers
sudo vim /etc/sudoers
```

```bash
sudo chmod -w /etc/sudoers && ls -hl /etc/sudoers
```

## 3.禁用root使用ssh登陆

我们用非root用户的重要原因，就是因为**我们要禁用root通过ssh登陆我们的VPS。**像腾讯云，在初始化系统的时候就是禁root的。这是为什么呢？

一般情况下，主机是允许root用户远程登陆的。而root用户的名字一般就是root。所以，如果攻击者知道你VPS的ip地址（ping你的域名获得）、用户名（默认有一个root可用）和ssh端口号（默认22），那么保护你电脑的就只有你的用户密码了。如果你的root密码被破解了（比如你用了一些123456的弱密码），那你的VPS就任人鱼肉了！如果你只是非root用户密码被破解，那么root用户可以暂时保护一下你。不过这种时候基本上已经非常危险了。

所以呀，我们用非root用户是有原因的，这样我们可以在用户名这个层面上增强VPS的安全防护。

当然，**如果你能进一步操作ssh端口号和ip，那么你的VPS的安全等级将呈指数级上升！**我下面还会介绍如何进行设置ssh端口和ip的！

这里我们开始讲讲**怎么禁用root用户远程ssh登陆。**首先，我们编辑ssh程序的设置文件：

```bash
sudo vim /etc/ssh/sshd_config
#vim使用方法  tips
#用vim打开后 ， 点i键进入输入模式
#esc退出
#ctrl+；调起命令 
# 输入wq回车  保存并推出
```


找到PermitRootLogin参数，它的值yes改成no。

重启ssh服务后生效：

```bash
sudo service sshd restart
```


这里要切记，不要关闭旧的Shell。新开一个Shell，用你刚刚的非root用户登陆Shell。成功后再关闭旧的Shell窗口。切记切记！
