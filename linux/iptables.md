## iptables

```shell
iptables -A OUTPUT -j REJECT # 关闭全部端口
iptables -A INPUT -j REJECT # 关闭全部端口

iptables -I INPUT -p tcp --dport 8001 -j ACCEPT    # 新增端口
iptables -I OUTPUT -p tcp --sport 8001 -j ACCEPT   # 新增端口
```

