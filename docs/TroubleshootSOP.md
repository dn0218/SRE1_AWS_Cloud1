#三层排障Checklist（系统层→网络层→应用层）

| 层级 | 排查工具/命令 | 常见症状 |
| :---: | :---: | :---: |
| 系统层 | top, free -h, df -h, systemctl status nginx | CPU满、内存不足、磁盘满、服务挂掉 |
| 网络层 | ss -tulpn, curl -v, telnet 80 | 端口没监听、安全组没放行、路由不通 |
| 应用层 | tail -f /var/log/nginx/access.log, error.log | 404、500、权限拒绝 |
