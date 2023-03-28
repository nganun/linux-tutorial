# Linux 应该这么学

## 4. Vim 编辑器与 Shell 命令脚本

### 4.1 Vim 文本编辑器 +

#### 4.1.1 编写简单文档

#### 4.1.2 配置主机名称

主机名称保存在 `/etc/hostname` 文件中，如果修改完成后没有立即生效，可以重启虚拟机

```sh
# 更改
vim /etc/hostname
# 查看
hostname
```

#### 4.1.3 配置网卡信息

在 RHEL5/6 中，网卡配置文件的前缀为 eth，第 1 块网卡为 `eth0`，第 2 块网卡为 `eth1`，以此类推
在 RHEL7 中，网卡配置文件前缀以 `ifcfg` 开始，再加上网卡名称，共同组成了网卡配置文件的名字，例如 `ifcfg-eno16777736`
在 RHEL8 中，网卡配置文件前缀仍以 `ifcfg` 开始，区别是网卡名称改成了类似于 `ens160` 的样子

> 现有一个名称为 `ifcfg-ens160` 的网卡设备，将其设备为开机自启动，并且 IP 地址、子网、网关等信息由人工指定，步骤如下 

```sh
# 1. 切换到 /etc/sysconfig/network-scripts 目录中，此目录存放着网卡的配置文件
# 2. 用 vim 修改网卡 ifcfg-ens160 的信息
# 设备类型：TYPE=Ethernet
# 地址分配模式：BOOTPROTO=static
# 网卡名称：NAME=ens160
# 是否启动：ONBOOT=yes
# IP 地址：IPADDR=192.168.10.10
# 子网掩码：NETMAST=255.255.255.0
# 网关地址：GATEWAY=192.168.10.1
# DNS 地址：DNS1=192.168.10.1
# 3. 执行重启网卡的命令
nmcli connection reload ens160
# 4. 用 ping 进行验证
ping 192.168.10.10
```

#### 4.1.4 配置软件仓库

1. 进入 `/etc/yum.repos.d/` 目录下，此目录存放着软件仓库的配置信息
2. 创建一个 `rhel8.repo` 的配置文件，名字随便，但后缀为 `repo`
```repo
# 仓库名称：具有唯一的标识名称，不应与其它软件仓库发生冲突
[BaseOS]
# 描述信息：描述性的词，用于识别软件仓库的用处
name=BaseOS
# 仓库位置：软件包的获取方式，可以使用 FTP 或 HTTP 下载，也可以是本地的文件(file)
baseurl=file:///media/cdrom/BaseOS
# 是否启用：设置此源是否可用；1 为可用，0 为禁用
enabled=1
# 是否检验：设置此源是否校验文件：1 为校验，0 为不校验
gpgcheck:0
# 公钥位置：若上面开户了校验功能，则此处为公钥文件位置。若没有开启，则省略不写
```
3. 创建挂载点进行挂载操作，并设置成开机自动挂载
```sh
mkdir -p /media/cdrom
mount /dev/cdrom /media/cdrom
vim /etc/fstab  # /dev/cdrom /media/cdrom iso9660 defaults 0 0
```
4. 使用 `dnf install httpd -y` 命令检查软件仓库是否已经可用

### 4.2 编写 Shell 脚本

> 查看当前 Shell
```sh
echo $SHELL
```

#### 4.2.1 编写简单的脚本

1. 脚本声明，告诉系统用哪种 Shell 解释器来执行此脚本
```
#!/bin/bash
```

2. 执行脚本
- bash example.sh
- ./example.sh
  - 此方式通过完整的路径名执行，通常会报权根问题，chmod u+x example.sh，加上权限后再执行就可以了

#### 4.2.2 接收用户的参数

```text
$0  当前 Shell 脚本程序的名称
$#  总共有几个参数
$*  所有位置的参数值
$?  显示上一次命令的执行返回值
$1/2/3  第 n 个位置的参数值
```

```sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*"
echo "第一个参数为$1，第5个参数为$5"
```

`bash example.sh one two three foue five six`

#### 4.2.3 判断用户的参数

- Shell 脚本中的条件测试语法如果成功，返回数字 0 ，如果不成功，返回非 0 数字
- 条件表达式两边必需要有一个空格

条件测试语句分为 4 种
1. 文件测试语句
  - 判断文件是否存在、权限是否满足等情况的运算符
  - -d 测试文件是否为目录类型
  - -e 测试文件是否存在
  - -f 测试文件是否为一般文件
  - -r/-w/-r 测试文件是否有权限读写执行

测试 `/etc/fstab` 是否为一个目录
```sh
[ -d /etc/fstab ]
echo $?
[ -f /etc/fstab ]
echo $?
```

2. 逻辑测试语句

&& 逻辑与，当前面的命令执行成功扣，才会执行它后面的命令

```sh
[ -e /dev/cdrom ] && echo "Exist"
```
|| 逻辑或，当前面的命令执行失败后，才会执行它后面的命令
```sh
echo $USER
[ $USER = root ] || echo "user"
su - qk
[ $USER = root ] || echo "user"
```

! 非，把条件测试中的判断结果取相反值
```sh
[ ! $USER = root ] || echo "user"
[ $USER = root ] && echo "root" || echo "user"
```

3. 整数值测试语句

可用的整数比较运算符
- -eq 是否等于
- -ne 是否不等于
- -gt 是否大于
- -lt 是否小于
- -le 是否小于等于
- -ge 是否大于等于

