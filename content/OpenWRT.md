#### 光猫关闭DHCP后OpenWrt Passwall无法使用的解决方案
##### 问题原因分析
当光猫关闭DHCP服务后，OpenWrt上的Passwall插件无法正常工作，主要原因如下：

‌1. DHCP服务依赖‌：Passwall插件需要正常的网络连接和DNS解析功能，而光猫关闭DHCP后，OpenWrt无法自动获取IP地址，导致网络连接中断

2‌. 网络配置缺失‌：OpenWrt在无DHCP环境下需要手动配置IP地址、子网掩码、网关和DNS，否则无法与光猫建立正常通信

‌Passwall工作原理‌：Passwall是基于ShadowsocksR-Plus修改的OpenWrt插件，依赖HaProxy负载均衡和正确的网络配置才能实现分流和代理功能

#### 解决方案
##### 第一步：配置OpenWrt网络
‌###### 1. WAN口设置‌：

- 进入OpenWrt管理界面 → 网络 → 接口 → WAN
- 将协议改为"静态地址"
- 手动配置IP地址、子网掩码、网关
  
‌‌###### 2. LAN口设置‌：

- 进入网络 → 接口 → LAN
- IP设置为指定网段，如10.0.0.1
- 将协议改为"静态地址"
- 将网关和DNS服务器设置为光猫的IP地址
- 在DHCP服务器设置中，确保"忽略此接口"选项被勾选
