![image](https://github.com/user-attachments/assets/1cd91301-9d1a-4dc5-91b4-4870bf73d36b)# ctyun  
## 弹性云主机
1、进入官网，控制中心，虚拟私有云
2、创建私有云（默认就好）vpc-server   2、创建一个子网subnet-server  
3、创建安全组（控制访问下） 
模板选择自定义：配置规则，添加规则。一个是ssh的，一个是业务端口
![图片1](https://github.com/user-attachments/assets/13a002c9-5c01-49fc-9eb8-12475c4d0667)  
设置出方向规则
![图片2](https://github.com/user-attachments/assets/e127b863-8f8d-40ff-994a-2fa1f76d0ed6)  
4、开弹性ip（比赛时候可能是开好的）  按量付费，按流量计费，带宽可以拉大  
![image](https://github.com/user-attachments/assets/4bbd9b7b-1e00-447f-931b-912eb8ddb763)  
5、开弹性云主机  
控制中心，弹性云主机->按量计费，随机分配，ecm--config，2核4gb通用型s7，Ubuntu，关闭主机安全防护  
![image](https://github.com/user-attachments/assets/b72073b6-a5af-4608-b507-295984dfae02)
网络配置：网卡、安全组、弹性ip使用已有->密码，ybh@200001130  账号是root  
![image](https://github.com/user-attachments/assets/17bbbc53-e3b3-437e-be60-1b731297f07b)  
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
![image](https://github.com/user-attachments/assets/d0121bed-928e-41a6-bd74-3c01f7fa5a0a)   
![image](https://github.com/user-attachments/assets/c5e34220-e606-453b-9738-7316398447e9)  
10、制作自启动  
cd /      先尝试启动 /opt/app-server1-demo start -config /opt/config.json  
vi /etc/systemd/system/app-server.service  
![image](https://github.com/user-attachments/assets/4dead811-03ce-40be-9037-c1a467e64f07)  
esc  :wq->systemctl start app-server->systemctl status app-server->systemctl enable app-server  
systemctl daemon-reload 之后可以reboot,也可以不用  
  
[Unit]  
Description=app-server1  
After=network-online.target  
Wants=network-online.target  

[Service]  
Type=simple  
User=root  
ExecStart=/opt/app-server1-demo start -config /opt/config.json  
ExecStop=/bin/kill -s QUIT $MAINPID  
Restart=always  
StandOutput=syslog  
StandError=inherit  

[Install]  
WantedBy=multi-user.target  

/etc/systemd/system/app-server.service  
  


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
13、弹性伸缩，伸缩配置  创建 as-app-server  
可用区都选上，通用型，s7，large.2  2c4g的，私有镜像，弹性ip不使用，密码ybh@200001130  
之后创建弹性伸缩组 as-group-app-server  
均衡分布，负载均衡启用（后端端口7000，权重100），创建  在弹性伸缩组处，  
![image](https://github.com/user-attachments/assets/d8ec77ad-ac71-4208-b6c9-bce653928b7b)  
修改，期望实例数2
![image](https://github.com/user-attachments/assets/729c5517-9d81-412f-878e-f21e5b91b2e0)  
之后等待负载均衡上线，负载均衡--后端主机组  弹性云主机处就有三个了  
![image](https://github.com/user-attachments/assets/8c176d15-aa9e-41e6-9bf1-bf581e568385)
将http://ip:7000提交


## 数据库  
1、进入  弹性云主机左上角方框，数据库--关系数据库PostgreSQL版，创建实例  
![image](https://github.com/user-attachments/assets/9415cd62-8372-4a5c-943f-8885b7de2404)
网络内选就好，安全组新建一个(端口号6543)，配置规则  
![image](https://github.com/user-attachments/assets/7a8f9755-1fdf-4ebe-9cea-24be1237a72d)  
![image](https://github.com/user-attachments/assets/a54af002-c84a-4357-8138-ffb6ef326ffa)  
![image](https://github.com/user-attachments/assets/1ee11a14-fc55-42e2-ae87-1d5f9db20196)  
在订单中查看开通情况，之后再pSQL下实例管理里，运行中，点进去  
![image](https://github.com/user-attachments/assets/ddc57d51-bcd9-4c73-a9f8-fa05c00aef0c)  
数据库管理--创建数据库：testdb 回到cmd    apt install postgresql-client  
![image](https://github.com/user-attachments/assets/6322f7e0-accc-4088-86d0-bbe8b6f9b802)  
回到浏览器实例管理，复制内网ip  登录 psql -h ip -U root -W testdb  
![image](https://github.com/user-attachments/assets/6ee75044-2306-4934-9ffe-1af9f3b1621a)  
实例管理里查看数据库端口，去安全组修改，之后  
![image](https://github.com/user-attachments/assets/9e2a1799-10c9-48a1-8d24-f0e6eaa11819)  
登录 psql -h ip -p 端口 -U root -W testdb  
![image](https://github.com/user-attachments/assets/90f0be23-9153-4f6a-9bc2-66be98e76e64)  
\l 看下列表->\C testdb选择数据库->操作->\q 退出  



## 对象存储
控制台主页，存储--对象存储，点击下边的bucket进入文件管理  
点击上传文件，上传一个写好的txt尝试，选文件后边的更多--设置读写权限，公共读  
读写权限私有，就可以生成分析链接，用curl ‘链接’就可以获取权限  
也可以用python  ->pip install boto3  ->写一个python文件 s3-test  
对象存储界面，access key管理，复制两个key；对象存储界面 域名信息复制终端节点  
![image](https://github.com/user-attachments/assets/79d5e5c4-0f9e-4848-8393-231afe802c8c)
零信任服务  :边缘安全加速平台 ，零信任服务，网络--连接器管理，创建连接器ctyun-test 
![image](https://github.com/user-attachments/assets/4ac012f6-1989-400b-b599-8c79be9dc3d6)  
云服务器安装docker   apt install docker.io  
安装好后，运行指令，完事后网页下一步->应用--应用程序，新增 -> 云服务器 ip a  查看ip  
![image](https://github.com/user-attachments/assets/b6abca9d-739a-407f-8397-ccfd21c3cae5)  
连接器管理：网络--连接器管理，右上角连接器管理  
![image](https://github.com/user-attachments/assets/6e20cdbe-4030-4801-88db-40a58b23801a)  
应用--应用配置--添加应用 ->  应用名称 styun-ssh   中等，下一步  
![image](https://github.com/user-attachments/assets/60f5b32e-c711-425c-84c9-6667a3ae943b)  
![image](https://github.com/user-attachments/assets/6bb12153-95ee-44a9-8647-b8779e23a293)  
之后回到连接器管理，  
![Uploading image.png…]()  
aone零信任 客户端下载  就可以通过之前看到的ip a的地址登录


















