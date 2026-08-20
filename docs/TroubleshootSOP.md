# 三层排障Checklist（系统层→网络层→应用层）

| 层级 | 排查工具/命令 | 常见症状 |
| :---: | :---: | :---: |
| 系统层 | top, free -h, df -h, systemctl status nginx | CPU满、内存不足、磁盘满、服务挂掉 |
| 网络层 | ss -tulpn, curl -v, telnet 80 | 端口没监听、安全组没放行、路由不通 |
| 应用层 | tail -f /var/log/nginx/access.log, error.log | 404、500、权限拒绝 |

## 特殊场景1: rm -f bigfile 删除了文件，但 df -h 还是显示 100%

```bash
lsof | grep deleted #过滤出所有已删除但仍在被进程占用的文件
lsof +L1 #显示link count为 0 的文件
```

## 故障场景：端口被占用（Address already in use）
- **现象**：`systemctl restart nginx` 失败，报错 `bind() failed (98: Address already in use)`。
- **排查指令**：`ss -tulpn | grep :<端口号>`（快速定位PID）。
- **根因**：其他进程（如临时Python/Node服务）未正常退出，或Nginx残留进程。
- **修复**：`kill -9 <PID>` 强制清理，随后重启目标服务。
- **面试补充**：如果端口被 `systemd` 托管或处于 `TIME_WAIT` 状态，需用 `ss -tunap` 查看状态，`TIME_WAIT` 通常等待2MSL后自动释放，不用杀进程。
