# 练习

本文讲述如何在一个 amd64 Debian 机器上用 QEMU 运行两个虚拟机，一个作为 THUPXE 服务端，一个作为 THUPXE 客户端，它们之间通过网络连接来进行环境配置的练习。

1. 创建服务端 VM 并安装 Debian，这个虚拟机会有两张网卡，第一张网卡（`ens3`）用于连接外网，第二张网卡（`ens4`）用于连接客户端：
    ```shell
    wget https://mirrors.tuna.tsinghua.edu.cn/debian-cd/current/amd64/iso-cd/debian-12.8.0-amd64-netinst.iso
    qemu-img create -f qcow2 server.qcow2 50G
    cp /usr/share/OVMF/OVMF_VARS.fd efivars.img
    # boot with installation iso
    qemu-system-x86_64 -accel kvm -cpu host -m 16G -smp 8 \
        -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp:127.0.0.1:2222-:22 \
        -device e1000,netdev=net1 -netdev socket,id=net1,listen=127.0.0.1:1234 \
        -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE.fd,readonly=on \
        -drive if=pflash,format=raw,unit=1,file=efivars.img \
        -drive file=server.qcow2,format=qcow2 \
        -cdrom debian-12.8.0-amd64-netinst.iso
    # boot without installation iso
    qemu-system-x86_64 -accel kvm -cpu host -m 16G -smp 8 \
        -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp:127.0.0.1:2222-:22 \
        -device e1000,netdev=net1 -netdev socket,id=net1,listen=127.0.0.1:1234 \
        -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE.fd,readonly=on \
        -drive if=pflash,format=raw,unit=1,file=efivars.img \
        -drive file=server.qcow2,format=qcow2
    ```
2. 按照本文档的其他内容配置服务端，需要注意的是，由于客户端 VM 的硬盘还没有分区表，在 `/srv/thupxe/thupxe.ipxe` 中把 THUPXE 设置安装到全盘：`thupxeroot=/dev/sda`
3. 创建客户端 VM，配置网卡让它连接服务端 VM 的第二张网卡：
    ```shell
    qemu-img create -f qcow2 client.qcow2 50G
    qemu-system-x86_64 -accel kvm -cpu host -m 16G -smp 8 \
        -device e1000,netdev=net0 -netdev socket,id=net0,connect=127.0.0.1:1234 \
        -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE.fd,readonly=on \
        -drive file=client.qcow2,format=qcow2
    ```