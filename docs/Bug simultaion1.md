# 目标：用 dd 填满磁盘，然后用 SOP 排查并修复。

## 步骤 1：制造故障
往 /root 目录下写 6.5GB 大文件（因为根分区剩余 6.3GB，6.5G 刚好填满）
```bash 
dd if=/dev/zero of=/root/diskfill bs=1M count=6500
```
<img width="766" height="306" alt="image" src="https://github.com/user-attachments/assets/cab96dae-d6d8-41b1-9bd2-ba1c5d40332e" />

## 步骤 2：观察现象
确认根目录填满
```bash
df -h
```

<img width="718" height="201" alt="image" src="https://github.com/user-attachments/assets/436f79c7-c4ef-44c4-90c2-e6080b339203" />

### 磁盘满了，Nginx 为什么没挂？
1. **Nginx Master 进程**早就启动好了，它的二进制文件、配置文件和 index.html 已经全部加载到**内存**或由 **内核缓存（Page Cache**提供。
2. **Nginx 的访问日志（access.log）和错误日志（error.log）是异步写入的**，如果磁盘突然写满，Nginx 默认会把写入失败的日志丢弃（跳过），而不会直接崩溃进程
3. 执行 nginx -s reload 或 systemctl restart nginx，Nginx 需要向 /var/run/nginx.pid **写入新 PID**，或者向日志文件写 reopen 日志，这时会**直接报错退出（无法启动）**。

#### 先删除（或移走）现有的日志文件
先停掉 Nginx（防止旧进程持有已删除文件的句柄）

```bash
systemctl stop nginx
```

强制删除 Nginx 的访问日志和错误日志（默认在 /var/log/nginx/ 下）

```bash
rm -f /var/log/nginx/access.log /var/log/nginx/error.log
```

现在填满根分区（务必用 fallocate 占满，避免保留块干扰）

```bash
df -h /
fallocate -l 500K /root/fillfile   # 大小根据实际剩余空间调整，确保 Use% 达到 100%
```

确认磁盘已满（显示 0% 可用）
```bash
df -h /
```
<img width="825" height="190" alt="image" src="https://github.com/user-attachments/assets/9e377369-7da2-409b-a65f-26ef8e09ff23" />

重启
```bash
systemctl restart nginx
```

查看状态
```bash
systemctl status nginx
```   
<img width="1885" height="455" alt="image" src="https://github.com/user-attachments/assets/ee9eb02d-8c84-4f82-90a0-7584a7fd976c" />

## 步骤 3：用三层排查法定位

## 步骤 4：修复
