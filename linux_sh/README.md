# Linux学习 
> 参考：http://www.runoob.com/linux/linux-file-attr-permission.html
## 操作命令
* ls -l

```
ls -l
total 64
drwxr-xr-x 2 root  root  4096 Feb 15 14:46 cron
drwxr-xr-x 3 mysql mysql 4096 Apr 21  2014 mysql
```
> 第0位：文件类型，1-3：所有者权限，4-6：所有者组权限，7-9：其他用户权限，-：无权限，r：读权限，w：读权限，r：执行权限

* chmod 命令修改权限
```
chmod [-R] xyz 文件或目录
chmod u=rwx,g=rx,o=r 文件名
```
* ls: 列出目录
```
-a ：全部的文件，连同隐藏档( 开头为 . 的文件) 一起列出来(常用)
-d ：仅列出目录本身，而不是列出目录内的文件数据(常用)
-l ：长数据串列出，包含文件的属性与权限等等数据；(常用)
```

* cd：切换目录
* pwd：显示目前的目录
* mkdir：创建一个新的目录
```
-m ：配置文件的权限
mkdir -m 711 test2
-p ：递归创建
```

* rmdir：删除一个空的目录
```
-p ：连同上一级『空的』目录也一起删除
```

* cp: 复制文件或目录
```
-a：相当於 -pdr 的意思，至於 pdr 请参考下列说明；(常用)

-d：若来源档为连结档的属性(link file)，则复制连结档属性而非文件本身；

-f：为强制(force)的意思，若目标文件已经存在且无法开启，则移除后再尝试一次；

-i：若目标档(destination)已经存在时，在覆盖时会先询问动作的进行(常用)

-l：进行硬式连结(hard link)的连结档创建，而非复制文件本身；

-p：连同文件的属性一起复制过去，而非使用默认属性(备份常用)；

-r：递归持续复制，用於目录的复制行为；(常用)

-s：复制成为符号连结档 (symbolic link)，亦即『捷径』文件；

-u：若 destination 比 source 旧才升级 destination ！
```

* rm: 移除文件或目录
```
-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
-i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！
-u ：若目标文件已经存在，且 source 比较新，才会升级 (update)
```
* cat  由第一行开始显示文件内容
```
-A ：相当於 -vET 的整合选项，可列出一些特殊字符而不是空白而已；
-b ：列出行号，仅针对非空白行做行号显示，空白行不标行号！
-E ：将结尾的断行字节 $ 显示出来；
-n ：列印出行号，连同空白行也会有行号，与 -b 的选项不同；
-T ：将 [tab] 按键以 ^I 显示出来；
-v ：列出一些看不出来的特殊字符
检看 /etc/issue 这个文件的内容：
```

* tac  从最后一行开始显示，可以看出 tac 是 cat 的倒著写！
* nl   显示的时候，顺道输出行号！
```
-b ：指定行号指定的方式，主要有两种：
-b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
-b t ：如果有空行，空的那一行不要列出行号(默认值)；
-n ：列出行号表示的方法，主要有三种：
-n ln ：行号在荧幕的最左方显示；
-n rn ：行号在自己栏位的最右方显示，且不加 0 ；
-n rz ：行号在自己栏位的最右方显示，且加 0 ；
-w ：行号栏位的占用的位数。
```

* more 一页一页的显示文件内容
```
空白键 (space)：代表向下翻一页；
Enter         ：代表向下翻『一行』；
/字串         ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
:f            ：立刻显示出档名以及目前显示的行数；
q             ：代表立刻离开 more ，不再显示该文件内容。
b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。
```

* less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
```
空白键    ：向下翻动一页；
[pagedown]：向下翻动一页；
[pageup]  ：向上翻动一页；
/字串     ：向下搜寻『字串』的功能；
?字串     ：向上搜寻『字串』的功能；
n         ：重复前一个搜寻 (与 / 或 ? 有关！)
N         ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
q         ：离开 less 这个程序；
```

* head 只看头几行
```
n ：后面接数字，代表显示几行的意思
```

* tail 只看尾巴几行
```
-n ：后面接数字，代表显示几行的意思
-f ：表示持续侦测后面所接的档名，要等到按下[ctrl]-c才会结束tail的侦测
```

