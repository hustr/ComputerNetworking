# 计网第三次实验 - 组网

## 实验要求

某学校申请了一个前缀为`211.69.4.0/22`的地址块，准备将整个学校连入网络。该学校有 4 个学院，1 个图书馆，3个学生宿舍。每个学院有 20 台主机，图书馆有 100 台主机，每个学生宿舍拥有 200 台主机。 
组网需求： 

- 图书馆能够无线上网
- 学院之间可以相互访问
- 学生宿舍之间可以相互访问
- 学院和学生宿舍之间不能相互访问
- 学院和学生宿舍皆可访问图书馆

## 网络划分

图书馆：`211.69.4.0/25`，网关`211.69.4.126`

图书馆子网掩码：`255.255.255.128`，掩码的反码：`0.0.0.127`

图书馆共127个地址。

学院1：`211.69.4.128/27`，网关`211.69.4.158`

学院2：`211.69.4.160/27`，网关`211.69.4.190`

学院3：`211.69.4.192/27`，网关`211.69.4.222`

学院4：`211.69.4.224/27`，网关`211.69.4.254`

学院子网掩码：`255.255.255.224`，掩码的反码：`0.0.0.31`

每个学院31个地址，学院公共前缀：`211.69.4.128/25`

宿舍1：`211.69.5.0/24`，网关`211.69.5.254`

宿舍1：`211.69.6.0/24`，网关`211.69.6.254`

宿舍1：`211.69.7.0/24`，网关`211.69.7.254`

宿舍子网掩码：`255.255.255.0`，掩码反码：`0.0.0.255`

每个宿舍255个地址。

## 访问控制设置

DHCP设置：

```html
Router(config)#ip dhcp pool cisco2 //网络地址池名
Router(dhcp-config)#network 192.168.2.0 255.255.255.0 //DHCP地址池范围
Router(dhcp-config)#default-router 192.168.2.1 //客户端的默认网关
Router(dhcp-config)#dns-server 114.114.114.114 //客户端的DNS服务器
Router(dhcp-config)#exit
```

ACL设置

在`access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.200.0 0.0.0.255`中 `0.0.0.255`是网络`192.168.10.0`的掩码`255.255.255.0`的反码。ACL使用反码知道在网络地址的多少位需要配比。在表里， ACL允许有源地址在`192.168.10.0/24`网络和目的地地址的所有主机在`192.168.200.0/24`网络。

总结起来只有一条：学院与学生宿舍不可访问。

控制学院路由器禁止所有从学院到宿舍的包即可。

```
Router(config)#access-list 100 deny ip 211.69.4.128 0.0.0.127 211.69.5.0 0.0.0.255
Router(config)#access-list 100 deny ip 211.69.4.128 0.0.0.127 211.69.6.0 0.0.0.255
Router(config)#access-list 100 deny ip 211.69.4.128 0.0.0.127 211.69.7.0 0.0.0.255
Router(config)#access-list 100 permit ip any any //记得放行其他的所有包
Router(config)#int e1/2 //学院的出端口
Router(config-if)#ip access-group 100 out //禁止到宿舍的包出去
Router(config-if)#end
```
