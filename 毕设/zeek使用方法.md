# zeek使用方法

[官网使用说明](https://docs.zeek.org/en/v4.2.0)

读取pcap数据包，并保存日志文件

```
zeek -r ./mnt/xmr数据集/12.2.xmr.pcapng LogAscii::logdir=/home/zt/mnt/xmr数据集/12.2xmr
```

zeek不同日志文件含义

* conn.log : 