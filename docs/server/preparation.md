# 准备工作

配置各项服务前，需要在 `/src/tftp` 下准备好下列文件：

- `/srv/tftp/bin-x86_64-efi/ipxe.efi`：用于 uefi 的 ipxe
- `/srv/tftp/bin-i386-pcbios/undionly.kpxe`：用于 bios 的 ipxe

这两个文件可以从 ipxe 源码编译：

```shell
git clone https://github.com/ipxe/ipxe.git
cd ipxe/src
make bin-i386-pcbios/undionly.kpxe
make bin-x86_64-efi/ipxe.efi
```

需要在 `/src/thupxe` 下准备好下列文件：

- `/srv/thupxe/thupxe.ipxe`: ipxe 配置文件，用于引导内核启动，假设服务器的地址是 10.0.0.1，根据需要修改 `thupxeclear` 参数：
    ```ipxe
    #!ipxe
    kernel vmlinuz initrd=initrd.img boot=thupxe ip=dhcp thupxeapi=http://10.0.0.1 BOOTIF=01-${netX/mac} consoleblank=0 console=tty0 thupxeroot=PARTLABEL=thupxeroot thupxeclear=yes
    initrd initrd.img
    boot
    ```
- 从 [thupxe/desktop-builder](https://github.com/thupxe/desktop-builder/) 的 CI Artifact 获取文件到下列位置：
    - `/srv/thupxe/vmlinuz`: Kernel
    - `/srv/thupxe/initrd.img`: Initramfs
    - `/srv/thupxe/images/`：System Image
    - 根据需要，可以在 CI 生成的文件的基础上做修改
- `/srv/thupxe/pxe.conf`：编写 THUPXE 配置文件，内容如下：
    ```
    # use udp multicast or not
    USE_MULTICAST=true
    # set rsync server address
    RSYNC_ADDR=rsync://10.0.0.1/images/
    # disable usb
    mkdir -p /run/systemd/system/
    ln -s /dev/null /run/systemd/system/udisks2.service
    ```