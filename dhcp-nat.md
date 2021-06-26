# DHCP,NAPT設定追加

## DHCP

### コマンドリスト

設定
(config)# ip dhcp excluded-address 始まりIP 終わりIP
(config)# ip dhcp pool プール名
(dhcp-config)# network ネットワークアドレス サブネットマスク（もしくはプリフィックス）
(dhcp-config)# default-router IPアドレス
(dhcp-config)# dns-server IPアドレス
(dhcp-config)# domain-name ドメイン名
(dhcp-config)# lease 日 時 分（もしくはinfinite無制限）

確認
#show ip dhcp pool
#show ip dhcp conflict
#show ip dhcp binding

### 設定

```
pppoe_user01(config)#ip dhcp excluded-address 192.168.1.200 192.168.1.254
pppoe_user01(config)#ip dhcp pool POOL
pppoe_user01(dhcp-config)#network 192.168.1.0 255.255.255.0
pppoe_user01(dhcp-config)#default-router 192.168.1.1
pppoe_user01(dhcp-config)#dns-server 8.8.8.8 8.8.4.4
pppoe_user01(dhcp-config)#domain-name example.local
pppoe_user01(dhcp-config)#lease 0 8 0
```

### 確認

```
pppoe_user01#show ip dhcp pool

Pool POOL :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 254
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.3          192.168.1.1      - 192.168.1.254     1
pppoe_user01#
pppoe_user01#show ip dhcp conflict
IP address        Detection method   Detection time          VRF
pppoe_user01#
pppoe_user01#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.2         0110.6f3f.660b.c7       Jan 01 1970 09:28 AM    Automatic
pppoe_user01#
```

## NAPT

### コマンドリスト

設定
(config-int)# ip nat inside
(config-int)# ip nat outside
(config)# access-list 1 permit 192.168.1.0 0.0.0.255
(config)# ip nat inside source list ACL番号 interface アウトのインターフェース名 overload

確認
#show ip nat statistics
#show ip nat translations

### 設定

```
pppoe_user01(config)#int vlan 1
pppoe_user01(config-if)#ip nat inside
pppoe_user01(config-if)#int d1
pppoe_user01(config-if)#ip nat outside
pppoe_user01(config-if)#exit
pppoe_user01(config)#ip access-list extended NAPT
pppoe_user01(config-ext-nacl)#permit ip 192.168.1.0 0.0.0.255 any
pppoe_user01(config-ext-nacl)#exit
pppoe_user01(config)#ip nat inside source list NAPT interface dialer 1 overload
```

