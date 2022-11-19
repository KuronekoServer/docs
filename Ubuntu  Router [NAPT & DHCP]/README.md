# Ubuntu  Router [NAPT & DHCP] 
Ubuntuルーター【NAPT＆DHCP】の場合 

```
You should see this on Windows PC.
If you do not, the diagram may collapse.
Network______________________________________________________________________________________________192.168.11.0/24
                      |
             [eth0 192.168.11.15]
             [  Ubuntu Router   ] 
             [eth0 192.168.20.1 ]
                      |
Private_______________|______________________________________________________________________________192.168.20.0/24
```
全てrootで実行してください。（スムーズに進みます）


netplanでeth1のipとdnsを設定します。
`/etc/netplan/00-installer-config.yaml`
```
network:
  ethernets:
    eth0:
      addresses:
      - 192.168.11.15/24
      routes:
      - to: default
        via: 192.168.11.1
      nameservers:
        addresses:
        - 1.1.1.1
        - 1.0.0.1
        search: []
    eth1:
      addresses:
      - 192.168.20.1/24
      dhcp4: no
      nameservers:
        addresses:
        - 1.1.1.1
        - 1.0.0.1
        search: []
  version: 2
```
有効化

```netplan apply```


`/etc/sysctl.conf`
アンコメントしてください

```net.ipv4.ip_forward=1```
有効化

```sudo sysctl -p```



```
apt install iptables-persistent
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.20.0/24 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```

```apt install isc-dhcp-server```

`/etc/dhcp/dhcpd.conf`

```
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
option routers 192.168.20.1;
option domain-name-servers 1.1.1.1;

subnet 192.168.20.0 netmask 255.255.255.0 {
      range 192.168.20.101 192.168.20.199;
}
```

```
sudo systemctl enable isc-dhcp-server.service
sudo systemctl start isc-dhcp-server.service
```