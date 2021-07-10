# Context-Based Access Control
Contextに応じたACL、ステートフルインスペクション



## 参考

[コンテキストベース アクセス制御: 概要と設定](https://www.cisco.com/c/ja_jp/support/docs/security/ios-firewall/13814-32.html)
[Internet - PPPoE Cisco Config](https://www.infraexpert.com/study/wan10.html)
[CBAC ( Context-Based Access Control )](https://www.infraexpert.com/study/aclz15.html)



## 方針

- 外部->内部はdeny
- 内部->外部はpermit
- 内部->外部はTCP、UDP、ICMP、FTPを検査



## 設定概要

- 検査(inspect)するプロトコルを指定
- 外部->内部をdenyするACL作成
- InspectとACLをインターフェースに適用



## 設定例

```
inspect
(config)# ip inspect name OutBound tcp
(config)# ip inspect name OutBound udp
(config)# ip inspect name OutBound icmp
(config)# ip inspect name OutBound ftp

deny ACL
(config)# ip access-list extended External
(config-ext-nacl)# deny ip any any

interface
(config)# interface dialer1
(config-if)# ip access-group External in
(config-if)# ip inspect OutBound out
```



## 確認

```
#show ip inspect ?
  all           Inspection all available information
  config        Inspection configuration
  interfaces    Inspection interfaces
  mib           FW MIB specific show commands
  name          Inspection name
  sessions      Inspection sessions
  sis           Inspection sessions (debug version)
  statistics    Inspection statistics
  tech-support  Inspection technical support

# show ip inspect config
Session audit trail is disabled
Session alert is enabled
one-minute (sampling period) thresholds are [unlimited : unlimited] connections
max-incomplete sessions thresholds are [unlimited : unlimited]
max-incomplete tcp connections per host is unlimited. Block-time 0 minute.
tcp synwait-time is 30 sec -- tcp finwait-time is 5 sec
tcp idle-time is 3600 sec -- udp idle-time is 30 sec
tcp reassembly queue length 16; timeout 5 sec; memory-limit 1024 kilo bytes
dns-timeout is 5 sec
Inspection Rule Configuration
 Inspection name OutBound
    tcp alert is on audit-trail is off timeout 3600
    udp alert is on audit-trail is off timeout 30
    icmp alert is on audit-trail is off timeout 10
    ftp alert is on audit-trail is off timeout 3600
```

