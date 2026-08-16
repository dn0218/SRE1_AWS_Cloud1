前提条件：AWS账号、密钥对权限（chmod 600）。

**架构拓扑图**

<img width="1450" height="602" alt="image" src="https://github.com/user-attachments/assets/5d0e6c5b-5109-47b0-a302-168286b72e07" />

<img width="1465" height="750" alt="image" src="https://github.com/user-attachments/assets/f3510099-7f82-4613-8b54-aa64115d34bc" />


**关键命令流水账（按顺序记录）：**


**VPC/子网/IGW/路由表创建（CLI或控制台步骤）。**

<img width="1452" height="771" alt="image" src="https://github.com/user-attachments/assets/29bab3cb-95b3-4503-8999-2ecfdfa3d0b9" />

**EC2启动参数（AMI、实例类型、密钥、安全组）。**
<img width="1347" height="897" alt="image" src="https://github.com/user-attachments/assets/0a1c3976-9ae4-4720-bf0d-5d9edb3de358" />

<img width="1377" height="562" alt="image" src="https://github.com/user-attachments/assets/ad30dd6d-035f-4b70-917e-d7901ad07e90" />

<img width="1770" height="450" alt="image" src="https://github.com/user-attachments/assets/be89c045-61c2-4b4e-a949-63b53251a5d5" />


**安全组与NACL对比表**

|组件 | 状态特性 | 核心区别|
|Security Group（安全组）	| 有状态（Stateful）| 	只要我允许了入站请求（如22端口），那么从这个EC2返回的响应包，无论出站规则怎么设，都会自动放行。（你不需要额外配出站规则）|
|Network ACL（网络ACL）|	无状态（Stateless）|	入站和出站规则必须同时配置。如果入站允许了22，出站必须显式允许高位端口（1024-65535）的回复包，否则连接不通。|



**验证方法：curl 命令和浏览器访问截图。**
