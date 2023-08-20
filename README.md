# 自用组网方案


## 组网需求

1. 公寓租客上网
2. 自用科学环境
3. 家庭成员环境
4. 同轴模拟监控转网络

## 解决办法

1. 公共环境需要安全稳定运行，如果主路由用 openwrt 我怕自己忍不住折腾它，ROS不会，直接采用 ikuai + 双openwrt (主路由i,旁路网关,DNS).
2. 自用科学直接采用最简单的方案买一个 CN2 GIA 线路，配置采用傻瓜插件 https://github.com/mack-a/v2ray-agent ,客户端用的是 HomeProxy .
3. 家庭成员需求包括备份照片与视频，小孩使用 chatgpt 等，同时需要屏蔽 18+ 内容，就需要自建 DNS 用的方案是 Mosdns .
4. 老式模拟摄像头不想布线，当时采用集中供电，所以有 一条供电线 + 同轴  可以拼出4根线，做一个 4 芯网线可以走 100M .

## 硬件方案

- 电信双光纤入户 1000M + 300M
- HP T620 Plus + HP NC364T 4 port + Mini PCIe 千兆(拆掉VGA接口安装)
- TP-Link 24 port 千兆交换机
- 绿联双盘位外置硬盘座 + 2块旧硬盘 （轻NAS盘）
- TP-LINK TL-NV6116C-L + 4T旧硬盘 + 淘宝山寨网络400W摄像头

## 设置与注意点

- 主路由：
	安装爱快，双线接入4口网卡 1,2 口，设置 wan1 wan2 
	设置LAN1 桥接其它 4 个端口
	设置DNS地址分别为 2个光猫的内网IP , <font color="#c00000">DNS 代理不要开启</font>
	设置DHCP 网关为 爱快LAN1地址，DNS为 自建DNS服务器址。
	设置 静态IP 需要科学上网的网关为旁路网关(旁路由)
	新建2个虚拟机，CPU选1核，内存512就行
- 旁路网关(旁路由)：
	安装 https://downloads.immortalwrt.org/ 最新版就行
	在软件包删除 dnsmsaq 即在网络选项中没有DHCP/DNS 这一项
	接口建议使用eth0，可以删除桥接br-lan
	LAN接口设置网关为 主路由地址，DNS地址为 自建DNS服务器
	在软件包中安装 HomeProxy 
	HomeProxy 设置DNS为 <font color="#c00000">WAN下发DNS</font> ,国内DNS默认<font color="#c00000">禁用</font>
	添加节点后启用
- DNS服务器设置
	安装 https://downloads.immortalwrt.org/ 
	同样删除 dnsmsaq
	接口设置网关同上，DNS地址为 127.0.0.1 (MOSDNS设置前为运营商地址)
	软件包安装 MosDNS 软件包可能没有LUCI https://github.com/sbwml/luci-app-mosdns 这里下载
	MosDNS 设置 <font color="#c00000">监听端口为 53 ，目的是接管默认DNS端口</font>
	上游DNS随便选，远程DNS即科学DNS我设置是1.1.1.3
	MosDNS设置黑名单屏蔽18+内容，具体列表在清单中

## 总结

这个方案只适合我，不一定适合所有人，希望能有所启发。

爱快的DNS在不开启代理的情况下，只为爱快本身使用，也就是当客户机的DNS地址指向爱快时，才发生作用。
想避免这个情况就开启爱快代理，模式选择第三方，地址填MOSDNS的。

双OP模式，主要就是为了稳定，再就是防止DNS泄漏，通过 https://ipleak.net 检测。只要检测不到你所在国家的地址即没有泄漏。

同轴改网线，单纯就是为了节约成本，事实证明可行，很多消费其实是无意义的。

