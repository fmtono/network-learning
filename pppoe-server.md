# cisco1812jでpppoe接続
複数clinet対応のpppoe-serverを作りたい
## 参考
[【3分で分かるFortinet】【第7回】Ciscoルータと作るMultiple PPPoE](https://licensecounter.jp/engineer-voice/blog/articles/20190904_3fortinet7ciscomultiple_pppoe.html)
[Cisco ルータを PPPoE サーバにするには](https://sig9.hatenablog.com/entry/2015/01/15/220100)
[CiscoルータでPPPoEサーバを設定する方法(複数拠点接続)](https://www.network-engineer.info/cisco/pppoeserver/)
[PPPoE Server/Clinet (ローカル認証）Cisco IOS Config](https://server-network-note.net/2017/12/cisco-ios-pppoe-server-clinet-local/)
[Cisco 固定 IPアドレス (IP1) PPPoEサーバ 設定例](https://www.bfoip.net/jirei-PPPoE03.html)
[Cisco PPPoEサーバ、PPPoEクライアント設定例](https://www.bfoip.net/jirei-PPPoE01.html)
[Cisco IOS のバーチャルアクセス PPP 機能](https://www.cisco.com/c/ja_jp/support/docs/wan/point-to-point-protocol-ppp/14943-4.html)
[Cisco IOS XE ブロードバンド アクセス集約および DSL コンフィギュレーション ガイド](https://www.cisco.com/c/ja_jp/td/docs/rt/branchrt/6400bbaggregators/rcs/001/bba-xe-3s/bba-preparing-xe.html)



## 設定概要

- PPPoE Server 1台、PPPoE Client 2台
- Clientには固定でIPを振る



## Cisco特有

- Virtual Template インターフェイス
  ダイナミックに作成される仮想アクセスインターフェイス用の設定、PPPoEなど
- bba-group
  仮想インターフェイスと物理インターフェイスを結びつけるグループ



## パラメータ

### PPPoE Server

| ホスト名     | ループバックIP |
| ------------ | -------------- |
| pppoe_server | 10.0.0.1       |

### PPPoE Client 

| ホスト名       | アカウント           | パスワード | 払い出しIP |
| -------------- | -------------------- | ---------- | ---------- |
| pppoe_client01 | user01@example.local | user01     | 10.1.1.1   |
| pppoe_client02 | user01@example.local | user02     | 10.2.1.1   |



## コンフィグ

### Server
```
no ip domain-lookup
logging console 4

hostname pppoe_server

username user01@example.local password 0 user01
username user02@example.local password 0 user02

interface Loopback1
 ip address 10.0.0.1 255.255.255.255
exit

ip local pool user01-pool 10.1.1.1
ip local pool user02-pool 10.2.1.1

interface Virtual-Template1
 mtu 1454
 ip unnumbered Loopback1
 peer default ip address pool user01-pool
 ppp authentication chap
exit

interface Virtual-Template2
 mtu 1454
 ip unnumbered Loopback1
 peer default ip address pool user02-pool
 ppp authentication chap
exit

bba-group pppoe pppoe_user01
 virtual-template 1
exit

bba-group pppoe pppoe_user02
 virtual-template 2
exit

vlan 2
exit

vlan 3
exit

interface FastEthernet2
 switchport access vlan 2
 no ip address
exit

interface FastEthernet3
 switchport access vlan 3
 no ip address
exit

interface vlan2
 no ip address
 pppoe enable group pppoe_user01
exit

interface vlan3
 no ip address
 pppoe enable group pppoe_user02
exit
```

### Client

```
no ip domain-lookup
logging console 4

hostname pppoe_user01

interface FastEthernet0
 no ip address
 pppoe enable
 pppoe-client dial-pool-number 1
 no shutdown

interface Dialer1
 ip address negotiated
 ip mtu 1454
 encapsulation ppp
 dialer pool 1
 dialer-group 1
 ppp authentication chap callin
 ppp chap hostname user01@example.local
 ppp chap password user01
 ppp ipcp route default

dialer-list 1 protocol ip permit

```

## 動作確認

### Server

```
pppoe_server#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/32 is subnetted, 3 subnets
C       10.2.1.1 is directly connected, Virtual-Access2.2
C       10.1.1.1 is directly connected, Virtual-Access2.1
C       10.0.0.1 is directly connected, Loopback1
pppoe_server#
pppoe_server#show pppoe sess
pppoe_server#show pppoe session
     2 sessions in LOCALLY_TERMINATED (PTA) State
     2 sessions total

Uniq ID  PPPoE  RemMAC          Port                  Source   VA         State
           SID  LocMAC                                         VA-st
     19     19  0019.0684.c8f8  Vl2                   Vt1      Vi2.1      PTA
                001b.2a60.ef5e                                 UP
     20     20  001b.2a60.f426  Vl3                   Vt2      Vi2.2      PTA
                001b.2a60.ef5e                                 UP
pppoe_server#
pppoe_server#show users
    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00

  Interface    User               Mode         Idle     Peer Address
  Vi2.1        user01@example.loc PPPoE        -        10.1.1.1
  Vi2.2        user02@example.loc PPPoE        -        10.2.1.1

pppoe_server#
pppoe_server#show pppoe
pppoe_server#show pppoe sum
pppoe_server#show pppoe summary
    PTA  : Locally terminated sessions
    FWDED: Forwarded sessions
    TRANS: All other sessions (in transient state)

                      TOTAL     PTA   FWDED   TRANS
TOTAL                     2       2       0       0
Vlan2                     1       1       0       0
Vlan3                     1       1       0       0
pppoe_server#
```

### Client

```
pppoe_user01#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 10.0.0.1 to network 0.0.0.0

     10.0.0.0/32 is subnetted, 2 subnets
C       10.1.1.1 is directly connected, Dialer1
C       10.0.0.1 is directly connected, Dialer1
S*   0.0.0.0/0 [1/0] via 10.0.0.1
pppoe_user01#
pppoe_user01#show pppoe sess
pppoe_user01#show pppoe session
     1 client session

Uniq ID  PPPoE  RemMAC          Port                  Source   VA         State
           SID  LocMAC                                         VA-st
    N/A     19  001b.2a60.ef5e  Fa0                   Di1      Vi2        UP
                0019.0684.c8f8                                 UP
pppoe_user01#
pppoe_user01#show pppoe su
pppoe_user01#show pppoe summary
1 client session
pppoe_user01#
pppoe_user01#show users
    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00

  Interface    User               Mode         Idle     Peer Address
  Vi2                             PPPoE        00:02:02 10.0.0.1

pppoe_user01#
```

