在Ubuntu下用jailkit建立一个被监禁的Shell

================================================================================

### Jailkit和jailed Shell ###

监狱性的shell是一类限制性的shell，提供给用户非常真实的Shell模样，但是不允许它查看和修改真正的文件系统。Shell内的文件系统不同于底层的文件系统。这种功能是通过chroot和其他多种程序实现的。举例来说，建立一个用户的linux shell可能仅仅为了玩耍。或者在一个限定的环境里运行一些程序的所有功能等。

在这个教程里我们将会探讨在Ubuntu下用jailkit建立一个监禁的shell。Jailkit是辅助程序，允许快速的建立一个监禁的shell，监禁的用户，在受监禁的环境里配置程序并运行。

Jailkit can be downloaded from [http://olivier.sessink.nl/jailkit/][1]

我们已经谈论过关于在Ubuntu下安装jailkit，如果有不懂，多看看那篇文章。

### 配置jailed Shell ###

#### 配置jail环境 ####

我们需要建立一个目录来存放所有jail环境的配置。这不是重点，我们可以创建个/opt/jail的目录。

    $ sudo mkdir /opt/jail

这个目录应为Root所有。所以用chown。

    $ sudo chown root:root /opt/jail

#### 2. 设置在jail中可用的程序　####

任何程序想要在jail中执行则必须用jk_init命令拷贝到目录中。

例如：

    $ sudo jk_init -v /jail basicshell 

    $ sudo jk_init -v /jail editors 

    $ sudo jk_init -v /jail extendedshell 

    $ sudo jk_init -v /jail netutils 

    $ sudo jk_init -v /jail ssh 

    $ sudo jk_init -v /jail sftp

    $ sudo jk_init -v /jail jk_lsh

或一次性解决：

    $ sudo jk_init -v /opt/jail netutils basicshell jk_lsh openvpn ssh sftp

像basicshell, editors, netutils是一些组名，其中包含多个程序。复制到jail shell中的每个组都是可执行文件,库文件等的集合。比如**basicshell**就在jail提供有bash, ls, cat, chmod, mkdir, cp, cpio, date, dd, echo, egrep等程序。

完整的程序列表设置，你可以在/etc/jailkit/jk_init.ini中查看。

    jk_lsh (Jailkit limited shell) - is an important section, and must be added.

#### 3. 创建将被监禁的用户 ####

需要将用户放入jail里。可以先创建一个

    $ sudo adduser robber

    Adding user `robber' ...

    Adding new group `robber' (1005) ...

    Adding new user `robber' (1006) with group `robber' ...

    Creating home directory `/home/robber' ...

    Copying files from `/etc/skel' ...

    Enter new UNIX password: 

    Retype new UNIX password: 

    passwd: password updated successfully

    Changing the user information for robber

    Enter the new value, or press ENTER for the default

            Full Name []: 

            Room Number []: 

            Work Phone []: 

            Home Phone []: 

            Other []: 

    Is the information correct? [Y/n] y

注意:目前创建的是一个活动在文件系统中的普通用户并没有添加到jail中。

在下一步这个用户会被监禁在jail里。

这时候如果你查看/etc/passwd文件，你会在文件最后看到跟下面差不多的一个条目。

    robber:x:1006:1005:,,,:/home/robber:/bin/bash

这是我们新创建的用户，最后部分的/bin/bash指示了这个用户如果登入了那么它可以在系统上正常的Shell访问

#### 4. 监禁用户 ####

现在是时候将用户监禁在jail中

    $ sudo jk_jailuser -m -j /opt/jail/ robber

执行上列命令后，用户robber将会被监禁。

如果你现在再观察/etc/passwd文件，会发现类似下面的最后条目。

    robber:x:1006:1005:,,,:/opt/jail/./home/robber:/usr/sbin/jk_chrootsh

注意:最后两部分表明用户主目录和shell类型已经被改变了。现在用户的主目录在/opt/jail(jail环境)中。用户的Shell是一个名叫jk_chrootsh的特殊程序，会提供jailed Shell。

jk_chrootsh这是个特殊的shell，每当用户登入系统时，它都会将用户放入jail中。

到目前为止jail配置已经几乎完成了。但是如果你试图用ssh连接，那么注定会失败,像这样：

    $ ssh robber@localhost

    robber@localhost's password: 

    Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-25-generic x86_64)

     * Documentation:  https://help.ubuntu.com/

    13 packages can be updated.

    0 updates are security updates.

    *** /dev/sda7 will be checked for errors at next reboot ***

    *** /dev/sda8 will be checked for errors at next reboot ***

    Last login: Sat Jun 23 12:45:13 2012 from localhost

    Connection to localhost closed.

    $

