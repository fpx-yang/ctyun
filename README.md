# ctyun
## 弹性云主机
1、进入官网，控制中心，虚拟私有云
2、创建私有云（默认就好）
3、创建安全组（控制访问下）  
模板选择自定义：配置规则，添加规则。一个是ssh的，一个是业务端口
![图片1](https://github.com/user-attachments/assets/13a002c9-5c01-49fc-9eb8-12475c4d0667)  
设置出方向规则
![图片2](https://github.com/user-attachments/assets/e127b863-8f8d-40ff-994a-2fa1f76d0ed6)  
4、开弹性ip（比赛时候可能是开好的）  按量付费，按流量计费，带宽可以拉大  
5、开弹性云主机  
控制中心，弹性云主机->按量计费--config，2核4gb，Ubuntu  
网络配置：网卡、安全组、弹性ip使用已有->密码，ybh@200001130  
购买、开通（费用中心，我的订单、控制台），在弹性云主机页面等待开通  
6、ssh连接   通过ip地址（公）在cmd中ssh root@ip  
7、将业务文件传过去  
cd /opt->如果有网址  wget https://jiangsu-10.zos.ctyun.cn/bucket-179393974/app-server1-demo  
或者本机再开一个scp "C:\Users\User\Desktop\学习\云计算\app-server1-demo" root@ip:/opt  
8、执行  
chmod 755 app-server1-demo->./app-server1-demo->./app-server1-demo democonfig  
vi config.json(将例子复制进来)  按i进入插入模式，输入,按esc进入命令模式，按：键进入底行模式  
:wq     vi config.json  
![image](https://github.com/user-attachments/assets/7d2bc525-e252-4df6-a4f8-7aee723da7d2)  
./app-server1-demo start -config config.json  
![image](https://github.com/user-attachments/assets/a308be1f-9258-4692-ba3d-d07103bd3da5)  
可以接收请求了  
9、测试  
复制公网ip，浏览器ip:7000  
![image](https://github.com/user-attachments/assets/ec4ce1aa-5477-4443-9638-7b26e5886c1b)  
ip:7000/calc?input=222    curl http://ip:7000/calc?input=222
![image](https://github.com/user-attachments/assets/d0121bed-928e-41a6-bd74-3c01f7fa5a0a)   ![image](https://github.com/user-attachments/assets/c5e34220-e606-453b-9738-7316398447e9)  
10、制作自启动  
cd /      先尝试启动 /opt/app-server1-demo start -config /opt/config.json  
vi /etc/systemd/system/app-server.service  
![image](https://github.com/user-attachments/assets/4dead811-03ce-40be-9037-c1a467e64f07)  
esc  :wq->systemctl start app-server->systemctl status app-server->systemctl enable app-server  
systemctl daemon-reload 之后可以reboot,也可以不用  
11、设置弹性负载均衡器  
虚拟私有云的界面，弹性负载均衡，经典型、外网，购买->点入弹性负载均衡器，添加监听器，tcp 7000  
![image](https://github.com/user-attachments/assets/ee248672-2d6f-4071-b050-36652bb812aa)  
继续添加后端主机  
![image](https://github.com/user-attachments/assets/55e375da-3ceb-40f4-934b-a96bc040c33a)  
7000端口，权重100  访问负载均衡器 ip地址的7000端口  
12、增加  
![image](https://github.com/user-attachments/assets/76109fed-7010-434b-87cb-fe7d0d95b847)  
不需要改什么，之后在镜像服务、私有镜像看到正常运行  
![image](https://github.com/user-attachments/assets/10602a46-5ef2-4154-aa77-125a5604462b)  
13、弹性伸缩，伸缩配置  
可用区都选上，通用型，s7，large.2  2c4g的，私有镜像，弹性ip不使用，密码ybh@200001130  
均衡分布，负载均衡启用（后端端口7000，权重100），创建  在弹性伸缩组处，修改，期望实例数2
之后等待负载均衡上线，负载均衡--后端主机组  
将http://ip:7000提交




