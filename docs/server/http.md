# HTTP Server

HTTP 服务器使用的是 nginx，配置步骤：

1. 使用 APT 安装：`sudo apt install nginx`
2. 编辑 `/etc/nginx/sites-available/thupxe.conf`，假设服务器的地址是 10.1.0.1，添加下列配置：
    ```
    server {
        listen 10.1.0.1:80;
        server_name 10.1.0.1;
        root /srv/thupxe;
        access_log /var/log/nginx/thupxe.access.log;
        error_log /var/log/nginx/thupxe.error.log warn;
    }
    ```
3. 添加软链接：`/etc/nginx/sites-enabled/thupxe.conf -> /etc/nginx/sites-available/thupxe.conf`
4. 启动 HTTP Server 并设置为开机启动：`sudo systemctl enable --now nginx`
