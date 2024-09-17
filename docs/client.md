# 客户端组件

## 原理

thupxe 在客户端侧的主要工具是 [desktop-builder](https://github.com/thupxe/desktop-builder)，它具有如下功能：

* 定制的 initrd script，用于在内核加载后下载系统镜像到本地、获取必要配置并引导。
* GitHub Workflow，用于在更新时自动构建系统镜像并发布。

此外，[`desktop-flasher`](https://github.com/thupxe/desktop-flasher/) 可用于制作 thupxe 专用的启动盘。

## 基本要求

在客户端上，thupxe 需要如下的条件方能工作：

* 启动时引导进入 iPXE 环境，可以通过 PXE 级联引导，或者预置 iPXE 镜像于相关存储设备。
* 存放系统镜像和持久化状态的空间，可以是 thupxe 独有的磁盘（硬盘或者 U 盘）、已经存在的 ext4 分区，也可以是一段保证不会被使用的空间片段。

这两个条件是互相独立的，即可以使用任何方式引导 iPXE，并告知 thupxe 使用的存储空间即可。

下面将从这两方面详细说明相关事项的原理和准备工作。

## 引导准备

!!! tip "PXE First"

    在可行的情况下，优先推荐使用网络引导，可以避免在本地部署 iPXE 镜像的工作。

### 网络引导

为了进行网络引导，需要进行以下的固件配置：

* 启动模式：UEFI
* 安全启动（Secure Boot）：关闭
* UEFI 网络堆栈：启用
* 启动顺序：IPv4 PXE 为第一优先级（也可能书写为 Network Boot 等字样），如机器有多个网卡，选择正确的网卡

配置成功的标志是开机显示 Booting from PXE 等字样，然后能够进入 iPXE 环境。

!!! warning "潜在问题"

    在某些设备上，PXE 引导可能会遇到以下的问题：

    1. 开机被跳过：在某些设置了固件密码的设备上（尤其是 Dell 特定机型），还可能需要使能在不输入密码的情况下进行网络引导，请在 Security 相关选项中多加查阅。
    2. 重启后无法使用：包括无法检测到网卡、无法分配到地址、直接黑屏、直接跳过等。此部分可查阅 [PXE 诊断](appendix/boot-diag.md)。

在具有还原卡和同传系统的机房中，这些配置通常可以快速地进行复制（可能叫做“传输 CMOS 配置”）。

### 本地引导

在无法进行网络引导的情况下（如难以修改配置、设备不支持等），可以在本地存储设备上预置 iPXE 镜像。如果使用 `thupxe-flasher` 制作启动盘，则它已经预置了 iPXE，仅需将相应设备配置为第一引导设备即可（legacy 或者 UEFI 均可，推荐 UEFI）。

如果没有独有存储，则可以将 `ipxe.efi` 可执行文件放置在 ESP 分区或者单个 FAT32 分区中。通常推荐的方法包括：

* 将 `ipxe.efi` 重命名为 `bootx64.efi` 放置在 ESP 分区的 `EFI/BOOT` 目录下，并将对应的磁盘调成第一引导设备。
* 将 `ipxe.efi` 放置在 ESP 分区的 `EFI` 目录下，并在固件中添加对此文件的引导项，设置为第一引导项。

某些同传系统要求每个“系统”都有系统分区，同时也会建立 ESP。此时，可以使用允许的最小系统分区体积，以加速复制。

## 存储准备

### 专用存储

如果活动场地的计算机可以接入 thupxe 专用的存储设备，就可以使用 `thupxe-flasher` 制作 thupxe 专用的启动盘。它可以自动地完成分区表建立、iPXE 引导预置、系统镜像预置等工作。具体请阅读相关工具的文档。

在 `thupxe-flasher` 烧录分区表时，会产生一个用于存放系统镜像和用户数据的 GPT 分区。
该分区记录于 GPT 分区表的名称为 `thupxeroot`，用于 thupxe 引导时识别（默认传的内核参数即为 `thupxeroot=PARTLABEL=thupxeroot`）。
该分区的 UUID 是根据硬盘的序列号生成的，规则为：

```python
part_uuid = uuid.uuid5(uuid.uuid5(uuid.NAMESPACE_DNS, "lab.cs.thu.edu.cn"), disk_serial)
```

其中，`disk_serial` 是硬盘的序列号。这样，可以批量完成烧写；烧写完成后，
再使用扫码枪读取印刷在硬盘表面的序列号，即可恢复出写入的 UUID。

使用硬盘序列号生成 UUID 的原因是，硬盘序列号既可以使用软件手段读取，
又可以在硬盘表面物理地读取，从而将物理可识读信息与写入硬盘的数据建立起联系，
使得烧录硬盘分区表结构的过程可以批量并行进行，无须按照特定顺序。
在烧录完成后，一边用扫码枪读取硬盘序列号，一边按顺序粘贴硬盘顺序标签，
同时序列号相应地被记录于计算机中的表格上。这样可以有效提高效率，且不易出错。

thupxe 系统引导后，默认会使用存储系统镜像的分区的 UUID 作为 DHCP 使用的主机 ID。

因此，在 DHCP 服务器上识别上述主机 ID，即可固定地为持有同一主机 ID 的客户端分配相同的 IP
地址和主机名，从而建立 IP 地址—主机名—主机 ID—硬盘序列号—主机编号/座位号的映射，
以满足各类活动的合规性要求。

如果使用无法物理识读序列号的设备（如大部分 U 盘），则需要考虑其他的对应方案，包括且不限于：

* 直接收集主机的 MAC 地址，并在 DHCP 服务器上进行固定分配（此步骤可选，但更有利于溯源）。
* 事先给硬盘编号，在烧录时记录分区 UUID 与编号的映射；在部署时记录硬盘与座位号/主机编号的映射。

对 U 盘的选型有以下要点：

* 价格相对低廉；
* 读写速度相对较快：不但要求为 USB3.0，而且对颗粒的质量也有一定要求，某些“投标 U 盘”是不可行的；
* 容量足够存放系统镜像：为 64GB 或以上；

建议提前购买多种型号的 U 盘，进行读写测试，并注意要在所使用的计算机上实际测试，以排除潜在问题。

目前较为推荐的型号有：

* 三星 Bar Plus 64GB

如果需要批量写入 U 盘，建议使用 USB Hub，同时连接多个写入设备，以提高效率。
不建议直接使用计算机上的 USB 口，以防多次频繁插拔降低其寿命。

### 非专用存储

如果活动场地的计算机无法接入 thupxe 专用的存储设备，则需要在已有的存储设备上进行空间预留。可行的方案包括：

* 单独的分区：如 `thupxeroot=/dev/sda2`。其中 `thupxeroot=` 后传入的参数格式与一般 Linux 系统接受的 `root=` 参数相同，也可以是诸如 `UUID=xxx`、`PARTLABEL=xxx` 等。
* 块设备（如 `/dev/sda`）的空间片段：如 `thupxeroot=/dev/sda thupxeoff=-25G:20G:0` 表示 `/dev/sda` 的最后 25G 开始的 20GB 空间。
  `thupxeoff` 语法格式为 `<start>:<size>:<end>`。按下列规则确定空间片段的偏移和长度：
  1. 将 start 和 end 转换为字节（其中 G、M、K 分别表示 GiB、MiB、KiB），并取对所选块设备大小的模；
  2. 若 start 为零，且 end 和 size 非零，则取 start 为 end - size；
  3. 若 size 为零，且 start 和 end 非零，则取 size 为 end - start；
  4. 若 size 和 end 为零，且 start 非零，则取 size 为所选块设备大小减去 start；
  5. 检查 start + size 不超出所选块设备大小，检查 start 非负，检查 size 为正；
  6. 确定空间片段的偏移为 start，长度为 size。

需要注意，在这些方案中，需要考虑：

1. 首次部署时，可能需要传入 `thupxeformat=yes` 强制格式化对应的分区。
2. 尽量选择最方便的空间预留方式。如机房使用的大部分同传系统无法分析非 NTFS 分区的内容，因此会进行逐扇区拷贝。此时可以考虑建立较小的分区欺骗同传系统仅复制分区表（和相关引导文件），并在后面预留较大的空白空间供 thupxe 实际使用。
3. 如果有多种不同类型的空间预留方式，则需要能在引导时识别这些设备，并传递不同的 `thupxeroot` 参数，具体可参考 [此博文](https://harrychen.xyz/2023/04/29/ipxe-configuration-dispatch-based-on-mac-prefix-with-nginx/)。

关于如何在不增加专用存储的情况下配置 thupxe 的更多实例，请参考 [此文档](appendix/boot-diag.md)。

## 系统镜像生成

### GitHub Actions

客户端的镜像在 [thupxe/desktop-builder](https://github.com/thupxe/desktop-builder/) 仓库的 CI 上自动构建并生成。生成后，可以从 Artifact 下载制作好的 image。

### 调试与 patch

TODO

## 系统镜像部署

### 镜像 overlay

镜像由多个 squash 文件组成，这些文件通过 overlay 叠加的顺序记录在 series 文件中，一行一个文件名。在 version 文件中记录当前镜像的版本号，客户端启动时会通过 HTTP 获取最新镜像的版本号，如果版本号不一致，会通过 RSYNC/UDP 获取新的镜像。

### rsync 自动同步

客户端在启动时，会访问服务端上的 HTTP Server，获取配置文件。配置文件中可以设置 USE_MULTICAST 变量，如果设置 USE_MULTICAST=false，那么客户端会通过 rsync 自动从服务端同步系统镜像。

### udp-cast 多播分发

客户端在启动时，会访问服务端上的 HTTP Server，获取配置文件。配置文件中可以设置 USE_MULTICAST 变量，如果设置 USE_MULTICAST=true，那么客户端会启动 udp-receiver，等待多播分发。

在服务端上运行 `udp-sender -f images.tar.zstd`，当看到足够数量的客户端出现时，按任意键开始使用 UDP 进行系统镜像分发。部分网络环境对组播的支持较差，可以添加 `--broadcast` 命令行参数。

需要注意 images.tar.zstd 的内容与通过 HTTP/RSYNC 提供的系统镜像的一致性，如果版本不一致，会导致客户端重复重启并通过 UDP 获取系统镜像。也可以先用 UDP 传大部分的系统镜像，再用 RSYNC 传输剩下的小文件。
