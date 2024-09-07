# 服务端组件

## 软件依赖

服务端需要运行如下软件：

1. DHCP Server isc-dhcp-server：用于给客户端分配 IP 地址和下发 PXE/iPXE 配置路径
2. TFTP Server tftpd-hpa：用于固件下载 iPXE 程序
3. HTTP Server nginx：用于提供 iPXE 配置、Linux 内核、initramfs 和 THUPXE 的配置文件
4. RSYNC Server rsync：用于提供 rootfs
5. UDP Multicast udpcast：用于通过组播方式下发 rootfs
6. NTP Server chronyd：负责客户端的时钟同步
7. DNS Server dnsmasq：负责客户端的域名解析

## 服务配置

DHCP Server 配置时需要注意：

1. 配置默认路由、DNS 服务器和 NTP 服务器为自己
2. 通过 DHCP Option 指定 next server 为自己，通过 user-class/vendor-class-identifier option 判断客户端是固件的 PXE 还是 iPXE，分别下发 iPXE 的 TFTP 下载路径或 iPXE 的配置文件路径
