# TFTP Server

TFTP 服务器使用的是 tftpd-hpa，配置步骤：

1. 使用 APT 安装：`sudo apt install tftpd-hpa`
2. 修改 `/etc/default/tftpd-hpa` 中的 TFTP_ADDRESS，把要监听 TFTP Server 的 IP 地址写进去
3. 启动 TFTP Server 并设置为开机启动：`sudo systemctl enable --now tftpd-hpa`
