# 模拟“端口 80 被占用”故障

## 搞破坏

<img width="877" height="533" alt="image" src="https://github.com/user-attachments/assets/247ed405-fb17-4140-8742-9a555fe1c4dd" />

Note: **"特权端口（<1024）必须用 root 启动"**

## 尝试重启 Nginx

<img width="1490" height="463" alt="image" src="https://github.com/user-attachments/assets/669d94dd-d9d0-4916-9bee-d438ed8f2aa2" />

<img width="1516" height="826" alt="image" src="https://github.com/user-attachments/assets/c6d16b8e-839b-4d67-8a53-483ca1bbce74" />

故障: **bind() to 0.0.0.0:80 failed (98: Address already in use)**

## 三层排查法定位, 修复与验证

<img width="1228" height="87" alt="image" src="https://github.com/user-attachments/assets/79fc3542-4840-4044-8d14-7ff5c4c059f4" />

网络层（核心）：用 ss -tulpn | grep :80 看谁占着端口。
应用层：确认这是哪个进程（Python3）

## 重启应用+验证

<img width="1452" height="463" alt="image" src="https://github.com/user-attachments/assets/b147dd94-ea8d-4fb5-a438-ed2a2fc95c51" />
