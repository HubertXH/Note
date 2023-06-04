[toc]
# man 命令
### 常用指令对应的数值：
- 1：用户在shell环境种可以操作的指令或可执行的文件
- 5：配置文件或者是某些文件的各式
- 8：系统管理员可用指令  

---

# 常用关机指令：   
- sync:将内存中的数据写入磁盘
- shutdown:关机。格式：shutdwom [-rhkc] [time][告警信息]；常用参数：-r 将系统服务停掉之后立即重新启动，-h 将系统服务停掉之后立即关机，
- reboot 重启,
- poweroff
- who : 查看在线用户信息
- netstat -a :查看联网情况
- ps -aux : 查看后台运行的程序

---

# Linux 文件属性：
> -rw-r--r-- 1 root root    0 Jun  1 18:34 test

1. 第一个位置代表文件类型：
   - d 代表目录
   - “-” 代表文件
   - i 代表链接文档
   - b 代表装置文件里面的可存储的接口设备
   - c 代表装置文件里面的穿行接口设备，例如：键盘，鼠标等等。

2. 接下里的字符，三个一组，组成三组不同的权限
   - 第一组：文件拥有者具有的权限
   - 第二组：群组账号的权限
   - 第三组：非拥有这且非群组的其他账号的权限
     - r:read 可读性权限
     - w：write 可写权限
     - x：execute 可执行权限

3. 接下来的 数字代表有多少个文档名链接到该节点    
4. root 代表文件的拥有者    
5. root 代表文件所属的群组    
6. 代表该文件的大小，单位为byte    
7. 代表该文件的创建时间或这最近的修改时间    
8. 为该文件的名称
   - 若该文档名称前有“.”则说明给文档为隐藏文件

例：
> -rw-r--r-- 1 root root    0 Jun  1 18:34 test

为普通文件，文件的拥有这拥有 读写权限，没有执行权限，该文件的用有者为root用户，该文件所属的群组为root该文件群组拥有读权限没有写和执行权限，其他用户也只有读的权限，该文件的大小为0bytes,时间为6月1日18：34，名称为test

---

# 改变文件权限和属性的命令：
- chgrp:改变文件所有群组
- chown:改变文件拥有这
- chmod：改变文件的权限

#### 1、chgrp：
格式：chgrp [-R] dirname/filename ..   
参数R意味着可以递归的持续变更，变更该文件或目录下的所有文件。  
dirname文件所在目录，filename文件名称  
==可以在此处添加群组的创建等笔记==  
#### 2、chown:
格式：chown [-R] 账号名称 文件或目录 或者 chown [-R] 账号名称：组名 文件或目录  
> chown hubert test   将拥有这更改为Hubert  
> chown hubert:guest test   将拥有这该外Hubert，群组改为 guest  
> chown .guest test 修改群组为guest .+群组名  
#### 3、chmod:
格式：chmod [-R] xyz filename/dirname;  

权限名称 | 对应数值
---|---
r | 4
w | 2
x | 1  
- 使用数字来改变权限：
chmod 777 test
> -rwxrwxrwx 1 root root    0 Jun  1 18:34 test  

解读：三个777分别代表了，拥有者，群组，其他账号的权限数字。
r+w+x=4+2+1=7  
- 使用符号类型改变权限：  
u=user,g=group,o=others,a=all;  
> chmod u=rwx,go=rx test  
> -rwxr-xr-x 1 root root    0 Jun  1 18:34 test  

> chmod g+w test  
> chmod r-w test  
> -r-xrwxr-x 1 root root    0 Jun  1 18:34 test    

---

# 目录相关操作  

| Commond | explanation |
|---------|-------------|
| .       | 当前目录        |
| ..      | 上层目录        |
| -       | 前一个工作目录     |
| ~       | 用户所在的家目录    |
| cd      | 变换目录        |
| pwd     | 显示当前目录      |
| mkdir   | 创建目录        |
| rmdir   | 删除目录        |

#### cd (change directory)
cd [路径]  
#### pwd (Print Woking Directory)
pwd [option]
> -p:显示正确的完整路径而非链接文档的路劲即显示元路劲  
#### mkdir (make directory)  
mkdir [-mp]  
> -p:循环建立目录。创建所有目录;预设情况下目录是一层一层建立的，不能一次性建立多层目录，而-p参数可以实现一次建立多层目录  
> -m:在创建目录的时候直接配置目录的权限  

