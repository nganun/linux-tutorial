# Linux 应该这么学

## 5. 用户身份与用户权限

### 5.1 用户身份与能力

> UID: User IDentification

- 管理员 UID 为 0： 系统管理员用户
- 系统用户 UID 为 1~999：默认服务程序会由独立的系统用户负责运行
- 普通用户 UID 从 1000 开始：是由管理员创建的用于日常工作的用户

> GID: Group IDentification

在 Linux 系统中创建每个用户时，将自动创建一个与其同名的基本用户组，而且这个基本用户组只有该用户一个人。
如果该用户以后被归纳到其他用户组，则这个其他用户组称之为扩展用户组。
一个用户只有一个基本用户组，但是可以有多个扩展用户组。

#### 5.1.1 id 命令

- 用法：id 用户名
- 功能：查看用户的基本信息，例如用户 ID、基本组、扩展组 GID
- 示例
```sh
id qk
```

#### 5.1.2 useradd 命令

- 用法：useradd [参数] 用户名
- 功能：创建用户。默认的用户家目录会被存放在 /home 目录中，默认的 Shell 解释器为 /bin/bash，而且默认会创建一个与该用户组同名的基本用户组。
- 参数
  - -d: 指定用户家目录
  - -u: 指定该用户默认的 UID
  - -g: 指定一个初始有用户基本组
  - -G: 指定一个或多个扩展组
  - -s: 指定该用户的默认 Shell 解释器
- 示例
```sh
useradd qk
id qk
# 如果用户的解释器被调协为 nologin，代表该用户不能登录到系统
useradd -d /home/linux -u 8888 -s /sbin/nologin qk
```

#### 5.1.3 groupadd 命令

- 用法：groupadd [参数] 群组名
- 功能：用于创建新的用户组
- 示例：
```sh
groupadd team
```

#### 5.1.4 usermod 命令

- 用法：usermod [参数] 用户名
- 功能：user modify，用于修改用户的属性，如用户的 UID、基本/扩展组、默认终端等。用户的信息保存在 `/etc/passwd` 文件中。可以直接修改此配置文件，也可以直接用此命令修改。
- 参数
  - -c: 填写用户的备注信息
  - -d -m: 重新指定用户的家目录并自动把旧的数据转移过去
  - -u: 修改用户的 UID
  - -g: 变更所属用户组
  - -G: 变更扩展用户组
  - -L: 锁定用户禁止基登录系统
  - -U: 解锁用户，允许其登录系统
  - -s: 变更默认终端
- 示例
```sh
id qk
# 将用户 qk 添加到 root 用户组
usermod -G root qk
usermod -u 8888 qk
usermod -s /sbin/nologin qk
su - qk
# This account is currently not available
```

#### 5.1.5 passwd 命令

- 用法：passwd [参数] 用户名
- 功能：用于修改用户的密码过期时间等信息。普通用户只能使用 passwd 命令修改自己系统的密码，root 可以修改其他所有人的密码，且 root 在修改自己或他人的密码时不需要验证旧密码。
- 参数：
  - --stdin: 通过标准输入修改用户的密码
  - -d: 使该用户可以使用空密码登录系统
  - -e: 强制用户在下次登录时修改密码
  - -S: 显示用户的密码是否被锁定，以及密码所采用的加密算法等
- 示例
```sh
# 通过标准输入修改用户密码
echo "NewPassWold" | passwd --stdin qk
# root 修改自己的密码 # 输入密码值 # 再次输入密码值
passwd
# root 修改其他人的密码 # 输入密码值 # 再次输入密码值
passwd qk
# 锁定用户
passwd -l qk
passwd -S qk
passwd -u qk
passwd -S qk
```

#### 5.1.6 userdel 命令
 
- 用法：userdel [参数] 用户名
- 功能：删除用户，默认家目录会保留下
- 参数：
  - -f: 强制删除用户
  - -r: 同时删除用户及家目录
- 示例：
```sh
userdel qk
id qk
cd /home
rm -rf qk
```

### 5.2 文件权限与归属

对于目录文件
- 可读：表示可以读取目录内的文件列表
- 可写：表示能够在目录内新增、删除、重命名文件
- 可执行：表示能够进入该目录

文件类型：
- -: 普通文件
- d: 目录文件
- l: 链接文件
- p: 管道文件
- b: 块设备文件
- c: 字符设备文件

```sh
ls -l Install.log
-rw-r--r-- 1 root root 34298 04-02 00:23 Install.log
# 普通文件，所属人可读写，所属组可读，其它可读，所有者，所属组，文件字节，最后修改时间，文件名
```

### 5.3 文件的特殊权限

#### 5.3.1 SUID

SUID 是一种对二进制程序进行设置的特殊权限，能够让二进制程序的执行者临时拥有所有者的权限（仅对拥有执行权限的二进制程序有效）
用户的密码保存在 /etc/shadow 文件内
一旦某个命令被设置了 SUID 权限，就意味着凡是执行该文件的人都可以临时获取到文件所有者所对应的更高权限。因此不要把 SUID 权限设置到 vim, cat, rm 等命令上

```sh
ls -l /etc/shadow # ----------
ls -l /bin/passwd # -rwsr-xr-x
```

#### 5.3.2 SGID

