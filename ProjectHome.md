# OpenWrt 使用 smarthosts 和 autoddvpn 实现完美翻墙 #

本文是告诉你如何使用 OpenWrt + smarthosts + autoddvpn 实现完美翻墙的方法。

## 1.需要具备的环境前提 ##

很多时候我们在网上搜索到一堆操作方法，但是如果你照着操作的话你就会发现在某步根本走不下去。比如网文告诉你操作一个命令，结果你机器里面根本没有这个命令。种种这些往往都是因为你和作者有的环境不一样，所以很多操作你就根本没法做。我把 **需要具备的环境前提** 放在一开头，目的就是希望让你检查自己的环境，如果你不具备这些环境的话，下面的操作你可能就会遇到很多问题。

| **前提要求** | **我的环境** | **其它可能兼容的环境** | **解决方法** |  **备注** |
|:-----------------|:-----------------|:--------------------------------|:-----------------|:------------|
| 路由器 | Linksys WRT160NL | 任意可以刷 openwrt 的路由器应该都可以 | taobao 你懂的 | 见下面备注   |
| 系统 | openwrt backfire 10.03.1（经测试最新的 attitude\_adjustment 可以完美使用本文的方法）  | 其它老版本理论上也差不多，你需要自己试试了 |  | 见下面备注   |
| 外挂硬盘 | 我的路由器带USB口，所以挂了一个硬盘在 /mnt 目录下 | 没有硬盘的用路由器自身的Flash存储也行，不知道空间是否够大 | 买个带USB口的路由器，这样就可以挂硬盘 |  见下面备注 |
| opkg | openwrt 系统自带  |  |  | 你需要清楚如何使用 opkg 安装软件   |
| 需要已经安装的软件 | dnsmasq, subversion, pptp  |  |  | 见下面备注  |
| 翻墙 VPN | 淘宝上买的 PPTP VPN  | 只要是 PPTP VPN 就行  | taobao 你懂的 | 后面 autoddvpn 翻墙需要用 |
| 本文写作时间 | 2012-12-08 更新于 2015-01-27 |  |  | 写这个目的是给你一个参考，或许你是 N 年后看到这篇文章，那么有可能里面写的东西已经不能用了   |
| 技术支持| 棒主妇开源 |  |  | 感谢棒主妇开源提供技术支持 http://www.bzfshop.net   |


**路由器备注：**自己去淘一个路由器吧，一般淘宝搜索 Linksys 路由器基本上都可以刷 OpenWrt（我还真没见过不能刷 openwrt 的  linksys 路由器）。其它品牌一般采用 Broadcom 无线解决方案的路由器也可以刷 openwrt。那种支持 dd-wrt 或者 tomato 的路由器基本上  99% 都可以刷 openwrt。国产的一些 TP-Link, D-Link , 水星 ... 有大概 50% 能刷 openwrt ，有一些不能刷。具体情况你需要自己解决了。如果是从来没有接触过 openwrt 的新手，那你可以去论坛淘一个  http://www.openwrt.org.cn/bbs/forum.php 。

**系统备注：** 这里下载 OpenWrt 的系统 http://downloads.openwrt.org/backfire/10.03.1/

**外挂硬盘：** 之所以外挂一个硬盘是因为路由器自身的 flash 只有 8MB太小了，我们需要下载很多代码之类，一般刷完 openwrt 你的剩余空间只有 2MB，我很怀疑你是否能再下载更多的东西放在这个里面。我后面所有的下载文件都会放到 **/mnt** 下面，如果你没有外挂硬盘。那你需要自己决定这些东西放在哪里了，并且确定你是否有足够的剩余空间来放下所有的文件。

**软件环境：**  dnsmasq 一般安装好 openwrt 就已经有了，所以基本上你不需要另外安装。

下面安装其它软件，前提是你已经配置好了 opkg ，至于 opkg 怎么配置需要你自己去 openwrt 相应的论坛找了。

Subversion 安装：
```
opkg install subversion-client
```

PPTP 安装：

```
## 后面这个是 luci 图形配置界面，不是必须，没有也行，只不过方便你以后使用而已，本文中本没有用到这个东西
opkg install pptp luci-proto-pptp  
```

