---
title: 在Ubuntu上安装 轻量级的Git仓库管理工具Gitea
date: 2024-08-13 15:31:32
tags: gitea, 
---

1. **安装git**：Gitea 需要 Git 才能运行，所以首先确保你的系统上安装了 Git。
    ``` bash
    sudo apt install git
    ```
    
2. **创建git用户**：出于安全考虑，最好为 Gitea 创建一个专用的系统用户。
    ```bash
    sudo adduser --system --group --disabled-password --shell /bin/bash --home /home/git git
    ```
3. **下载gitea**：访问 Gitea 的官方下载页面来获取最新版本的 Gitea 二进制文件。选择适合你系统架构的版本下载。
    文件地址`https://dl.gitea.com/gitea/`
    下载合适的版本
    ``` bash
    wget -O gitea https://dl.gitea.com/gitea/1.22.1/gitea-1.22.1-linux-amd64
    ```
4. **给gitea文件赋予权限**：将下载的文件移动到全局位置，并给予执行权限。
    ``` bash
    sudo mv gitea /usr/local/bin/gitea 
    sudo chmod +x /usr/local/bin/gitea
    ```
5. **创建必要的文件夹**: 创建一个文件夹来存放 Gitea 的数据、配置和日志。
    ``` bash
    sudo mkdir -p /var/lib/gitea/{custom,data,indexers,public,log} 
    sudo chown git:git /var/lib/gitea/{data,indexers,log} 
    sudo chmod 750 /var/lib/gitea/{data,indexers,log} 
    sudo mkdir /etc/gitea 
    sudo chown root:git /etc/gitea 
    sudo chmod 770 /etc/gitea
    ```
6. **创建服务**：创建一个 systemd 服务文件来管理 Gitea 服务。
    官方提供的配置文件`https://github.com/go-gitea/gitea/blob/main/contrib/systemd/gitea.service`，按照自己的配置修改
    ``` bash
    wget -O gitea.service https://raw.githubusercontent.com/go-gitea/gitea/main/contrib/systemd/gitea.service
    sudo cp gitea.service /etc/systemd/system/gitea.service
    ```
7. **启动gitea服务**：启动 Gitea 服务并设置为开机启动。
    ``` bash
    sudo systemctl enable gitea 
    sudo systemctl start gitea
    # 查看运行情况
    ps -aux | grep gitea
    ```
8. **配置nginx**
    ``` conf
    server {
        listen       80;
        server_name  your_server_name;
    
        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass    http://127.0.0.1:3000;
        }
        location ~ .*\.(js|css|png)$ {
            proxy_pass  http://127.0.0.1:3000;
        }
    }
    ```
9. **访问gitea**
    浏览器访问`http://your-server-ip:3000`或者`your_server_name`,来访问 Gitea 的安装向导。

