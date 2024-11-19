# DNS Server

DNS 服务器使用的是 coredns，配置步骤：

1. 参考 [coredns/debian](https://github.com/coredns/debian) 在 Debian 上构建并安装 coredns
2. 编辑 `/etc/coredns/Corefile`，假设服务器的地址是 10.1.0.1，想要把域名 example.xyz 指向服务器，添加下列配置：
    ```
    . {
        bind 10.1.0.1
        root /etc/coredns
        file db.hijack example.xyz
        log     # coredns.io/plugins/log
        errors  # coredns.io/plugins/errors
    }
    ```
3. 编辑 `/etc/coredns/db.hijack`，为 example.xyz 设置 DNS 记录：
    ```
    ;
    $TTL    600
    @       600 IN  SOA ns root 1 604800 86400 2419200 86400
    @       600 IN  NS      ns

    ns      IN      A       10.1.0.1
    @       IN      A       10.1.0.1
    ```
3. 启动 DNS Server 并设置为开机启动：`sudo systemctl enable --now coredns`