## 2. 使用smarthosts 简单翻墙 ##

有些时候我们上不了网，并不是对方 IP 被封了，而是因为我们的 DNS 被污染解析到一个错误的 IP 地址上去了。既然 IP 地址解析到错误的地方去了，你自然连接不上。解决这个问题的办法很多，网络上很多文章一般都是教你修改 hosts 文件。
感谢 smarthosts 工程的兄弟，他们很辛苦的把很多被污染的 IP 地址找到，然后给我们打了个包。我们需要做的只是简单的使用 OpenWrt 的 dnsmasq 调用这个配置就行。

**（1） 下载 smarthosts 工程**

smarthosts 工程在这里：http://code.google.com/p/smarthosts/

使用 SVN 下载代码（为什么要用 SVN 下载后面会说，你先按照操作就行）
```
 ## 进入目录，注意：我在 /mnt 下面是挂了一个硬盘，所以有足够的磁盘空间，
cd /mnt/etc 

## 用 SVN 下载代码
svn checkout http://smarthosts.googlecode.com/svn/trunk/ smarthosts
```


**（2）让 dnsmasq 使用 smarthosts 的配置**

```
## 注意： 由于 smarthosts 更新 dnsmasq.conf 文件非常慢，我们直接使用 hosts 文件
## dnsmasq.conf 文件加一句，加在文件最后面

## 使用中国 IP，优点：快，缺点：容易被封
addn-hosts=/mnt/etc/smarthosts/hosts

## 或者使用美国 IP，优点：不容易被封，缺点：慢
## addn-hosts=/mnt/etc/smarthosts/hosts_us

## 注意：上面 中国IP 和 美国IP 只能选一个，千万别同时用，否则就混乱了

## 重启 dnsmasq
/etc/init.d/dnsmasq restart

```

重启一下你的电脑，或者禁用一下你的网卡再重新启用一次（目的是清除你电脑上已有的 DNS 缓存，确保之后用的都是 dnsmasq 取得的地址）。

OK，大功告成！！！
就这么简单？
是的，就这么简单！！！
你现在试试访问  https://www.facebook.com ，看看是不是能上去了？ （注意是： HTTP\*S**） 包括你现在试试访问 Google , GMail 是不是明显很快了？**

大部分网络这个时候应该能明显感觉到更快了，而且能上 https://www.facebook.com 和 https://www.twitter.com 了。

你试了，还是不行？
OK，不是楼主骗了你，只能说明你的网络可能被封的很严重，那么你需要继续往下看， autoddvpn 才能挽救你。


## 3. 使用 autoddvpn 复杂翻墙 ##

DNS 污染只是一种简单的封网方法，既然简单，所以对应的解封方法也简单，smarthosts 就是用于解决这种简单问题的。

对于一些更加严厉的内容，比如我 1 岁时候光着屁股挂着蛋满大街跑，结果有好事者拍了视频传到某视频网站上去了，点击率还颇高。
今天我是一个 体面、光鲜、永远正确、永远英明 的领导者，这视频绝对是黑暗的资本主义社会要故意要抹黑我的伟大英明的凶器，于是我下令不只是污染DNS，还要直接在电信接口把  IP 都封掉，对于这种国家特级机密绝对不能让屁民们看到。

这种情况下，smarthosts 也没办法，只能使用  VPN 来翻墙了。

autoddvpn : http://code.google.com/p/autoddvpn/

autoddvpn 是一个在 DD-WRT 平台用于自动翻墙的程序（别急，我会教你怎么在 openwrt 里面完美使用它）。

我想很多人都有经验，如果我挂了 VPN 翻墙，墙是翻了，但是上国内的网络就变得很慢了，非常苦恼。
是的，autoddvpn 就是解决这个问题的，简单的说： 国内流量不走VPN，翻墙流量才走VPN，实现你国内访问快，同时又可以随时翻墙的苦恼。
具体我不多说，你可以自己去 autoddvpn 项目看介绍。

在这里感谢 autoddvpn 的创建者为我们提供了一个这么好的工具。

