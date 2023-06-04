# NFS操作


```
安装只要 yum -y install nfs-utils即可

systemctl enable rpcbind
systemctl start rpcbind
systemctl enable nfs-server
systemctl start nfs-server

一、NFS服务器的设定

NFS服务器的设定可以通过/etc/exports这个文件进行，设定格式如下：

分享目录      主机名称或者IP(参数1，参数2）

/arm2410s   10.22.22.*(rw,sync,no_root_squash)

可以设定的参数主要有以下这些：

rw：可读写的权限； 
ro：只读的权限； 
no_root_squash：登入到NFS主机的用户如果是root，该用户即拥有root权限；
root_squash：登入NFS主机的用户如果是root，该用户权限将被限定为匿名使用者nobody； 
all_squash：不管登陆NFS主机的用户是何权限都会被重新设定为匿名使用者nobody。 
anonuid：将登入NFS主机的用户都设定成指定的user id，此ID必须存在于/etc/passwd中。 
anongid：同anonuid，但是变成group ID就是了！ 
sync：资料同步写入存储器中。 
async：资料会先暂时存放在内存中，不会直接写入硬盘。 
insecure：允许从这台机器过来的非授权访问。 
例如可以编辑/etc/exports为： 
/tmp　　　　　*(rw,no_root_squash) 
/home/public　192.168.0.*(rw)　　 *(ro) 
/home/test　　192.168.0.100(rw) 
/home/linux　 *.the9.com(rw,all_squash,anonuid=40,anongid=40) 
设定好后可以使用以下命令启动NFS: 
/etc/rc.d/init.d/portmap start (在REDHAT中PORTMAP是默认启动的） 
/etc/rc.d/init.d/nfs start 
exportfs命令
如果我们在启动了NFS之后又修改了/etc/exports，是不是还要重新启动nfs呢？这个时候我们就可以用exportfs命令来使改动立刻生效，该命令格式如下： 
exportfs [-aruv] 
-a ：全部mount或者unmount /etc/exports中的内容 
-r ：重新mount /etc/ exports中分享出来的目录
-u ：umount 目录 
-v ：在 export 的时候，将详细的信息输出到屏幕上。 
具体例子： 
[root @test root]# exportfs -rv  （全部重新export一次！）
exporting 192.168.0.100:/home/test 
exporting 192.168.0.*:/home/public 
exporting *.the9.com:/home/linux 
exporting *:/home/public 
exporting *:/tmp 
reexporting 192.168.0.100:/home/test to kernel 
具体例子：
[root @test root]#exportfs -au （全部都卸载了）
[root @test root]# vi /etc/exports

/home/soft 192.168.2.11(rw)
[root@localhost init.d]# nfs start
-bash: nfs: command not found
[root@localhost init.d]# ./nfs start
Starting NFS services: [ OK ]
Starting NFS quotas: [ OK ]
Starting NFS daemon: [ OK ]
Starting NFS mountd: [ OK ]

二、NFS客户端的操作： 
1、showmout命令对于NFS的操作和查错有很大的帮助，所以我们先来看一下showmount的用法 如无showmount命令 yum -y install showmount 安装之
showmount 
-a ：这个参数是一般在NFS SERVER上使用，是用来显示已经mount上本机nfs目录的cline机器。 
-e ：显示指定的NFS SERVER上export出来的目录。 
例如： 
showmount -e 192.168.1.136 
Export list for localhost: 
/tmp * 
/home/linux *.linux.org 
2、mount nfs目录的方法： 
mount -t nfs hostname(orIP):/directory /mount/point 
具体例子： 
Linux: mount -t nfs 192.168.0.1:/tmp /mnt/nfs 
[root@localhost /]# showmount -e 192.168.0.169
Export list for 192.168.0.169:
/home/opt/RHEL4U5 192.168.0.0/255.255.252.0
You have new mail in /var/spool/mail/root

mount -t nfs 192.168.0.169:/home/opt/RHEL4U5 /mnt/soft

 NFS报错：
mount.nfs: rpc.statd is not running but is required for remote locking.
mount.nfs: Either use '-o nolock' to keep locks local, or start statd.
mount.nfs: an incorrect mount option was specified

需要开启rpcbind
/etc/init.d/rpcbind start

 

NFS的防火墙配置

通过命令rpcinfo -p 可查看nfs使用的端口：



其中 portmapper，nfs 服务端口是固定的分别是 111和2049； 
另外 rquotad，nlockmgr，mountd 服务端口是随机的。由于端口是随机的，这导致防火墙无法设置。 
2、这时需要配置/etc/sysconfig/nfs 使 rquotad，nlockmgr，mountd 的端口固定。 找到以下几项，将前面的#号去掉。

LOCKD_TCPPORT=32803

LOCKD_UDPPORT=32769

MOUNTD_PORT=892

 

service nfs restart 

 

配置iptables,注意tcp与udp区分

-A INPUT -m state --state NEW -m udp -p udp --dport 111 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 924 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 33745 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 963 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 2049 -j ACCEPT

-A INPUT -m state --state NEW -m udp -p udp --dport 976 -j ACCEPT

-A INPUT -m state --state NEW -m udp -p udp --dport 979 -j ACCEPT

 -A INPUT -m state --state NEW -m tcp -p tcp --dport 111 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 927 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 33993 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 966 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2049 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 979 -j ACCEPT

service iptables restart
```