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

## 故障场景：配置语法错误

| 命令 | 行为 | 配置错误时的影响 | 生产环境建议 |
| --- | --- | --- | --- |
| nginx -t | 只检查语法，不碰进程 | 报错但**服务无影响** | ✅ 执行100次都安全 |
| nginx -s reload | 平滑重载，旧进程处理完请求才退出 | 报错但**旧进程继续服务（用户无感）** | ✅** 首选**（零停机） |
| systemctl restart | 先杀旧进程，再起新进程 | 旧进程被杀，新进程起不来 = **全站宕机** | ❌ 绝对禁止在未测试时直接使用 |

- 严格遵循 **'测试-重载'（Test-Reload） 流程**
- 先用 **nginx -t 验证语法**，确认无误后执行 **nginx -s reload 实现平滑生效** ，绝不直接使用 systemctl restart 以避免服务中断
