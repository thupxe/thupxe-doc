# DHCP Server

DHCP 服务器使用的是 isc-dhcp-server，配置步骤：

1. 使用 APT 安装：`sudo apt install isc-dhcp-server`
2. 修改 `/etc/default/isc-dhcp-server` 中的 INTERFACESv4，把要开启监听 DHCP Server 的接口写进去
3. 修改 `/etc/dhcp/dhcpd.conf`，以 10.1.0.0/16 为例，假设服务器的地址是 10.1.0.1，做如下配置：
    ```
    authoritative;
    option client-architecture code 93 = unsigned integer 16;

    class "pxe" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
    }

    subnet 10.1.0.0 netmask 255.255.0.0 {
        interface ens4;
        option routers 10.1.0.1;
        option domain-name-servers 10.1.0.1;
        option ntp-servers 10.1.0.1;

        # ip address pool for pxe clients
        pool {
            allow members of "pxe";
            range 10.1.1.0 10.1.10.254;
            next-server 10.1.0.1;
            # dispatch pxe filename
            if exists user-class and option user-class = "iPXE" {
                # this is ipxe, use our ipxe config
                option vendor-class-identifier "PXEClient";
                filename "http://10.1.0.1/thupxe.ipxe";
            } elsif option client-architecture = encode-int(7, 16) {
                # this is uefi, boot to ipxe
                filename "bin-x86_64-efi/ipxe.efi";
            } elsif option client-architecture = encode-int(0, 16) {
                # this is legacy bios, boot to ipxe
                filename "bin-i386-pcbios/undionly.kpxe";
            }
        }

        # ip address pool for non-pxe clients
        pool {
            deny members of "pxe";
            range 10.1.11.0 10.1.20.254;
        }

        # assign static ip address for known clients
        group {
            use-host-decl-names on;

            host named-client-1 {
                # match against dhcp client identifier
                option dhcp-client-identifier 00:11:22;
                fixed-address 10.1.21.1;
            }
        }
    }
    ```
4. 启动 DHCP Server 并设置为开机启动：`sudo systemctl enable --now isc-dhcp-server`

配置时需要注意：

1. 配置默认路由、DNS 服务器和 NTP 服务器为自己
2. 通过 DHCP Option 指定 next server 为自己，通过 user-class/vendor-class-identifier option 判断客户端是固件的 PXE 还是 iPXE，分别下发 iPXE 的 TFTP 下载路径或 iPXE 的配置文件路径