好了，下面我们开始说怎么在 openwrt 下面使用 autoddvpn，老套路

**（1） 下载 autoddvpn工程**

使用 SVN 下载代码（为什么要用 SVN 下载后面会说，你先按照操作就行）
```
 ## 进入目录，注意：我在 /mnt 下面是挂了一个硬盘，所以有足够的磁盘空间，
cd /mnt/etc 

## 用 SVN 下载代码
svn checkout http://autoddvpn.googlecode.com/svn/trunk/  autoddvpn
```

**（2）让 dnsmasq 使用 autoddvpn 的配置**

```
## dnsmasq 一般 openwrt 系统自带，不需要另外安装，理论上安装好你就应该有  /etc/dnsmasq.d  这个目录
## 如果你的系统没有 dnsmasq.d 目录，那么你需要自己 mkdir 建立一个，然后修改/etc/dnsmasq.conf 增加了一条 conf-dir=/etc/dnsmasq.d
 
cd /etc/dnsmasq.d  

## 让 dnsmasq 使用 autoddvpn ，建立一个软链接
ln -s  /mnt/etc/autoddvpn/grace.d/dnsmasq_options  ./autoddvpn.conf

## 重启 dnsmasq
/etc/init.d/dnsmasq restart

```


**（3）从 autoddvpn 生成 openwrt 能用的配置**

前面说了 autoddvpn 是为 dd-wrt 写的，所以 openwrt 不能直接使用。
需要需要生成一个能给 openwrt 使用的文件。

```
# generate route
cd /mnt/etc/autoddvpn/grace.d/
/bin/grep "^route add" vpnup.sh |  /bin/sed "s/ gw / dev /g" | /bin/sed "s/VPNGW/1/g" > /mnt/etc/autoddvpn/gen_route.sh
```

先别疑惑，后面我会告诉你这个 gen\_route.sh 文件怎么用，现在你只要知道需要生成这么个文件就行。

如果你有兴趣，可以看看这个自动生成的  gen\_route.sh 文件，它的内容大致如下：


```
route add -host 8.8.8.8 dev $1
route add -host 8.8.4.4 dev $1
route add -host 208.67.222.222 dev $1
route add -net 174.36.30.0/24 dev $1
route add -net 184.73..0.0/16 dev $1
##########  下面省略无数行  太多了   ##############
```


**（4）配置你的 PPTP VPN**

首先你需要有一个可以翻墙的 pptp vpn 账号 （taobao 你懂的），如果你没有 pptp vpn ，那么你不用往下看了。
因为后面必须要用到 pptp vpn。

PPTP VPN 需要配置 /etc/ppp/options.pptp 文件，下面是我的这个文件的配置供你参考：

```
debug
kdebug 1 
lock 
noauth 
nobsdcomp 
nodeflate
idle 0
defaultroute 
lcp-echo-interval 3
lcp-echo-failure 5
maxfail 0

## 下面的配置非常重要，这里是指明加密方式，很多时候如果你的 pptp 死活连接不上，那么 99% 的原因是下面的配置不对

mppe required
mppe stateless
refuse-pap
refuse-chap
refuse-eap
refuse-mschap
```

修改  /etc/config/network   配置文件，在末尾增加：

注意：下面的名字 GFWVPN 不要改，因为我后面的程序是写死这个名字的，你要是改了，后面程序就不工作了，
除非你清楚整个 autoddvpn 的原理，否则下面 defaultroute 0 之类的东西你别乱改，切记切记。
如果你一定要改，那确定你自己已经清楚所有的程序，那么你可以随便修改。

```
config 'interface' 'GFWVPN'
        option 'ifname' 'GFWVPN'
        option 'proto' 'pptp'
        option 'username' '你的用户名'
        option 'password' '你的密码'
        option 'defaultroute' '0'
        option 'peerdns' '0'
        option 'server' 'VPN 服务器地址'
```

注意这里的  option 'defaultroute' '0' ,  option 'peerdns' '0' 设置，这里设置 VPN 不要修改路由器的缺省路由。
即使 VPN 联通了，缺省的路由也还是走你的 adsl ，不会走 VPN。 那如何让国外流量走 VPN 呢？ 继续往下看 ...

