# 系统架构

THUPXE 包括如下组件：

1. iPXE: 在客户端上由 BIOS/UEFI 通过 PXE 或 USB 启动，负责下载并启动 Linux 内核和 initramfs
2. 定制的 initramfs：和服务端进行通信，将 rootfs 同步到本地，再切换到本地的 rootfs 并进入图形界面
3. DHCP Server：在服务端上给客户端分配 IP 地址，下发 PXE 配置
4. TFTP Server：在服务端上提供 iPXE 程序，供固件下载 iPXE
5. HTTP Server：在服务端上提供 iPXE 的配置，供 iPXE 找到 Linux 内核 和 initramfs；同时也提供 THUPXE 的配置
5. RSYNC Server/UDP Multicast：在服务端上提供 rootfs，使得客户端可以下载 rootfs
6. NTP Server：负责客户端的时钟同步
7. DNS Server：负责客户端的域名解析

## 启动流程

客户端从开机到进入图形界面，经过了如下的流程：

1. 开机后，由于 PXE 为第一启动优先级，客户端进入 PXE Boot 模式
2. 客户端上的固件通过 DHCP 从服务器获取 IP 地址，并从 DHCP Option 中获得 iPXE 的 TFTP 地址
3. 客户端上的固件通过 TFTP 下载 iPXE 并启动 iPXE
4. iPXE 再次通过 DHCP 从服务器获取 IP 地址，并从 DHCP Option 中获得 iPXE 配置的 HTTP 地址
5. iPXE 下载 iPXE 配置并执行，iPXE 配置中指定了 Linux 内核和 initramfs 的路径，iPXE 通过 HTTP 下载内核和 initramfs 并启动
6. Linux 内核启动，执行 initramfs 内的脚本（见 desktop-builder 的 thupxe.script）
7. 脚本寻找本地用于存储 rootfs 的分区，通过 HTTP 下载 THUPXE 的配置文件
8. 如有需要，格式化分区；检查本地分区是否有已经下载好的 rootfs image，如果没有，或者版本和服务端上的 rootfs 不同，则通过 rsync 下载 rootfs image
9. rootfs image 下载好后，挂载并成为新的根文件系统

## 网络配置

客户端与服务端需要在同一个二层网络下，该网络下仅有服务端一个 DHCP 服务器。如果需要用组播方式下发 rootfs，需要尽量避免丢包。