SGID 特殊权限有两种应用场景
- 当对二进制程序进行设置时，能够让执行者临时获取文件所属组的权限
- 当对目录进行设置时，则是让目录内新创建的文件自动继承该目录原有用户组的名称

```sh
mkdir testdir
chmod -R 777 testdir
chmod -R g+s testdir
ls -ald testdir # drwxrwsrwx
```

- chmod
  - change mode，用于设置文件的一般权限与特殊权限
  - chmod [参数] 文件名

```sh
chmod 760 anacoda-ks.cfg
```

- chown
  - change own，设置文件的所有者与所有组
  - chown 所有者:所属组 文件名

```sh
chown qk:qk anaconda-ks.cfg
chown -R qk:qk UI
```

以上两个命令对文件夹进行操作时，加上 -R 来表示递归

#### SBIT

Sticky Bit 保护位。可确保用户只能删除自己的文件，而不能删除其他用户的文件。
当目录被设置 SBIT 特殊权限后，文件的其他用户权限部分的 x 执行权限就会被替换成 t 或 T，原本有 x 执行权限则会写成 t，原来没 x 执行权限则会被写成 T

u+s 设置 SUID 权限
u-s 取消 SUID 权限
g+s 设置 SGID 权限
g-s 取消 SGID 权限
o+t 设置 SBIT 权限
o-t 取消 SBIT 权限

```sh
chmod -R o+t linux/
# 第一位为特殊权限，后三位为标准权限
chmod -R 7777 linux/
```

### 5.4 文件的隐藏属性

#### 5.4.1 chattr 命令

- 用法：chattr [参数] 文件名称
- 功能：设置文件的隐藏属性，change attribute，+ 为添加属性，- 为删除属性
- 参数：
  - i: 无法对文件进行修改；如果对目录设置了该参数，只能修改其中的子文件内容，而不能新建或删除文件
  - a: 仅能追加内容，无法删除或覆盖内容
  - s: 彻底从硬盘中删除
- 示例
```sh
echo "for test" > test
rm test
echo "for test" > test
chattr +a test
rm test # Operation not permitted
```

#### 5.4.2 lsattr 命令

- 用法：lsattr [参数] 文件名称
- 功能：查看文件的隐藏权限
- 示例
```sh
lsattr test
```

### 5.5 文件访问控制列表

ACL： Access Control List
针对指定的用户或用户组设置文件或目录的操作权限

#### 5.5.1 setfacl 命令

- 用法：setfcal [参数] 文件名称
- 功能：set files ACL，用于管理文件的 ACL 规则
- 参数：
  - -m: 修改权限
  - -M: 从文件中读取权限
  - -x: 删除某个权限
  - -b: 删除全部权限
  - -R: 递归子目录
- 示例：
```sh
# 普通用户原来无法进入 root 目录，现在可以进入了
setfacl -Rm u:qk:rwx /root
# 针对用户组进入设置
setfacl -m g:qk:rw /etc/fstab
# 删除一条指定的权限
setfacl -x g:qk /etc/fstab
```

如果设置了 ACL ，文件属性中的 . 会变成 +

#### 5.5.2 getfacl 命令

- 用法：getfacl [参数] 文件名称
- 功能：查看文件的 ACL 权限，get files ACL
- 示例
```sh
getfacl /root
```

- 备份
```sh
getfacl -R home > backup.acl
# 备份时已经指定了目录，在恢复时不需要写对应的目录名称
setfacl --resore backup.acl
```

### 5.6 su 命令与 sudo 命令

su 命令用来切换用户，使当前用户在不退出登录的情况下，顺畅地切换到其他用户。
- - 意味着完全切换到新的用户，即把环境变量信息也变更为新用户的相关信息，则不是保留原始的信息。
```
su - qk
```

sudo 命令用于给普通用户提供额外的权限，语法为 sudo [参数] 用户名
配置文件为 /etc/sudoers
- 参数
  - -h: 列出帮助信息
  - -l: 列出当前用户可执行的命令
  - -u 用户名或UID：以指定的用户身份执行命令
  - -k: 清空密码的有效时间，下次执行 sudo 时需要再次进行密码验证
  - -b: 在后台执行指定的命令

也可以用 visudo 命令直接更改配置文件

```sh
谁可以使用 允许使用的主机 = （以谁的身份） 可执行命令的列表
谁可以使用：稍后要为哪个用户进行命令授权
允许使用的主机：可以填写 ALL 表示 不限制来源的主机，亦可填写如 192.168.10.0/24 这样的网段来限制地址
以谁的身份：可以填写 ALL 表示 系统最高权限，也可以是另外一位用户的名字
可执行命令的列表：可以填写 ALL 表示不限制命令，也可以填写如 /usr/bin/cat 这样的文件名称列表，用逗号隔开
qk ALL=(ALL) ALL

```

```sh
sudo -l # 会让输入当前用户的密码
# qk 用户登录
ls /root # Permission denied
sudo ls /root # work
```

```sh
# 配置 whereis
qk ALL=(ALL) /usr/bin/cat,/usr/sbin/reboot
```

下次执行 sudo 不需要再输入密码

```sh
qk ALL=(ALL) NOPASSWD:/usr/bin/cat,/usr/sbin/reboot
```
