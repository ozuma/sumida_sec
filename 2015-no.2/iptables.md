
# 複数ポートをリダイレクトする

`-m multiport`フラグはカンマ区切りで使う。範囲を示したいときはコロンを使うと良い

```
# iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 2200:2221 -j REDIRECT --to-port 2222
# iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 2223:2299 -j REDIRECT --to-port 2222
```

## 確認

iptablesの`-nL`オプションを使う

```
[root@cent6 sysconfig]# iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22459 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
```

natテーブルは`-t nat`付ける

```
[root@cent6 sysconfig]# iptables -t nat -nL 
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
REDIRECT   tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpts:2200:2221 redir ports 2222 
REDIRECT   tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpts:2223:2299 redir ports 2222 

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
```

最後に`/etc/init.d/iptables save`しておく

## ファイルへの書き方

natテーブルのPREROUTINGチェインに書く

```
[root@cent6 sysconfig]# cat iptables
*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A PREROUTING -i eth0 -p tcp -m tcp --dport 22 -j REDIRECT --to-ports 2222 
-A PREROUTING -i eth0 -p tcp -m tcp --dport 2200:2221 -j REDIRECT --to-ports 2222 
-A PREROUTING -i eth0 -p tcp -m tcp --dport 2223:2244 -j REDIRECT --to-ports 2222 
-A PREROUTING -i eth0 -p tcp -m tcp --dport 2246:2299 -j REDIRECT --to-ports 2222 
COMMIT

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -p icmp -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 2222 -j ACCEPT 
-A INPUT -p tcp -m state --state NEW -m tcp --dport 2245 -j ACCEPT 
-A INPUT -j REJECT --reject-with icmp-host-prohibited 
-A FORWARD -j REJECT --reject-with icmp-host-prohibited 
COMMIT
[root@cent6 sysconfig]# 
```

なお、丁寧に2222/tcpを避けて書いているけど、ループは発生しないので2222/tcpで内部リダイレクトしても構わない。

```
-A PREROUTING -i eth0 -p tcp -m tcp --dport 2200:2244 -j REDIRECT --to-ports 2222 
```


