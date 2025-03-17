# SSH 简明教程

## 什么是 SSH？

SSH（Secure Shell）是一种加密的网络协议，用于安全地访问远程服务器。它通过加密通信来防止数据被窃听或篡改

## 基本用法

1. 连接到远程服务器

   ```bash
   ssh username@remote_host
   ```

   - `username`:远程服务器的用户名
   - `remote_host`:远程服务器的 IP 地址或域名

2. 指定端口

   ```bash
   ssh -p port_number username@remote_host
   ```

3. 使用密钥登录

   - 生成密钥对

     ```bash
     ssh-keygen # 后续会对话式地询问密钥所使用的格式和保存位置， 文件名等
     ```

   - 将公钥（一般以 .pub 结尾）内容添加到远程服务器中用户 home 文件夹下的`.ssh/authorized_keys`文件末尾，如果不存在则新建该文件，文件名需保持一致且保证当前用户对其有可读权限
   - 如果创建的密钥不是默认命名（如 rsa 格式的默认命名为 id_rsa）, 或未放置在默认位置（当前用户 home 目录的.ssh 文件夹内）， 可以通过 config 文件配置, 如下是一组示例配置：

     ```config
     Host github                              #自定义host名
        HostName github.com                   #远程服务器的域名或IP
        User git                              #用户名，同ssh username@remote_host方式中的用户名
        IdentityFile ~/.ssh/id_ed25519        #私钥文件所在位置
        Port 1922                             #指定端口，不指定则使用默认的22端口号进行连接
     ```

   - 完成上述配置后可通过`ssh name`进行连接， 如上述配置， 可通过`ssh github`进行连接
   - 使用其他 ssh 进行连接

     config 文件中进行配置时， 可指定其他 ssh 配置作为中转， 常见的场景是通过堡垒机进行连接

     ```config
     Host myHost
        HostName domainName
        User username                #用户名
        ProxyJump anotherHost        #跳板连接
     ```

## SSH Tunnel（隧道）

SSH 隧道可以用于安全地转发网络流量，常用于绕过防火墙或加密不安全的协议

1. 本地端口转发

   ```bash
   ssh -L local_port:remote_host:remote_port username@ssh_server
   ```

   - `local_port`: 本地端口
   - `remote_host`: 远程服务器的 IP 或域名
   - `remote_port`: 远程服务器的端口
   - `ssh_server`: SSH 服务器的 IP 或域名

   示例：将本地的 8080 端口转发到远程服务器的 80 端口，访问 localhost:8080 将会连接到远程服务器的 80 端口：

   ```bash
   ssh -L 8080:localhost:80 username@ssh_server  #username@ssh_server可被替换成有效的配置名， 即在～/.ssh/config中定义的一组配置
   ```

2. 远程端口转发

   ```bash
   ssh -R remote_port:localhost:local_port username@ssh_server
   ```

   - `remote_port`: 远程服务器的端口
   - `local_port`: 本地端口

   示例：将远程服务器的 8080 端口转发到本地的 80 端口，远程用户访问 ssh_server:8080 将会连接到本地的 80 端口：

   ```bash
   ssh -R 8080:localhost:80 username@ssh_server
   ```

3. 动态端口转发（SOCKS 代理）

   ```bash
   ssh -D local_port username@ssh_server
   ```

   - `local_port`: 本地端口

   示例：创建一个本地 1080 端口的 SOCKS 代理, 配置浏览器或其他应用使用 localhost:1080 作为 SOCKS 代理，所有流量将通过 SSH 隧道加密传输：

   ```bash
   ssh -D 1080 username@ssh_server
   ```

## 其他常用选项

- `-v`: 显示详细的调试信息（可重复使用 -vv 或 -vvv 增加详细程度）
- `-f`: 后台运行 SSH
- `-N`: 不执行远程命令，仅用于端口转发
- `-C`: 压缩数据传输

## 断开连接

   在 SSH 会话中，输入 exit 或按 Ctrl+D 即可断开连接。
