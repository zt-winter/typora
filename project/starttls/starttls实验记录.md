# starttls实验记录

​	实验的基本流程：

* 收集网站域名数据
* 使用nslookup命令查看域名使用邮件服务器
* 使用nmap扫描邮件服务器，查看使用了ssl邮件传输的ip和端口。如果一个ip的某个端口是常见的邮件端口，并且支持ssl。那么我们认为它很有可能使用了starttls，并且可能有论文作者说的漏洞。
* 处理nmap的扫描结果
* 使用论文作者提供的攻击注入脚本测试所有使用ssl的邮件端口
* 处理得到的最后结果

​	首先我们在alexa网站上收集了大约61万个域名（即文件top-1m.csv）。接着我们写了脚本（mx.sh），使用nslookup找到网站的邮件服务器。结果保存在mxresult和domain中，大概有7万条记录。

​	然后我们使用` nmap -F -oX - {host} --script ssl-cert`查看有哪些网站的。由于扫描的网站数量太大，nmap配置做了些针对性

```
-F 扫描更少的端口，只扫描基本常用端口，加快一次扫描速度
-oX 将得到的扫描结果保存为xml文件格式，便于后续处理
--script ssl-cert 这是在nmap官网上验证端口是否支持ssl的一个脚本
```

同时师兄还做了一个多进程的脚本，实现多进程执行nmap扫描命令，加速了实验过程，即代码文件中mp_nmap.py。执行mp_nmap.py的每一条扫描记录都保存在file文件夹中。

​	然后在file文件夹下写了脚本1.py，处理每个xml文件，将所有的结果保存到file文件夹下的result中。一个ip可能有多个端口都开放了ssl邮件服务，像25,993,995,110端口都开放。这里一条记录对应一个ip的一个端口。result中结果总共15575

​	由于论文作者给出的代码是直接以虚拟机的方式给出，然后将result文件复制到虚拟机中，然后再写脚本command.py，针对不同的端口执行不同的攻击注入命令，然后将结果写到finalResult中，然后根据结果的不同，写class.py进行分类。结果分为四类

- 攻击注入成功 173个
- 没有完全成功注入，但程序判断可能有攻击注入了 90个
- 没有注入成功 2349个
- 在攻击注入过程中，服务器关闭了连接 2662个

分别在class文件夹中injection、no_injection、probably_injection、close_connection。还要很多攻击并没有服务器响应(time out)，应该也可以归为攻击失败。



后续的问题

* 在mxresult和domain中有许多重复邮件服务器，如mx.sina.net。即多个网站的mx记录相同
* 一个邮件服务器地址有多个端口开放，需要分别统计。
* 用nmap --ssl-cert的脚本只能判断开启了ssl，并不能统计是否开启了starttls