现在重启你的路由器，重启之后你应该能用  ifconfig 看到 PPTP VPN 已经连接上了，
大概如下：

```
pptp-GFWV Link encap:Point-to-Point Protocol  
          inet addr:10.1.1.84  P-t-P:10.1.1.2  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1400  Metric:1
          RX packets:32001 errors:0 dropped:0 overruns:0 frame:0
          TX packets:25977 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3 
          RX bytes:20291692 (19.3 MiB)  TX bytes:6807209 (6.4 MiB)
```

如果你的 ifconfig 看不到这个信息，那只能自己去看系统日志 （luci 里面的 system log），自己想办法弄通你的 pptp vpn。

注意： 你必须保证你的 PPTP VPN 已经可以连接上了，否则下面的步骤你走不下去的。

注明： 如果你是按照这里的方法配置的VPN，那如果VPN断掉 OpenWrt 系统会自动重连所以不用担心

**（4）翻墙、翻墙**

你已经从 autoddvpn 生成了我们需要的文件 gen\_route.sh 了，并且你的 PPTP VPN 已经成功连接上了。
OK，恭喜你，从现在开始，你可以翻墙了。

当 PPTP VPN 连接上之后， 它会自动调用一个回调脚本，这个脚本就是翻墙的关键。
我写了一个脚本 pptp-GFWVPN （是的，这个文件名字和 VPN 名字一模一样）

代码可以在这里下载：http://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/

你需要把这个脚本放到 /etc/ppp/ip-up.d 下，并且确保它有可执行权限

注意：如果你的系统没有 ip-up.d 目录，那你需要自己建立，或者干脆就把代码放到 ip-up 脚本中去，具体怎么操作就需要根据你的系统的具体情况而定了

```
## 复制到目录下
cp  pptp-GFWVPN   /etc/ppp/ip-up.d

## 增加可执行权限
cd   /etc/ppp/ip-up.d
chmod a+x pptp-GFWVPN

```


pptp-GFWVPN  脚本如下：

```
#!/bin/sh

set -x
export PATH="/bin:/sbin:/usr/sbin:/usr/bin"

## configure device name here
VPNDEVICE=pptp-GFWVPN

case $1 in
     "pptp-GFWVPN")
     	goto addRoute
      	;;
      *)
        # others
        exit 0
        ;;
esac

addRoute:

## add route which was generated from autoddvpn
/bin/sh /mnt/etc/autoddvpn/gen_route.sh $VPNDEVICE 

sleep 10

## OpenWrt would do this automatically
## /usr/sbin/iptables -t nat -A POSTROUTING -o $VPNDEVICE -j MASQUERADE
## /sbin/route add -net $REMOTESUB netmask $REMOTENET dev $VPNDEVICE

##### begin batch route #####      

# your own custom IP 
#route add -host 192.168.200.123 dev $VPNDEVICE
#route add -net 192.168.0.0/16  dev $VPNDEVICE

##### end batch route #####      

## restart dnsmasq
/etc/init.d/dnsmasq restart

exit 0

```

从上面的脚本你可以看到，我们调用了 /bin/sh /mnt/etc/autoddvpn/gen\_route.sh $VPNDEVICE  ，是的，这里我们就是用了从 autoddvpn 生成的文件

现在，再次重启你的路由器，等重启完成之后你试试执行   route -n ，看看是不是多出很多额外的路由来了，大概如下：

```
66.102.0.0      0.0.0.0         255.255.240.0   U     0      0        0 pptp-GFWVPN
64.233.160.0    0.0.0.0         255.255.224.0   U     0      0        0 pptp-GFWVPN
208.117.224.0   0.0.0.0         255.255.224.0   U     0      0        0 pptp-GFWVPN
72.14.192.0     0.0.0.0         255.255.192.0   U     0      0        0 pptp-GFWVPN
173.194.0.0     0.0.0.0         255.255.0.0     U     0      0        0 pptp-GFWVPN
75.126.0.0      0.0.0.0         255.255.0.0     U     0      0        0 pptp-GFWVPN
69.63.0.0       0.0.0.0         255.255.0.0     U     0      0        0 pptp-GFWVPN

##########  此处省略无数行，太多了 ##############
```

