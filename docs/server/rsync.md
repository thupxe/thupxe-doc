# RSYNC Server

RSYNC 服务器使用的是 rsync，配置步骤：

1. 使用 APT 安装：`sudo apt install rsync`
2. 编辑 `/etc/rsyncd.conf`，假设服务器的地址是 10.1.0.1，添加下列配置：
    ```
    hosts allow = 10.1.0.0/255.255.0.0 127.0.0.0/255.0.0.0
    readonly = yes

    [images]
    path = /srv/thupxe/images
    ```
3. 启动 RSYNC Server 并设置为开机启动：`sudo systemctl enable --now rsync`
