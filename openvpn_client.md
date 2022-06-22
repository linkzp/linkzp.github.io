# OpenVPN全流量转发配置

---

author: "梅刚"

date: "2022-06-22"

---



> 假设已经完成基本的OpenVPN服务器和客户端配置



### 1. 客户端配置

客户端配置很简单，只需要在配置的中添加命令： 

```
redirect-gateway def1
```

如果 OpenVPN服务器和客户端在同一个局域网，则替换成:

``` 
redirect-gateway local def1
```

上述命令也可以通过服务器自动下发，无需在每个客户端上配置。服务器推送命令如下

``` 
push "redirect-gateway def1"
```

和

``` push "redirect-gateway local def1"
push "redirect-gateway local def1"
```



配置有上述命令的客户端在VPN客户端成功连接服务器之后，回向本地操作系统添加如下路由（假设VPN分配网段为 192.168.99.0/24, 本地IP为192.168.99.9）：

``` 0.0.0.0          0.0.0.0     192.168.99.1     192.168.99.9     11
0.0.0.0          0.0.0.0     192.168.99.1     192.168.99.9     11
```

即将 VPN网关作为本地的默认网关，这样流量默认路由到服务器



## 2. 服务器端配置

1. 通过 iptables 开启 SNAT模式，将客户端发送的请求转发到外网

   ``` 
   iptables -t nat -A POSTROUTING -s 192.168.99.0/24 -o eth0 -j MASQUERADE

​		

2. 在服务器配置里添加 下发 DNS配置的命令

   ```
   push "dhcp-option DNS 8.8.8.8"
   ```

   此处选择适合于 VPN服务器的域名服务器，疑问对DNS服务器的访问也会走VPN流量。当然客户端也可以单独配置