如果你能看到上面的结果，好了，恭喜你，你已经成功翻墙了，现在你可以试试访问  http://www.youtube.com 看看是不是能够成功访问了？

如果还是不行，--_-- 别怪我，你换个 PPTP VPN 吧，肯定是买的这个 VPN 不行。_


## 4. 翻墙需要与时俱进 ##

大家从上面可以看到，我们其实是利用了 smarthosts  和 autoddvpn 里面记录了很多的 IP 地址和路由，但是所有人都知道，
这些信息是会不断变化的，所谓 道高一尺，魔高一尺二，虽然是比我们高了，但是也就是个二，所以不用怕它。

怎么保持持续的更新呢？ 想起我们前面用 svn 下载代码了吗？ 是的，你可能已经猜到了，用  svn up 来每天更新同步代码。

好了，我写了一个脚本用于每天同步代码。这个脚本 UpdateGFW 设定 cron 为每天凌晨 2:00 自动执行一次，更新代码，这样就可以保持
你的翻墙永远是最新的。

代码可以在这里下载：http://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/

UpdateGFW 脚本如下：

```
#!/bin/sh

### for smarthosts project
cd /mnt/etc/smarthosts
/opt/usr/bin/svn up

###  for autoddvpn project

# update svn
cd /mnt/etc/autoddvpn
/opt/usr/bin/svn up

# generate route
cd grace.d
/bin/grep "^route add" vpnup.sh |  /bin/sed "s/ gw / dev /g" | /bin/sed "s/VPNGW/1/g" > /mnt/etc/autoddvpn/gen_route.sh

/etc/init.d/dnsmasq restart
```

有了上面这个自动更新脚本，你再也不怕翻不了墙了。


## 5. 实现自己自定义的访问 ##

有些你想访问的网站，但是并不在 smarthosts 和 autoddvpn 里面， 怎么办呢？

没问题，你可以自己动手，下面是自己操作的方法：

假设你想访问一个网站  www.aaa.com ，但是 smarthosts 和 autoddvpn 都没有包含它，那么你自己动手操作：

**（1）dnsmasq 建立一个自己的文件**

添加一个自己的 dnsmasq 配置文件（如果已经有了，请忽略这步）

```
cd /etc/dnsmasq.d/

vim myown.conf

```

myown.conf 文件里面加入：

```
server=/aaa.com/8.8.4.4
```

然后重启 dnsmasq 服务  /etc/init.d/dnsmasq restart


**（2）查询到网站的真实 IP**

```
nslookup  www.aaa.com  8.8.4.4 

## 得到如下结果
192.168.200.123
192.168.100.123

```

**（3）把 IP 加入到  pptp-GFWVPN 脚本中去**

就是我们放在 /etc/ppp/ip-up.d/ 目录下的  pptp-GFWVPN

```

##### begin batch route #####      

# your own custom IP 
route add -host 192.168.200.123 dev $VPNDEVICE
route add -host 192.168.100.123  dev $VPNDEVICE

##### end batch route #####      

```

断掉路由器上你的 VPN 链接，然后重新连接一下 VPN  （或者干脆重启一下路由器），现在你就可以顺利的访问 www.aaa.com 这个网站了。

## 6. autoddvpn 引起的一些问题 ##

由于 autoddvpn 带了一个 dnsmasq 的配置文件，这个里面写死了 facebook 之类的 IP
如果一旦这个 IP 不起作用了，而 autoddvpn 又没有及时更新的话，那你就上不去了
所以我现在一般 不用 autoddvpn 自带的 dnsmasq 配置文件，而是自己写一个

代码可以在这里下载：http://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/

下面是我的 dnsmasq 配置文件，其它的根据你自己的需求往里面加吧

