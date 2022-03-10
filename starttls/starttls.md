# starttls安全问题测量

​	本次实验主要是测量alexa排名网站的邮件服务器使用starttls存在的安全问题。主要参考文献是USENIEX2021年的一篇论文《*Why TLS is better without STARTTLS:*  *A Security Analysis of STARTTLS in the Email Context*》。该文测量发现大量邮件服务器由于历史兼容原因使用了starttls技术，并存在安全漏洞。文章详细内容可以参考姜鹏辉师兄之前的[2021年第30期讨论班ppt](https://docs.mesalab.cn/pages/viewpage.action?pageId=27692690)和师兄之前对starttls的[调研工作](https://docs.mesalab.cn/pages/viewpage.action?pageId=27696309)。

​	实验的基本流程可以分为三个步骤

1. 获取alexa排名网站的邮件服务器
2. 获取支持ssl邮件服务器及开放的邮件服务端口
3. 测量邮件服务是否存在安全问题

分别对应实验代码的三个部分

```zsh
# zt @ zt in ~/code/starttls [11:34:38]
$ tree -L 1
.
├── command-injection-tester	//第三部分
├── email_server	//第一部分
└── ssl_test	//第二部分
```



### 获取alexa排名网站的邮件服务器

```zsh
# zt @ zt in ~/code/starttls/email_server [11:41:07]
$ tree -L 1
.
├── deduplicate.py
├── domain
├── domaindup
├── duplicate
├── mxresult
├── mx.sh
└── top-1m.csv
```

首先我们从alexa网站收集了前618947个网站信息，然后使用nslookup（mx.sh脚本）查询网站的邮件服务器地址，将查询记录保存为mxresult，然后提取其中的邮件服务器域名信息保存为domain。由于存在不同网站使用的同一个邮件服务器，所以使用deduplicate.py将所有重复的邮件服务器域名只保存一个，得到去重后的domaindup；同时将重复的网站记录下来保存为duplicate。查看duplicate可以发现大量不同的网站应该都使用了google的邮件服务器。

```
$ vim duplicate
aspmx.l.google.com
alt2.aspmx.l.google.com
alt4.aspmx.l.google.com
alt3.aspmx.l.google.com
alt1.aspmx.l.google.com
```

最后我们从前62万个网站中得到了不重复的邮件服务器域名17330个，结果保存在domaindup中。

### 获取支持ssl的邮件服务器及开放的邮件服务端口

```zsh
# zt @ zt in ~/code/starttls/ssl_test [12:40:59]
$ tree -L 1
.
├── base-log.log
├── mp_nmap.py
├── port
├── port.py
├── result
├── result.py
├── test.py
└── xml
```

​		首先使用nmap官网提供的ssl脚本可以检测服务器支持ssl的端口，这样做可以快速从获取的网站邮件服务器列表中找到可能支持starttls的邮件服务器，便于后续进行攻击测试。所以执行mp_nmap.py处理email_server中的domaindup文件，将所有的邮件服务器域名经过nmap扫描之后的结果以xml文件格式存放在xml文件夹中。这里由于邮件服务域名太多，所以使用了多进程处理。

​		然后使用result.py对产生的xml文件进行处理。主要思路是如果一个邮件服务器域名对应ip的邮件协议端口支持ssl，那么认为有很大可能使用了starttls，值得后续进行攻击测试。

```
//这里说明下邮件协议端口
smtp:25,465,587
imap:143,993
pop3:110,995
```

在观察扫描结果中发现有邮件服务器支持多种不同的邮件协议，如同时支持smtp和imap，所以在处理过程中按照不同的协议将结果保存在result文件夹的smtp、imap、pop3文件中。同时还观察到一个协议在一台邮件服务器上有多个端口开放，比如smtp协议，25、465、587端口全部开放了，为了避免重复计算，将同一协议的不同端口按照一个协议处理，不重复计算。同时使用port.py文件将nmap扫描结果按照端口不同进行分类并保存在port文件夹中，方便与result.py的结果进行对比。

最后得到的统计结果如下所示

| 协议:端口号 | 去掉重复ip后的数量              | 没有去掉重复ip的数量 |
| :---------- | ------------------------------- | -------------------- |
| smtp:25     | 3238                            | 4599                 |
| smtp:465    | 2274                            | 2466                 |
| smtp:587    | 2067                            | 2214                 |
| smtp        | <font color=red>**4223**</font> |                      |
| imap:143    | 2007                            | 2149                 |
| imap:993    | 2456                            | 2652                 |
| imap        | <font color=red>**2504**</font> |                      |
| pop3:110    | 1950                            | 2112                 |
| pop3:995    | 2337                            | 2540                 |
| pop3        | <font color=red>**2406**</font> |                      |

同时我们还可以看出存在同一个ip地址上有多个邮件服务域名的情况。smtp协议的25端口去掉重复ip地址造成的差别还非常大，数量从4599到3238。

后续我们将会测试红色标记的ip和端口是否存在starttls安全漏洞。

### 测量邮件服务是否存在安全问题

```zsh
//commandimap.py和commandsmtp.py是临时文件
//logs是command_injection_tester.py执行过程中产生的日志
//readme是command_injection_tester.py的说明
# zt @ zt-zjh in ~/code/starttls/command-injection-tester on git:main x [20:49:14]
$ tree -L 1
.
├── class.py
├── classPort.py
├── commandimap.py
├── command_injection_tester.py
├── command.py
├── commandsmtp.py
├── finalResult
├── logs
└── README.m
```



​		《*Why TLS is better without STARTTLS:*  *A Security Analysis of STARTTLS in the Email Context*》的作者写了starttls漏洞测试的代码，可以用来测试邮件服务器是否有starttls安全漏洞。执行command.py，对第二部分发现的邮件服务器端口进行测试，将测试结果保存到finalResult中。然后执行class.py和classPort.py对finalResult中测试结果进行统计。

| 协议端口数     | 邮件服务器                      | 存在安全问题（存在安全漏洞+可能存在安全漏洞） | 所占比例    |
| -------------- | ------------------------------- | --------------------------------------------- | ----------- |
| smtp:25        | 3238                            |                                               |             |
| smtp:465       | 2274                            |                                               |             |
| smtp:587       | 2067                            |                                               |             |
| smtp           | <font color=red>**4223**</font> | 46+41                                         | 1.1% + 1%   |
| imap:143       | 2007                            |                                               |             |
| imap:993       | 2456                            |                                               |             |
| imap           | <font color=red>**2504**</font> | 17+0                                          | 0.6% + 0%   |
| pop3:110       | 1950                            |                                               |             |
| pop3:995       | 2337                            |                                               |             |
| pop3           | <font color=red>**2406**</font> | 4+2                                           | 0.2% + 0%   |
| smtp+imap+pop3 | 9133                            | 67+43                                         | 0.7% + 0.4% |



### 评估

​		在论文中作者对全网ipv4进行扫描，扫描结果如下

![论文数据](D:\typora\starttls\论文数据.png)

作者按照协议总共发现了15729835个邮件服务器，其中有321254个邮件服务存在安全问题，占2%。这与我们的这次测量的结果有差别。区别是作者对pop3和imap协议测量时只测量110和143端口，对993和995端口没有统计。另外作者测的是全网ip，而我们是测量alexa排名网站。还有论文发表后，可能有安全维护的邮件服务器会修改安全措施。这些应该都是误差的来源。

​		作者漏洞测试代码基本功能作者的核心代码主要是ServerTest类，下面主要是两个功能函数：

* sanity_test:测试该ip端口是否支持tls
* injection_test:构造starttls攻击的数据包，并在tls握手前发送出去，根据返回结果判断攻击测试是否成功

```python

115 	def _sanity_test(self, logger: logging.Logger, context: ssl.SSLContext, **kwargs):
116         try:
117             with socket.create_connection((self.hostname, self.port), timeout=self.timeout) as sock:
118                 logger.info("Sanity test...")
119                 try:
120                     resp = self._recv_multiple_segments(sock, logger)
121                     log_trace(logger, resp, incoming=True)
122                     for payload in kwargs.get("pretls"):
123                         log_trace(logger, payload, incoming=False)
124                         sock.send(payload.encode())
125                         resp = self._recv_multiple_segments(sock, logger)
126                         log_trace(logger, resp, incoming=True)
127                     with context.wrap_socket(sock=sock, server_hostname=self.hostname) as ssock:
128                         logger.debug("<----- TLS Handshake ----->")
129                         for payload in kwargs.get("posttls"):
130                             ssock.send(payload.encode())
131                             log_trace(logger, payload, incoming=False)
132                             resp = self.recv_from_ssl(ssock)
133                             log_trace(logger, resp, incoming=True)
134                 except socket.timeout as e:
135                     msg = f"Sanity test failed. The connection to {self.hostname}:{self.port} timed out."
136                     logger.error(red(msg))
137                     return False
138                 except ssl.SSLError as e:
139                     msg = f"Sanity test failed. Could not perform TLS handshake with {self.hostname}:{self.port}."
...

165			def _injection_test(self, logger: logging.Logger, context: ssl.SSLContext, **kwargs) -> bool:
168 			try:
169             with socket.create_connection((self.hostname, self.port), timeout=self.timeout) as sock:
170                 logger.info("Testing for command injection...")
171                 resp = self._recv_multiple_segments(sock, logger)
172                 log_trace(logger, resp, incoming=True)
173                 pretls = kwargs.get("pretls")
174                 for payload in pretls:
175                     log_trace(logger, payload, incoming=False, formatter=red if payload == pretls[-1] else None)
176                     sock.send(payload.encode())
177                     resp = self._recv_multiple_segments(sock, logger)
178                     log_trace(logger, resp, incoming=True)
179                 try:
180                     with context.wrap_socket(sock=sock, server_hostname=self.hostname) as ssock:
181                         logger.debug("<----- TLS Handshake ----->")
182                         ssock.setblocking(False)
183
184                         resp = self.recv_from_ssl(ssock)
185                         if resp:
186                             log_trace(logger, resp, incoming=True, formatter=red)
187                             if kwargs.get("test")(resp):
188                                 logger.warning(red("Command injection here!"))
189                             else:
190                                 logger.warning(red("Probable command injection. Response looks different than expected."))
191
192                         else:
193                             logger.debug("No response in encrypted context, trying real command now ...")
194                             payload = kwargs.get("command")
195                             ssock.send(payload.encode())
196                             log_trace(logger, payload, incoming=False)
...
```

### 问题

​		目前没有找到从能直接识别服务器是否存在starttls的方法，从作者的代码中也没有看到能直接判断starttls的实现方法。

