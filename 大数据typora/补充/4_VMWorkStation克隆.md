1、修改网卡配置，输入 vi /etc/sysconfig/network-scripts/ifcfg-eth0

出现类似如下配置信息：

DEVICE=eth0 #网卡接口名称
HWADDR=00:0C:29:DA:F7:6C #网卡设备MAC地址
TYPE=Ethernet #网卡类型
UUID=4961e853-5a5d-4980-bcd1-a29d996ba7b9
ONBOOT=yes #系统启动时是否自动加载
NM_CONTROLLED=yes
BOOTPROTO=static #启用地址协议 --static:静态协议 --bootp协议 --dhcp协议
IPADDR=192.168.111.11 #网卡IP地址
NETMASK=255.255.255.0 #网卡网络地址
GATEWAY=192.168.111.2 #网卡网关地址

删掉HWADDR（物理地址）和UUID，重启系统会自动创建，根据个人情况修改成如下配置

DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.111.19
NETMASK=255.255.255.0
GATEWAY=192.168.111.2

2、修改主机名称，输入 vi /etc/sysconfig/network

改为如下配置，我新克隆的主机名称是node9

NETWORKING=yes
HOSTNAME=node9

3、删除网卡相关信息，输入 rm -rf /etc/udev/rules.d/70-persistent-net.rules

4、重启系统，输入 init 6

### ok！成功克隆了一台虚拟机，接下来你可以进行各种操作了