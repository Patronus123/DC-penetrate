环境配置：
  靶机与kali虚拟机在同一个网段，使用NAT

信息收集：
kali使用arp-scan扫描发现网段中活跃主机：
![image](https://github.com/user-attachments/assets/797a43c4-6a3e-464f-a8b0-be7282e03fc2)
可确定靶机IP地址为192.168.27.165（kali地址为192.168.27.144）

nmap -A -v 192.168.27.144扫描主机开放端口、操作系统等信息：
![image](https://github.com/user-attachments/assets/fa830a3c-3c68-40ad-ac6c-e6cc2921991d)
nmap探测到的信息包括：靶机的22端口开放（ssh），80端口开放（http，apche httpd2.4.7）
OS：Linux 3.x|4.x（3.2-4.9）
根据收集的基本信息得到的基本思路是寻找网站漏洞，试图得到账号密码，尝试利用得到的账号密码与靶机进行ssh连接，如果成功连接则尝试提权获取flag

尝试访问靶机网页：
![image](https://github.com/user-attachments/assets/6f5cab36-dae8-4475-b3ef-5f019ce686d8)
失败，显示只能从本机访问
使用burp截包看一下：
![image](https://github.com/user-attachments/assets/5831150e-1a4e-4a24-9dcd-22882d759c23)
尝试使用伪造x-forwarded-for方法访问网站，参考：https://blog.csdn.net/xiao__gui/article/details/83054462
在bp的代理模块中使用请求头置换功能实现xff请求头伪造：
![image](https://github.com/user-attachments/assets/4a008a33-d4e6-425b-a823-4a01b610cca5)
![image](https://github.com/user-attachments/assets/088620a0-7aed-4d6a-abb1-9a5bffc288e0)

成功访问到目标网站
使用searchdir工具扫描网页目录结构：
![image](https://github.com/user-attachments/assets/d07ae70e-d0e9-4715-9e89-a5cd3a922970)
![image](https://github.com/user-attachments/assets/47585733-dcc9-45d7-9f5e-9967581f7a0c)
发现有robots.txt，看下能不能访问：
![image](https://github.com/user-attachments/assets/17a19e87-6904-4b47-a445-cc0d8e5be211)
任意useragent都可以访问根目录下的heyhoo.txt，看一下：
![image](https://github.com/user-attachments/assets/8b648a38-6831-43b8-959d-436531aa5cdd)
莫名其妙，没啥帮助
返回index，看下能不能register一个用户登录：
![image](https://github.com/user-attachments/assets/431dac9d-3252-4783-bd40-90b8e8392787)
可以看到存在已经注册的用户
注册完毕，账号名为albus，密码为123.com，返回登录：
![image](https://github.com/user-attachments/assets/83ae6173-7a58-4dd2-806a-3766e89af29a)
成功登录，观察上方url，发现后面跟了个user_id=12，怀疑能不能直接修改user_id来切换到其他用户：
user_id=1：
![image](https://github.com/user-attachments/assets/696d54f7-a2a7-4932-9c0b-c0143627fe6c)
成功切换，返回了其他用户的信息
![image](https://github.com/user-attachments/assets/68eba697-8fac-40ac-93e6-5ee10f3d875f)
并且可以直接查看网页源代码，从而获得账号密码：
将查看到的账号密码保存下来，同理得到其他用户的账号密码，生成账号密码字典如下：
这里直接在bp中把数据包发送给repeator更加方便：
user：
![image](https://github.com/user-attachments/assets/7d9154bc-0169-4672-a076-ef79af8c1883)
password：
![image](https://github.com/user-attachments/assets/b8dcb343-20ef-495a-8c81-46bf7841e823)

想起来之前用nmap扫描到靶机的ssh服务开启，尝试使用这些账号密码破解来获得ssh连接
使用hydra对目标靶机的ssh服务进行爆破：（hydra使用参考：https://blog.csdn.net/m0_59598029/article/details/133217000）
![image](https://github.com/user-attachments/assets/784edda3-baad-4c26-b309-797cfe93b330)
扫描出一组有效的账密：alice，4lic3
ssh登录一下：
![image](https://github.com/user-attachments/assets/ceafc752-d7d9-4931-91b3-b33cfd00eed2)
简单查看一下有啥文件可以访问：
![image](https://github.com/user-attachments/assets/f627fca4-61b5-47e0-b5f7-2f63a2671e3c)
可以运行php命令
使用ls -al发现alice藏了小秘密，是什么呢