```
## server configuration

server=/google.com/8.8.4.4
server=/facebook.com/8.8.8.8
server=/fbcdn.net/8.8.4.4
server=/twitter.com/8.8.4.4
server=/apple.com/114.114.114.114
server=/youtube.com/8.8.4.4
server=/ytimg.com/8.8.4.4
server=/imageshack.us/8.8.4.4
server=/books.com.tw/8.8.4.4
server=/book.com.tw/8.8.4.4
server=/blogspot.com/8.8.4.4
server=/appspot.com/8.8.4.4
server=/wikipedia.org/8.8.4.4
server=/ifeng.com/8.8.4.4
server=/dropbox.com/8.8.4.4
server=/t.co/8.8.4.4
server=/bit.ly/8.8.4.4

```


## 7. Twitter 上关注我 ##

我的 twitter :  qiang\_yu\_china

亲，如果你觉得这篇文章对你有用，请给个好评吧， 谢谢！


本来想把这个文章同步发到 http://www.openwrt.org.cn/ 上，结果注册帐号，然后等 1 个小时，最终我的帐号还是没有权限发贴，甚至没有回帖的权限
，所以想想就算了，等哪位有心的朋友看到这篇文章，请帮忙贴到 openwrt.org.cn 上去吧


## 8. shadowsocks ##

距离我上次写翻墙已经过去2年多了，期间经历着和 GFW 不断的斗智斗勇，也不断的变换新的方法。最近非常忙，没有太多时间静下来好好的写文章了，只能草草的留下一些或许你们用得上的东西

最近一直在使用 shadowsocks 作为翻墙的工具，用于替代 VPN 和 SSL Tunnel。仔细测试过各方面的性能，shadowsocks 可以说性能非常卓越，尤其是打开多个网页高并发的时候，SSL Tunnel 容易卡顿， VPN 容易掉线， shadowsocks 响应确非常迅速，很满意它的性能

用 shadowsocks 的方法很简单

1. 你买一个便宜的 VPS ，搭建一个 shadowsocks server ，注意，越少人用的越好，如果名声在外连卖菜的老婆婆都知道的VPS就别买了，买一个封一个没意义

2. 配置你的 OpenWrt 路由器，安装 shadowsocks-libev 这个客户端，和服务器建立链路

3. 用一个 PAC 文件配置到你的浏览器代理里面，PAC 里面指定哪些网站走 shadowsocks 代理，哪些不走，这样就实现了透明翻墙，你只需要维护这个 PAC 文件就可以了 （不懂 PAC 的自己 Google ，主流浏览器都支持 PAC，我推荐用 Chrome + Proxy SwitchyOmega + PAC 文件）

openwrt-shadowsocks-libev  官方提供的下载只支持 attitude\_adjustment 版本以上的 OpenWrt，而我至今还只是用着  openwrt backfire 10.03.1 ，遍寻网上发现大家都没法在 10.03.1 上跑起来，所以这里我提供一个我自己编译好的包给大家下载，让你的 10.03.1 也可以跑 shadowsocks。

我自己编译的 shadowsocks-libev 包，在  openwrt backfire 10.03.1 下可以完美运行，这个包唯一依赖的就是系统自带的  libopenssl 库，没有别的依赖了，所以在安装这个包之前，你需要先安装 libopenssl 库

```

## 安装 10.03.1 系统自带的 open ssl 库，版本应该是 0.9.8r-1
opkg install libopenssl

```

用于透明翻墙的包，需要搭配别的程序一起使用

https://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/shadowsocks-libev-spec_2.1.1-1_ar71xx.ipk

最简单的 shadowsocks 客户端程序，我现在用的就是这个，简单好用够用

https://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/shadowsocks-libev_2.1.1-1_ar71xx.ipk

PAC文件的例子，你看看就懂了

https://code.google.com/p/openwrt-smarthosts-autoddvpn/source/browse/gfw-proxy.pac


**最后：**

谨以此文感谢我们永远 伟大、英明、正确、先进、无敌 的 GFW，正是在和 GFW 做斗争的无数个日日夜夜中，我们的翻墙技术得到了不断的提高，
我们的网络操作水平得到了常足的进步，而这些进步又奉献在社会主义的经济建设中。可以这么说，中国的 GDP 是我们的功劳，同时也是 GFW 的
功劳，我们衷心的感谢这个一直陪伴我们成长的 好导师、好朋友。