* df：列出文件系统的整体磁盘使用量
```
-a ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；
-k ：以 KBytes 的容量显示各文件系统；
-m ：以 MBytes 的容量显示各文件系统；
-h ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；
-H ：以 M=1000K 取代 M=1024K 的进位方式；
-T ：显示文件系统类型, 连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；
-i ：不用硬盘容量，而以 inode 的数量来显示
```

* du：检查磁盘空间使用量
```
-a ：列出所有的文件与目录容量，因为默认仅统计目录底下的文件量而已。
-h ：以人们较易读的容量格式 (G/M) 显示；
-s ：列出总量而已，而不列出每个各别的目录占用容量；
-S ：不包括子目录下的总计，与 -s 有点差别。
-k ：以 KBytes 列出容量显示；
-m ：以 MBytes 列出容量显示；
```

* fdisk：用于磁盘分区
```
-l ：输出后面接的装置所有的分区内容。若仅有 fdisk -l 时， 则系统将会把整个系统内能够搜寻到的装置的分区均列出来。
```

* mkfs：磁盘格式化
```
-t ：可以接文件系统格式，例如 ext3, ext2, vfat 等(系统有支持才会生效)
```

* fsck：磁盘检验
```
-t : 给定档案系统的型式，若在 /etc/fstab 中已有定义或 kernel 本身已支援的则不需加上此参数
-s : 依序一个一个地执行 fsck 的指令来检查
-A : 对/etc/fstab 中所有列出来的 分区（partition）做检查
-C : 显示完整的检查进度
-d : 打印出 e2fsck 的 debug 结果
-p : 同时有 -A 条件时，同时有多个 fsck 的检查一起执行
-R : 同时有 -A 条件时，省略 / 不检查
-V : 详细显示模式
-a : 如果检查有错则自动修复
-r : 如果检查有错则由使用者回答是否修复
-y : 选项指定检测每个文件是自动输入yes，在不确定那些是不正常的时候，可以执行 # fsck -y 全部检查修复。
```

* mount/unmount：磁盘挂载/磁盘卸载
```
-f ：强制卸除！可用在类似网络文件系统 (NFS) 无法读取到的情况下；
-n ：不升级 /etc/mtab 情况下卸除。
```

* 解压/压缩
```
1、*.tar 用 tar -xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar -xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar -xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar -xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压
```

* linux时间管理
```
1. 查看时间：date
2. 查看时区：date -R 或 date +%Z
3. 设置时间：date -s "2018/8/8 10:00:00" 或 date -s 10:10:10
4. 修改时区：cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
5. 修改时区：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
6. 查看硬件时间：hwclock 或 clock
7. 将硬件时间写入到系统时间：hwclock -s
8. 将系统时间卸乳到硬件时间：hwclock -w
9. 时间同步：yum install ntpdate
```

* nohup无log启动：nohup java -jar xxxx.jar >/dev/null 2>&1 &
* 环境变量：/etc/profile
* 查询目录下文件大小：
```
du -sh *
du -ah --max-depth=1     a表示显示目录下所有的文件和文件夹（不含子目录），h表示以人类能看懂的方式，max-depth表示目录的深度。
```
* 已安装软件: rpm -qa | grep jdk
* 安装软件: yum install jdk-8u131-linux-x64.rpm 
* 切换java版本: update-alternatives --config java
* 检索文件内容：cat dataservice.out | grep Cmt | tail -1000
* 传输文件：scp jdk-8u121-linux-x64.rpm root@10.168.24.62:/
* 查看端口占用：netstat -tunlp | grep 端口号：netstat -antp | grep 61617
```
-t (tcp) 仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名
```
* linux查看进程莫名消失
```
1. dmesg | egrep -i -B100 'killed process'
2.1 egrep -i 'killed process' /var/log/messages 
2.2 egrep -i -r 'killed process' /var/log
3. journalctl -xb | egrep -i 'killed process'
```
* man 查询软件文档
* iptables
  * iptables -nL --line-number

* ss -npl
* ip link
* tcpdump -i eth0 -nv tcp dst port 21985
* jmap -histo:live pid > 21840.log
