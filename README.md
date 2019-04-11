# network-set

**配置电脑路由实现有线进内网，无线访问外网**

在有些ctf线下比赛的时候，主办方会给根网线，题目和提交平台都是通过该网线进行进入内网访问的

但是插上网线就没法百度查资料了，连续的插拔网线都很麻烦，所有进行本地路由表的配置很有必要

首先把有线的网线插好，无线的热点连好



我这里测试的时候内网ip是`210.29.64.133`

```
tracert 210.29.64.133
```

先插个网线瞧瞧路径

```
  1    16 ms     1 ms    <1 毫秒 114.213.64.1
  2    <1 毫秒   <1 毫秒   <1 毫秒 210.29.79.141
  3     2 ms     2 ms     1 ms  210.29.79.22
  4     1 ms     1 ms     1 ms  210.29.64.133
```

走的是`114.213.64.1`这个默认网关

在不拔掉网线的情况下，再连上热点，命令窗口下查看路由表

```
route print
```

获取有线的默认网关信息，和无线的默认网关信息,这里主要看网络目标，网络掩码，网关

```
网络目标        网络掩码          网关       接口   			跃点数
0.0.0.0        0.0.0.0     114.213.68.1   114.213.68.171     35
0.0.0.0        0.0.0.0     192.168.43.1   192.168.43.195     35
....
```

举个我电脑的情况，114是内网的，192是我热点的（实际操作可以先连一个查看下，再连另一个就可以区分谁是谁了）

接下来删除这2个默认网关

```
route delete 0.0.0.0 mask 0.0.0.0
```

填加路由表规则

```
route add 目标网段 mask 目标网段子网掩码 通过的网关
```

把外网（也就是无线热点）设置成默认网关

网卡也要设置对，理由`route print`可以查看网卡的代号

```
 16...00 ff bb 38 71 ba ......TAP-Windows Adapter V9
 17...00 bb 60 62 62 d2 ......Microsoft Wi-Fi Direct Virtual Adapter
  2...02 bb 60 62 62 d1 ......Microsoft Wi-Fi Direct Virtual Adapter #2
 19...80 fa 5b 61 49 ed ......Realtek PCIe GbE Family Controller
 12...00 50 56 c0 00 01 ......VMware Virtual Ethernet Adapter for VMnet1
  7...00 50 56 c0 00 08 ......VMware Virtual Ethernet Adapter for VMnet8
 14...42 53 ee 61 e2 2d ......Intel(R) Wireless-AC 9462
 22...00 bb 60 62 62 d5 ......Bluetooth Device (Personal Area Network)
  1...........................Software Loopback Interface 1
```

wifi是2，网线是19

```
route add 0.0.0.0 mask 0.0.0.0 192.168.43.1
```

然后把目标的网段添加成静态路由表,注意是网段

然后网卡要设置对，如果有时候不对的话可以这么设置[IF 19]这里网线是19

```
route add 210.29.64.0 mask 255.255.252.0 114.213.64.1 [IF 19]
```

接下来就可以为所欲为了



参考链接：

<https://www.cnblogs.com/fetty/p/4524029.html>

<https://blog.csdn.net/lhjllff12345/article/details/75339922>
