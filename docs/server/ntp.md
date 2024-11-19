# NTP Server

NTP 服务器使用的是 chronyd，配置步骤：

1. 使用 APT 安装：`sudo apt install chrony`
2. 编辑 `/etc/chrony/chrony.conf`，假设服务器的地址是 10.0.0.1，添加下列配置：
    ```
    allow 10.0.0.0/16
    ```
3. 启动 NTP Server 并设置为开机启动：`sudo systemctl enable --now chronyd`
