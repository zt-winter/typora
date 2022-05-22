```mermaid
graph TD
1(流量)-->2(sapp:流量处理平台)
2(sapp:流量处理平台)-->3(nirvana_server:整形平台)
3(nirvana_server:整形平台)-->4(nirvana_client)
3(nirvana_server:整形平台)-->5(tango_client)
4(nirvana_client)-->6(kafka)
5(tango_client)-->7(minio)
5(tango_client)-->8(redis)
```