```
mkdir -m 744 test2
ll
drwxr-xr-- 2 root root 4096 Jun  2 15:06 test2
```
#### rmdir (remove directory)  
rmdir [-p] dirName  
> -p:连同上层目录，空目录一起移除（慎用）  
## 执行路劲变量PATH  
> echo $PATH 查看path变量的定义  

```
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
PATH="${PATH}:/dirName"  
将路径加入的path的搜索范围
# 文件与目录的管理  
#### ls
ls [option] fileNam/dirName
> -a:列出全部文件，包含隐藏文件  
> -d:仅列出目录本身，而不是列出目录内的文件数据  
> -l:列出文件的所有属性数据  
#### cp (copy)  
cp [option] source1 source2 ... destination  
> -a:  
> -i:若destination 已经存在则在覆盖是会询问是否需要覆盖  
> -p:连同文件的属性（权限，用户，时间）一起复制过去  
> -r:递归持续复制，用于复制整个目录  
#### rm (remove)  
rm [option]  
> -f:强制删除  
> -i:删除时候会询问  
> -r:递归删除，用于删除目录，非常危险  
#### mv (move)
mv [option] source desctination  
> -f:强制移动，若目标已经存在则会覆盖  
> -i:交互模式，若目标已经存在则会询问是否覆盖  
> -u:若目标已经存在而且source比较新才会更新  
----
# 文件内容操作 

| Commond | explanation   |
|---------|---------------|
| cat     | 由第一行开始显示文件内容  |
| tac     | 由最后一行开始显示文件内容 |
| nl      | 显示时同时显示行号     |
| more    | 一页一页的显示文件内容   |
| head    | 只查看头几页        |
| tail    | 只查看尾部数据       |
| od      | 以二进制的方式读取文件   |

#### cat (concatenate)
cat [option] filename
> -n:显示出行号，空白行也有行号

head [option] filename
> -n : 显示的行数(取出文件头几行)  

tail [option] filename  
> -n: 显示的行数(取出文件最后几行)  
----
#### AWK过滤排序
对文件第二列进行去重并输出
awk '{print $2}' access_log.2020-*.txt | awk '!a[$1]++{print}' > accesslog-0512.txt    
统计出第二列出现的次数并按从大到小的顺序输出
awk '{print $2}' access_log.2020-04-15.*.txt | sort | uniq -c | sort -nr
# 打包指令：tar
tar    
```
- [-z|-j|-J][cv][-f 需要新建的文档名称] filename  打包与压缩
- [-z|-j|-J][tv][-f 现有文档名称]                 查看文档
- [-z|-j|-J][xv][-f 现有文档名称] [-C 目录]       解压缩    
```
-c:建立打包文件    
-t:查看打包文件含有那些文档名
-x:解压打包或解压缩    
-z:进行gzip压缩或解压缩    
-j:进行bzip2压缩或者解压缩   
-J:进行xz压缩或者解压缩    
-C:指定解压是的路历    
-p:保留备份数据的原本权限与属性    
-P:保留绝对路劲    

  tar -cvf test.tar test # 得到test.tar备份文件 
  tar -zcvf test.tar.gz test # 得到test.tar.gz备份文件
  tar -jcvf test.tar.bz2 test # 得到test.tar.bz2备份文件
  tar -xzvf test.tar.gz # 解压文件

----
# vi
## 三种模式
- command mode : 一般指令模式，使用vi进入后的默认模式,实现删除，粘贴，复制功能
- insert mode  : 编辑模式，实现编辑功能，i-插入，o-新增一行,-r替换
- command-line mode : 指令行模式，[:/?]进入该模式，保存，离开，读取，搜索等功能    
**一般指令模式和 指令和模式不可直接转换**。   
**一般指令模式下快捷键**：

| 指令                   | 说明                               |
|----------------------|----------------------------------|
| h                    | 左                                |
| j                    | 下                                |
| k                    | 上                                |
| l                    | 右                                |
| ctrl + f             | 下一页                              |
| ctrl + b             | 上一页                              |
| 0                    | 该行的首字母                           |
| $                    | 该行的尾字母                           |
| G                    | 该文档的最后一行                         |
| gg                   | 该文档的第一行                          |
| nG                   | 该文档的第N行                          |
| n<Enter>             | 光标向下移动N行                         |
| /word                | 在文档中向下搜索word字                    |
| ?word                | 在文档中向上搜索word字                    |
| :n1,n2s/word/word1/g | 在第n1行与n2行之间搜索word并使用word1进行替换    |
| :n1,$s/word/word1/gc | 在全文中搜索word字段并使用word1替换，并在替换时进行询问 |
| :n1,$s/word/word1/gc | 在全文中搜索word字段并使用word1替换           |
| x,X                  | x-一行中向后删除一个字符，-X一行中向前删除一个字符      |
| dd                   | 删除一整行                            |
| yy                   | 复制一整行                            |
| ndd                  | 删除下面的N行数据                        |
| nyy                  | 复制下面的N行数据                        |
| p,P                  | p 为将已复制的数据在光标下一列贴上，P 则为贴在游标上一列   |
| u                    | 复原前一个动作                          |
| ctrl + r             | 重做上一个动作                          |
| .                    | 重复前一个动作                          |

#### 指令列模式
| 指令        | 说明         |
|-----------|------------|
| ：w        | 将编辑的数据写入硬盘 |
| ：q        | 离开vi编辑模式   |
| ：w!       | 强制将数据写入硬盘  |
| ：q!       | 强制离开       |
| ：wq       | 存储后离开      |
| ：set nu   | 显示行号       |
| ：set nonu | 不显示行号      |

#### 区块编辑
| 指令         | 说明                 |
|------------|--------------------|
| v          | 字符选择，会将光标经过的地方反白选择 |
| V          | 列选择，会将光标经过的列反白选择   |
| [ctrl + v] | 区块选择，可以用长方形的方式选择资料 |
| y,p,d      | 复制，粘贴，删除           |

#### 多文件编辑
| 指令     | 说明             |
|--------|----------------|
| ：n     | 编辑下一个文件        |
| ：N     | 编辑上一个文件        |
| ：files | 列出目前该vim所开启的文件 |

----
# 变量的设定：
- echo ${variable} / $variable 查看变量的内容    
- variable=content 变量赋值(注意等号左右不可又空格)   
- 当变量的内容中有空格或特殊字符时可以使用["]或者[']来修饰整个内容：    
- ["]中的特殊字符保持其原有的特性，而[']中的特殊字符则为字符串失去其原有特性    
- 在赋值的过程中如果赋值的内容来源于执行其它指令得到信息，则可以使用[`指令`]或者[$(指令)]来实现    
- **变量添加内容时可以使用以下格式来完成:["$variable"]或[${variable}]，例如：PATH="$PATH":/local/bin 或 PATH=${PATH}:/local/bin** 
- 变量取消的指令（删除）:unset--> unset varibale
- 修改PATH变量：PATH=$PATH:/home/;对PATH变量的修改只能维持在退出或者系统重新启动前。且只对当前的shell生效，如果需要子shell也生效则需要进行EXPORT到处。
### 环境变量功能
env:envirnoment 查看所有的环境变量    
set: 观察所有的变量(环境变量和自定义变量)    
export:将自定义变量转换成环境变量    
unset:删除环境变量    
### 默认常用环境变量：    
- HOME:当前用户的主目录
- IFS：shell用来将文本字符串分割成字段的一系列字符
- **PATH:shell查找命令的目录列表，由冒号分隔**。
### 定位系统环境变量
####  启动bash shell的三种方式
1. 登陆时作为默认登陆shell
2. 作为非登录shell的交互式shell
3. 作为运行脚本的非交互式shell

- 登录shell： 登陆时会从5个不同的启动文件中读取命令。
```
/etc/profile; //系统默认的主启动文件，系统上每个用户登录时都会执行这个启动文件。
$HOME/.bash_profile;
$HOME/.baserc;
$HOME/.bash_login;
$HOME/.profile
```
- 交互式shell登录。交互式shell不用访问/ect/profile，而回直接检查用户的主目录$HOME/.bashrc文件 *（1、用来查来/etc目录下通用bashrc问价，2、为用户提供一个定制自己的命名别名和私有脚本函数的地方）。*
- 非交互式shell。

#### 子shell
bash 命令创建一个新的子shell，子shell继承父shell 的部分环境变量

#### 后台模式 
- 在命令的后面加“**&**”符即可将命令置入后台运行。    
- 当使用&符号将命令置入后台进行运行的时候系统会返回命令在后台的作业号，和命令的进程号。    
- 使用**jobs**命令可以将在后台的所有作显示出来（所有用户的进程）

#### 环境变量持久化
1. 可以将改变的环境变量写进/ect/profile中（若更新系统的版本则，该文件很可能会本更新，修改过的环境变量将会丢失）
2. 推荐将在/etc/profile.d的目录中创建一个以.sh结尾的文件，把所有新的或修改的全局环境变量放于该文件中。
### 读取键盘变量，数组和宣称    
read [-pt] variable : 读取来自键盘的输入值赋予变量,参数p:后加提示语,参数t:等待的时间[秒]    
```
read -p "Please input your name:" -t 30 name
```
#### declare/typeset 宣告变量的类型    
declare [-aixr] variable : a将变量申明为一个数组,i将变量声明为一个int值,x将变量转为环境变量,r将变量声明为只读变量，不可更改，不可nuset    
#### array 数组型变量    
```
var[1]="small"
var[2]="middle"
var[3]="large"
echo "${var[1]},${var[2]},${var[3]}"
```    
#### 变量内容的编辑    
| 变量设定方式               | 说明                              |
|----------------------|---------------------------------|
| ${variable#关键词}      | 若变量内容从头开始的数据符合『关键词』，则将符合的最短数据删除 |
| ${variable##管检测}     | 若变量内容从头开始的数据符合『关键词』，则将符合的最长数据删除 |
| ${variable%关键词}      | 若变量内容从尾向前的数据符合『关键词』，则将符合的最短数据删除 |
| ${variable%%关键词}     | 若变量内容从尾向前的数据符合『关键词』，则将符合的最长数据删除 |
| ${variable/旧字符/新字符}  | 若变量内容符合『旧字符串』则『第一个旧字符串会被新字符串取代』 |
| ${variable//旧字符/新字符} | 若变量内容符合『旧字符串』则『全部的旧字符串会被新字符串取代』 |

#### 变量的测试、替换
| 设定方式             | str 没有设定          | str 为空字符串         | str 已设定为非空字符串  |
|------------------|-------------------|-------------------|----------------|
| var=${str-expr}  | var=expr          | var=              | var=$str       |
| var=${str:-expr} | var=expr          | var=expr          | var=$str       |
| var=${str+expr}  | var=              | var=expr          | var=$str       |
| var=${str:+expr} | var=              | var=              | var=$str       |
| var=${str=expr}  | var=expr,str=expr | var=expr,str不变    | var=$str,str不变 |
| var=${str:=expr} | var=expr,str=expr | var=expr,srt=expr | var=$str,str不变 |

#### 查看应用程序使用的环境变量：
cat /proc/pid/environ
### Linux设置编码格式
当前会话设置：export LANG=en_US.UTF-8    
文件路径：/etc/sysconfig/i18n     

```
LANG=en_US.UTF-8    
SUPPORTED="zh_CN.GBK:zh_CN.UTF-8:zh_CN:zh:en_US.UTF-8:en_US:en"    
SYSFONT="latarcyrheb-sun16"
```

| 变量        | 含义                   |
|-----------|----------------------|
| LANG      | 当前系统的编码              |
| SUPPORTED | 支持的编码格式最多两种(其中一种必为英文 |
| SYSFONT   | 所使用的字体               |

## 指令别名

```
alias 列出所有指令别名
alias lm='ls -al | more'  定义一个指令的别名
unalias 指令，取消指令别名
```
# linux 终端锁定 
ctrl+s 锁住终端，终端无任何反映
ctrl+q 解锁终端

# centos 下防火墙的使用
- firewall-cmd --state :查看防火墙的状态    
- firewall-cmd --list-ports :查看已经开放的端口    
- firewall-cmd --reload :重新启动防火墙    
- firewall-cmd --zone=public --add-port=80/tcp --permanent :开启端口    
---zone 作用域    
--add-port=80/tcp 开放的端口及协议    
--permanent 永久生效    
- systemctl stop firewalld.service：禁止防火墙
- systemctl disable firewalld.service：禁止防火墙开机启动
---
- service iptables status :查看防火墙的状态  
- service iptables stop：停止防火墙
- service iptables start：启动防火墙
- service iptables restart：重启防火墙
- chkconfig iptables off：永久关闭防火墙
- chkconfig iptables on： 永久关闭后重启
- vim /etc/sysconfig/iptables：开启端口
- -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
---

# 系统信息查看    

```
查看系统的发行版本：lsb_release -a
```
## LINUX 用户各项限制查看：ulimit -a
## 修改用户最大线成数：
在文件 /etc/security/limits.conf  文件末添加
```
username   hard   nproc  num    
username   soft   nproc  num
```
username-账户名称    
num-修改后的线程数    
## 修改用户最大文件数：
在文件 /etc/security/limits.conf  文件末添加    

```
username   hard   nofile  num    
username   soft   nofile  num
```
username-账户名称    
num-最大文件数    

## 设置当前环境下最大虚拟内存使用情况
sysctl -w vm.max_map_count=262144


**问题：linux :Can't locate Switch.pm**    

```
方案：yum install perl-Switch
```


**当前代理机上已经还有指纹则需要去下面的文件删除指纹：**    

```
vim /root/.ssh/known_hosts
```


**找出N天前的问题件并删除**    

```
find /d1/pricelog/price-api/ -mtime +7 -type f -exec rm -f {} \\;
```


分隔，gsub全部替换    


```
awk -F "productId=" '{print $2}' 641.txt |awk '{gsub(/shopId=/,""); print $0}' | awk '{gsub(/，/,","); print $0}' | sort | uniq > 642-result.txt
```


通过排序，去重完成文件内内容的去重

sort | uniq 

#  LINUX系统时间校准
ntpdate -u x.x.x.x(ipAddress)

ntp常用服务器地址：
- 中国国家授时中心：210.72.145.44
- NTP服务器(上海) ：ntp.api.bz
- 美国： time.nist.gov
- 复旦： ntp.fudan.edu.cn
- 微软公司授时主机(美国) ：time.windows.com
- 北京邮电大学 : s1a.time.edu.cn
- 清华大学 : s1b.time.edu.cn
- 北京大学 : s1c.time.edu.cn
- 台警大授时中心(台湾)：asia.pool.ntp.org

ps -ef | grep apache | awk '{ print $2 }' | sudo xargs kill -9    

# windows和Linux之间传输文件
windows和Linux之间传输文件，在安装有Xshell或者SecureCRT的工具上，可以直接使用rz或sz
- rz 从windows向Linux传输文件
- sz 从Linux向windows出传输文件    

ln -sf /direct  /vritual direct

vritual direct -> direct

 netstat -atnp | grep 22673 | grep "10.5.11.25" | wc -l

# 网络相关
### 执行DNS搜索（域信息搜索器）
dig @8.8.8.8 域名地址 +trace    
nslookup 域名地址
---

### Linux 抓包
#### tcpdump

```
tcpdump tcp port 3306 and host 10.5.11.43 -w ./ddpim_09.cap
```
- tcp:只抓取TCP协议的数据包    
- port: 根据端口号过滤    
- host: 根据主机IP过滤    
- -w: 将抓取的数据以文件的格式存储到后面的文件中

---


### LINUX DNS刷新
#### 查看是否可用：    
systemctl is-active systemd- resolved    
返回 active则表示可用      
#### 刷新DNS    
systemd-resolve --statistics    
systemd-resolve --flush-caches    
systemd-resolve --statistics

---
# LINUX 磁盘相关
### 查看磁盘容量
df -h 
du -h --max-depth=1

### 清理文件内容
true > assce.log | echo > assce.log| cat /dev/null > assce.log    

文件删除后，文件句柄没有被释放的问题。

描述：
    在linux上删除文件后，如果有进程一直在查看或者使用该文件则该空间并不会被真正的释放所以需要先停止使用的线程后改空间在会被释放。使用lsof | grep "delete"可以找到已删除但并未真正释放的线程及文件。    
    lsof director/filename

查看文件删除后因为程序继续运行而为释放的磁盘空间：    
**lsof | grep delete**

# 系统进程相关
### lsof-常用命令说明  
lsof filename 显示打开指定文件的所有进程    
lsof -g gid 显示归属gid的进程情况    
lsof +d /DIR/ 显示目录下被进程打开的文件    
lsof +D /DIR/ 同上，但是会搜索目录下的所有目录，时间相对较长    

---

lsof -i 用以显示符合条件的进程情况   
lsof -i[46] [protocol][@hostname|hostaddr][:service|port]    
46 --> IPv4 or IPv6    
protocol --> TCP or UDP    
hostname --> Internet host name    
hostaddr --> IPv4地址    
service --> /etc/service中的 service name (可以不只一个)    
port --> 端口号 (可以不只一个)  

### 进程停止
kill -[signal] [pid/GPID]     
当PID位负数的时候则表示位组ID,可以杀死一组进程    
常用的singal有1,9,15,18,19,20    
- 1-重新加载系统配置文件    
- 9-强制性杀死一个进程    
- 15-正常停止一个进程(可被拒绝)
- 18-重新运行被暂停的进程
- 19-暂停（不会被阻塞，拒绝）
- 20-暂停（但会被阻塞）    

