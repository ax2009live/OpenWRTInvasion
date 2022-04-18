# OpenWRTInvasion  openwrt MiRouter telnet frpc 开启 openwrt 路由器 ( 小米路由器 ) 的 telnet，开通内网穿透；青春版 mini 3C 3A 4Q 4 4C 4A千兆版 4A 等

### 我想要的功能
都需要进入系统的权限；

  1、从外网能访问路由器的网页端；<br>
  2、本地和外网能进入路由器系统里设置 - 通过内外穿透<br>
  3、能通过路由器访问路由器下的客户端 - 通过内外穿透

### 个人的观点：
小米路由器有手机app可以满足很多的功能远程启动、WIFI设置、vpn设置等等；<br>
其他固件是没有的，所以不建议去刷其他版本固件；我相信应有人跟我想法一样；<br>
有可能空间太小，不刷其他固件，无法DIY；



### 方法分析
查看代码，
<pre>cd /tmp
rm -rf busybox
chmod +x  /tmp/busybox
/tmp/busybox telnetd</pre>
可以发现，所有的目的就是通过 /tmp/busybox telnetd 开启 23 端口，以便用户能登录路由器；

busybox 通过漏洞上传，<br>
#### 不同 cpu 的架构需要相应的 busybox、frpc、nohup，不能混用

参考了 https://github.com/acecilia/OpenWRTInvasion 和网络上的帖子，发现都很复杂，且都是劝用户刷其他的固件；


