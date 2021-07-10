# IOSをSCPでコピー

## 参考
[[CiscoルータからSCPでIOSを抽出する方法メモ](https://poppycompass.hatenablog.jp/entry/2018/09/29/203855)

## 設定
```
interface vlan 1
ip address 192.168.1.1 255.255.255.0
no shut
exit

username cisco privilege 15 password cisco

line vty 0 4
login local
exit

hostname R1
ip domain-name r1.com

crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3
ip scp server enable

aaa new-model
aaa session-id unique
aaa authentication login default local
aaa authorization exec default local none
enable secret cisco
```

## SCP
- dir flash: ファイル名確認
- scp cisco@192.168.1.1:/ファイル名 ./
- scp ./ cisco@192.168.1.1:/ファイル名