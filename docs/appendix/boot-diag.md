# 引导诊断

## 引导相关

此处提供部分已有系统在不增加专用存储情况下的配置方法，供参考。

### 计算机一

Dell，有还原卡。

1. 进入安装有还原卡驱动的 Windows 系统，右键单击托盘中的“磁盘监视器”——登录，进入还原卡管理工具的分区工具。
2. 如有必要，删除一些已有的系统（L2 可选 win7 test）。创建新的分区和系统，名称为 iPXE，分区表格式为 GPT，文件系统必须为 FAT32，大小尽量小（以节省传送时间），但需要大于 300MB（否则无法识别为系统分区）。由于还原卡只支持从后往前删除分区，所以建议使用最后一个分区。
3. 点击“更新分区”后，还原系统会自动创建 ESP 分区、MSR 分区和对应的系统分区（部署中不使用后两者），并重启。
4. 重启后，按 F12 选择 LiveUSB 引导。依旧会进入还原卡界面，按 Home 进入管理。将鼠标移动到 iPXE，按 Ctrl-O 进入临时开放模式，再次选择 U 盘引导，进入 Ubuntu Live。
5. 在磁盘管理工具中确认 ESP 分区的编号和设备名称，格式化为 FAT32。挂载 ESP 分区，将 `ipxe.efi` 放置到 `EFI/BOOT/bootx64.efi`。卸载 ESP 分区。
6. 进入 iPXE，应当可以正常进入 PXE 流程，从服务器部署系统。
7. 重启到 Windows。进入还原卡“系统设置”，将默认系统修改为 iPXE。此外，可以将其他系统在启动菜单中隐藏（名称前加 “~”）。进入分区工具，将所有共享分区对 iPXE 系统隐藏（取消分区），点击“更新分区”重启。
8. 确认无误后，使用同传功能将 CMOS 参数、分区表相关参数、iPXE 分区数据（只要选 ESP）、配置（默认系统）传送到各个机器。重启，验证各机器正确工作。

### 计算机二

HP，有还原卡。

1. 进入安装有还原卡驱动的 Windows 系统 / 开机按 F10，右键单击托盘中的“噢易”——登录，进入还原卡管理工具。
2. 如有必要，删除一个已有的系统（L2 可选 Windows 7 BY）。创建新的系统分区，名称为 iPXE，分区表格式为 GPT，大小尽量小。
3. 点击“执行”后，还原卡会自动创建 ESP 分区和对应的系统分区（部署中不使用后者）。
4. 重启到还原卡界面，按 F10 进入管理，选择安装系统，选择 iPXE 分区，并选择准备好的 LiveUSB 引导，进入 Ubuntu Live。
5. 在磁盘管理工具中确认 ESP 分区的编号和设备名称，格式化为 FAT32。挂载 ESP 分区，将 `ipxe.efi` 放置到 `EFI/BOOT/bootx64.efi`。卸载 ESP 分区。
6. 重启到还原卡界面，此时可以调整默认系统为 iPXE，隐藏其他系统。引导 iPXE 系统，应当可以正常进入 PXE 流程，从服务器部署系统。
7. 确认无误后，使用同传系统中的“差异拷贝”功能将分区表相关参数、iPXE 分区数据（需要选择“逐个扇区发送”）、配置（默认系统选择等）传送到各个机器。重启，验证各机器正确工作。注意：同传系统依赖于 WoL 和 PXE 启动，在进行同传时，需要关闭服务器的 DHCP，以免造成混乱。

### 计算机三

Dell，有还原卡。

1. 在 Windows 系统中打开还原卡的分区管理
2. 选择分区 2（数据分区），删除分区
3. 选择添加系统，名称 iPXE，系统类型 Linux，系统盘容量 5GB（只使用 EFI 分区，其他无用分区空间太大，会浪费传输时间）
4. 选择 Windows 10 系统，选择系统属性，添加 `~` 隐藏
5. 点击分区更新，确认，等待系统重启
6. 重新进入 Windows 10，进入还原卡系统设置，更改默认系统为 iPXE
7. 插入 U 盘，在引导界面移动到 iPXE 按 Ctrl-O 进入 U 盘引导
   1. 格式化 EFI 分区，将提供的 `ipxe.efi` 放置到 EFI 分区的 `/EFI/BOOT/bootx64.efi`
   2. 分区表结构会在下次引导之后被覆盖，因此只能使用现有的分区结构（或者后部未使用空间），无法使用 partlabel 找到此时新建的分区
8. 重启进入 iPXE 系统，检查是否正常引导至 Linux

## PXE 相关

可以在 `pxe.conf` 中插入一些修补命令，解决部分网络引导问题。

强制启用网卡的 WOL 功能，在部分网卡上可以解决下次重启时 PXE 无法初始化硬件的问题（需要在 initramfs 中预置 ethtool）。

```bash
for i in /sys/class/net/en*; do
 if [ "$(cat "$i/type")" = "1" ]; then
   intf="$(basename "$i")"
   ethtool -s "$intf" wol g || true
 fi
done
```

自动向 UEFI 固件写入下次引导项为本次使用的引导项，以解决有时引导配置无法持久化的问题：

``` bash
(
  mkdir -p /run/efivarfs
  if mount -t efivarfs efivarfs /run/efivarfs; then
    if [ -e "/run/efivarfs/BootCurrent-8be4df61-93ca-11d2-aa0d-00e098032b8c" ]; then
      bootcurrent=$(dd if=/run/efivarfs/BootCurrent-8be4df61-93ca-11d2-aa0d-00e098032b8c bs=1 count=2 skip=4 | xxd -p)
      echo "BootCurrent: $bootcurrent"
      (echo -ne '\x07\x00\x00\x00'; dd if=/run/efivarfs/BootCurrent-8be4df61-93ca-11d2-aa0d-00e098032b8c bs=1 count=2 skip=4) > /tmp/.bootnext
      cat /tmp/.bootnext >/run/efivarfs/BootNext-8be4df61-93ca-11d2-aa0d-00e098032b8c
      if [ "$?" -eq 0 ]; then
        echo "Write bootnext"
      else
        echo "Cannot Write bootnext"
      fi
    fi
    umount /run/efivarfs
  fi
  rmdir /run/efivarfs
)
```