```sh
[ 10 -gt 10 ]
echo $?
[ 10 -eq 10 ]
echo $?
```

```sh
free -m | grep Mem | awk '{print $4}'
FreeMem=`free -m | grep Mem | awk '{print $4}'`
echo $FreeMem
[ $FreeMem -lt 1024 ] && echo "Insufficient Memory" || echo "Memory is OK"
```

4. 字符串测试语句

用于判断字符串是否为空值，或两个字符串是否相同。
它经常用来判断某个变量是否未被定义（即内容为空值）

- = 比较字符串内容是否相同
- != 比较字符串内容是否不同
- -z 判断字符串内容是否为空

```sh
[ -z $String ]
echo $0
[ ! $LANG = "en.US" ] && echo "Not en.US"
```

### 4.3 流程控制语句

#### 4.3.2 if 条件测试语句

- 单分支结构
```sh
#!/bin/bash
# 判断 /media/cdrom 目录是否存在
DIR="/media/cdrom"
if [ ! -d $DIR ]
then
  mkdir -p $DIR
fi
```
- 双分支结构
```sh
#!/bin/bash
# ping -c 次数 -i 发送间隔 -W 等待超时时间
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null
if [ $? -eq 0 ]
then
  echo "Host $1 is On-line."
else
  echo "Host $2 is Off-line."
fi
```
- 多分支结构

read 用来读取用户输入的信息，-p 参数用于向用户显示一些提示信息

```sh
#!/bin/bash
read -p "Enter your source (0-100):" GRADE
if [ $GRADE -ge 85 ] && [ $GRADE -le 100 ] ; then
  echo "$GRADE is Excelent"
elif [ $GRADE -ge 70 ] && [ $GRADE -le 84 ] ; then
  echo "$GRADE is Pass"
else
  echo "$GRADE is Fail"
fi
```

#### 4.3.2 for 条件循环语句

```sh
#!/bin/bash
# 为文件中的用户，批量创建密码
read -p "Enter The Users Password:" PASSWORD
for UNAME in `cat users.txt`
do
  id $UNAME $> /dev/null
  if [ $? -eq 0 ]
  then
    echo "$UNAME, Already exists"
  else
    useradd $UANME &> /dev/null
    echo $PASSWD | passwd --stdin $UNAME &> /dev/null
    echo "$UNAME, Create success"
  fi
done
```

```sh
#!/bin/bash
# 测试文本中的 IP，是否在线
HLIST=$(cat ~/ipaddrs.txt)
for IP in $HLIST
do
  ping -c 3 -i 0.2 -W $IP &> /dev/null
  if [ $? -eq 0 ]
  then
    echo "Host $IP is On-line"
  else
    echo "Host $IP is Off-line"
  fi
done
```

$() 是一种完全类似于反引号的操作符，效果是执行括号或又引号括起来的字符串中的命令

#### 4.3.3 while 条件循环语句

$RANDOM 会产生一个 0~32767 之间的一个数字
% 取余
expr 进行计算

```sh
#!/bin/bash
# 猜数
PRICE=$(expr $RANDOM % 1000)
TIMES=0
echo "商品实际价格为 0 ~ 999 之间，猜猜看是多少？"
while true
do
  read -p "请输入你猜测的价格数目：" INI
  let TIMES++
  if [ $INT -eq $PRICE ] ; then
    echo "恭喜你答对了，实际价格为 $PRECE"
    echo "您总共猜测了 $TIMES 次"
    exit
  elif [ $int -gt $PRICE ] ; then
    echo "太高了"
  else
    echo "太低了"
  fi
done
```

#### 4.3.4 case 条件测试语句

```sh
#!/bin/bash
read -p "请输入一个字符，并按Enter键确认：" KEY
case $KEY in
  [a-z][A-Z])
    echo "您输入的是字母"
    ;;
  [0-9])
    echo "您输入的是数字"
    ;;
  *)
    echo 您输入的是其它字符
esac
```

### 4.4 计划任务服务程序


- 一次性计划任务
  - -f 指定包含命令的任务文件
  - -q 指定新任务名称
  - -l 显示待执行任务的列表
  - -d 删除指定的待执行命令
  - -m 任务执行后向用户发邮件

交互式设置
```sh
at 23:30
systemctl restart httpd
<Ctrl>+<d>
at -l
echo "systemctl restart httpd" | at 23:30
at -l
# 删除第 2 个任务
atrm 2
```

把计划任务写入脚本，当用户激活该脚本后再开始倒计时执行
```sh
at now +2 MINUTE
systemctl restart httpd
<ctrl>+<d>
```

- 长期性计划任务
  - -e 编辑计划任务
  - -u 指定用户名称
  - -l 列出任务列表
  - -r 删除计划任务

分、时、日、月、星期(0与7都是星期日)、命令
```sh
# whereis tar
# 每周一、三、五凌晨 3:25 ，用 tar 命令把某个网站的数据目录进行打包备份
25 3 * * 1,3,5 /usr/bin/tar -czvf back.tar.gz /home/wwwroot
0 1 * * 1-5 /usr/bin/rm -rf /tmp/*
```

  - , 表示多个时间段
  - - 表示一段连续的时间周期
  - **/ 表示执行任务的间隔时间

  - 分字段必需有数值，绝对不能为空或是 * 号
  - 而日和星期不能同时使用，会发生冲突

```sh
crontab -r # 删除所有
crontab -l # 查看
```




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