如下是我的使用方法，供有需要的朋友参考；压缩档下载后解压
## 下面以小米路由器 4C 和 4Q 为例来说明，其他的大同小异，原理搞清楚，其他 cpu 版也类似；
### 小米路由器 4C： 文件里已内置了 busybox，无需网络，使用联发科的cpu，一般都是 MIPSEL 
  登录小米路由器 4C，复制 stok 后的内容 fcd63673ec1125a356bc1cf0b836abbc<br>
  打开的网址类似 http://192.168.31.1/cgi-bin/luci/;stok=fcd63673ec1125a356bc1cf0b836abbc/web/home#router<br>
  运行 start_telnet_联发科MIPSEL_无需网络.bat 输入路由器的 IP 地址和 stok 值，完成就可以 telnet 登录路由器了；
  ![image](https://user-images.githubusercontent.com/41521020/163879847-0b920f0d-cbd8-4701-b2ad-b6f345157c7f.png)

  
  重启后 /tmp 里的文件重置了，如需路由器重启后也可以登录：每次启动让路由器执行命令 busybox telnetd，和其他命令，<br>
  新建目录<pre> mkdir -p /userdisk/ddns/</pre>
  把 busybox 复制到目录 ddns 里:<pre> cp /tmp/busybox /userdisk/ddns/</pre>
  vi /userdisk/ddns/drop, 内容如下，退出保存后，赋予执行权限 chmod +x /userdisk/ddns/drop，
  <pre>
  #!/bin/sh
  /usr/sbin/iptables -I INPUT -p tcp -m multiport --dport 23,80 -s 10.0.0.0/24 -j ACCEPT
  # 10.0.0.0/24 为路由器登录 vpn server 后的网段，防火墙开通网段10.0.0.0/24的23.80端口后，
  # 外部可以通过 vpn 的方式来访问到此路由器的 23 80 端口；
  /userdisk/ddns/busybox  telnetd
  # 开启 tenet 服务</pre>
  打开 /etc/rc.local，然后在 exit 0 之前增加一行 /data/ddns/drop 保存退出，<br>
  16M 空间很小，如果想安装内网穿透工具，只能利用/tmp 来存放文件；<br>
  下载 nohup 文件
  <pre>wget -P /userdisk/ddns http://ax2009live.f3322.net:1680/other/frp/MR4C/nohup &&  chmod +x nohup</pre>
  /etc/rc.local 增加内容;
  <pre>
  wget -P /tmp  http://ax2009live.f3322.net:1680/other/frp/MR4C/frpc && chmod +x /tmp/frpc
  # 下载 frpc 客户端并赋予执行权限
  /userdisk/ddns/nohup /tmp/frpc -c /userdisk/ddns/frpc-1.ini > /dev/null 2>&1 &
  /userdisk/ddns/nohup /tmp/frpc -c /userdisk/ddns/frpc-2.ini > /dev/null 2>&1 &
  # 后台执行 frpc，
  </pre>  
### 小米路由器 4Q：我从 busybox.net 下载的 busybox 文件太大，所以无法上传，只能开机在线下载；CPU 架构 MIPS,跟联发科不一样；
![image](https://user-images.githubusercontent.com/41521020/163875371-9b1f9904-6996-4430-a1d4-9e752ef2cef8.png)

（ 如有朋友能提供文件大小比较小的 busybox，请邮件联系我，谢谢 ）<br>
运行 start_telnet_高通MIPS_需网络.bat 输入路由器的 IP 地址和 stok 值，完成就可以 telnet 登录路由器了；

打开 /etc/rc.local，然后在 exit 0 之前增加一行 /data/ddns/drop 保存退出，
  
  新建目录 <pre>mkdir -p /data/ddns</pre>
  下载 nohup 文件并赋予执行权限
  <pre>wget -P /data/ddns http://ax2009live.f3322.net:1680/other/frp/MR4Q/nohup &&  chmod +x nohup </pre>
  我设置的脚本 /etc/rc.local
  <pre> #!/bin/sh
  /usr/sbin/iptables -I INPUT -p tcp -m multiport --dport 23,80 -s 10.0.0.0/24 -j ACCEPT
  wget -P /tmp  http://ax2009live.f3322.net:1680/other/frp/MR4Q/busybox
  chmod +x /tmp/busybox
  /tmp/busybox telnetd
  
  wget -P /tmp  http://ax2009live.f3322.net:1680/other/frp/MR4Q/frpc
  chmod +x /tmp/frpc
  /data/ddns/nohup /tmp/frpc -c /data/ddns/frpc-1.ini > /dev/null 2>&1 &
  /data/ddns/nohup /tmp/frpc -c /data/ddns/frpc-2.ini > /dev/null 2>&1 &
  </pre>
  可能有人会发现，如果开机的时候网络有问题，没有下载 busybox 和 frpc 文件，则无法实现想要的功能;<br>
  所以我新建两个脚本文件来判断 busybox 和 frpc 是否存在，每十分钟检查一下:如文件不存在，则下载后执行命令；
  busybox-test
  <pre> #!/bin/sh
  if [ ! -f "/tmp/busybox" ];then
  wget -P /tmp  http://ax2009live.f3322.net:1680/other/frp/MR4Q/busybox
  chmod +x /tmp/busybox
  /tmp/busybox telnetd
  fi</pre>
  frpc-test 
  <pre> #!/bin/sh
  if [ ! -f "/tmp/frpc" ];then
  wget -P /tmp  http://ax2009live.f3322.net:1680/other/frp/MR4Q/frpc
  chmod +x /tmp/frpc
  /data/ddns/nohup /tmp/frpc -c /data/ddns/frpc-1.ini > /dev/null 2>&1 &
  /data/ddns/nohup /tmp/frpc -c /data/ddns/frpc-2.ini > /dev/null 2>&1 &
  fi</pre>
  crontab -e 增加如下两条命令；
<pre>*/10 * * * * /data/ddns/frpc-test >/dev/null 2>&1
*/10 * * * * /data/ddns/busybox-test >/dev/null 2>&1</pre>
  
  记得更改 root 的密码：passwd root<br> 再次开通 telnet 会把 root 密码清掉；<br>
  有眼尖的朋友可能发现 4C 和 4Q 存放文件的位置不一样；<br>
  df -h 后你才去选择存放文件的位置，每个路由器可能不一样；<br>
  rootfs 不可写 tmpfs 启动后文件丢失<br>
  ![image](https://user-images.githubusercontent.com/41521020/163874464-483746ef-5c5b-4fc3-b0df-5f921c9d097c.png)
  
  有朋友可能会问，为什么不使用 ssh 端口登录，因为 4C 和 4Q 都没有 ssh 的服务器程序 dropbear，所以无法开启 22 端口；<br>
  输入 which dropbear 能看到位置，说明可以开通 22 端口，就不需要 23 端口了；<br>
  ![image](https://user-images.githubusercontent.com/41521020/163878440-2ae7c733-737f-4531-ada5-9fa81441fd57.png)


  
  