连接会立马关闭，这意味着用户已经活动在一个受限制的shell中。

#### 5. 给在jail中的用户Bash Shell ####

下个重要的事情是给用户一个正确的bash shell，但是他却在jail中。

打开下面的文件

    /opt/jail/etc/passwd

这是个jail中的password文件。类似如下

    root:x:0:0:root:/root:/bin/bash

    robber:x:1006:1005:,,,:/home/robber:/usr/sbin/jk_lsh

将/usr/sbin/jk_lsh改为/bin/bash

    root:x:0:0:root:/root:/bin/bash

    robber:x:1006:1005:,,,:/home/robber:/bin/bash

保存文件并退出。

#### 6. 登入jail ####

现在让我们再次登入jail

    $ ssh robber@localhost

    robber@localhost's password: 

    Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-25-generic x86_64)

     * Documentation:  https://help.ubuntu.com/

    13 packages can be updated.

    0 updates are security updates.

    *** /dev/sda7 will be checked for errors at next reboot ***

    *** /dev/sda8 will be checked for errors at next reboot ***

    Last login: Sat Jun 23 12:46:01 2012 from localhost

    bash: groups: command not found

    I have no name!@desktop:~$

jail说'I have no name!'，哈哈。现在我们在jail中有个完整功能的bash shell。

现在通过操作检查环境。jail中的root /实际就是真实文件系统中的/opt/jail.但这只有我们自己知道，jail用户并不知情。

    I have no name!@desktop:~$ cd /

    I have no name!@desktop:/$ ls

    bin  dev  etc  home  lib  lib64  run  usr  var

    I have no name!@desktop:/$

也只有我们通过jk_cp拷贝到jail中的命令能使用。

如果登入失败，请检查一下/var/log/auth.log的错误信息。

现在尝试运行一些网络命令，类似wget的命令。

    $ wget http://www.google.com/

如果你获得类似的错误提示：

    $ wget http://www.google.com/

    --2012-06-23 12:56:43--  http://www.google.com/

    Resolving www.google.com (www.google.com)... failed: Name or service not known.

    wget: unable to resolve host address `www.google.com'

你可以通过运行下列两条命令来解决这个问题:

    $ sudo jk_cp -v -j /opt/jail /lib/x86_64-linux-gnu/libnss_files.so.2

    $ sudo jk_cp -v -j /opt/jail /lib/x86_64-linux-gnu/libnss_dns.so.2

这样才能正确的定位到libnss_files.so和libnss_dns.so

### 在jail中运行程序或服务　 ###

此时此刻配置已经完成了。Jails可以在限制/安全的环境里运行程序或服务。用**jk_chrootlaunch**命令在jail中启动一个程序或守护进程。

    $ sudo jk_chrootlaunch -j /opt/jail -u robber -x /some/command/in/jail

jk_chrootlaunch工具可以在jail环境中启动一个特殊的进程同时指定用户特权。如果守护进程启动失败，请检查/var/log/syslog/错误信息。

在jail中运行程序之前，该程序必须已经用jk_cp命令复制到jail中。

   jk_cp -　将文件包括权限信息和库文件复制到jail的工具　

进一步阅读有关其他jailkit命令信息，可以阅读文档，[http://olivier.sessink.nl/jailkit/][1]

--------------------------------------------------------------------------------

via: http://www.binarytides.com/setup-jailed-shell-jailkit-ubuntu/

译者：[Luoxcat](https://github.com/Luoxcat) 校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创翻译，[Linux中国](http://linux.cn/) 荣誉推出

[1]:http://olivier.sessink.nl/jailkit/
[2]:
[3]:
[4]:
[5]:
[6]:
[7]:
[8]:
[9]:
[10]:
[11]:
[12]:
