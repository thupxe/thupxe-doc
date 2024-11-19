# 服务端组件

## 软件依赖

服务端需要运行如下软件：

1. DHCP Server isc-dhcp-server：用于给客户端分配 IP 地址和下发 PXE/iPXE 配置路径
2. TFTP Server tftpd-hpa：用于固件下载 iPXE 程序
3. HTTP Server nginx：用于提供 iPXE 配置、Linux 内核、initramfs 和 THUPXE 的配置文件
4. RSYNC Server rsync：用于提供 rootfs
5. UDP Multicast udpcast：用于通过组播方式下发 rootfs
6. NTP Server chronyd：负责客户端的时钟同步
7. DNS Server coredns：负责客户端的域名解析

