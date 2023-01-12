# Linux基础--个人VPS安全

### 1.更新软件

旧版本的软件可能会含有某些漏洞，从而成为攻击者发起攻击的基础。这个怎么说呢，其实作为个人用户没有太多可以做的。
**请保持软件的最新状态吧！**
我使用`OpenMediaVault`这个NAS系统时，安全相关的更新会强制自动安装的。

```bash
sudo apt-get update && sudo apt-get upgrade
```

### 2.使用非root用户

##### 1）创建非root用户

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

