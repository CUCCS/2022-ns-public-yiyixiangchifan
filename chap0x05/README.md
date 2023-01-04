# 基于 Scapy 编写端口扫描器

## 实验目的

- 掌握网络扫描之端口状态探测的基本原理

## 实验环境

- python + [scapy](https://scapy.net/)

## 实验要求

- [x] 禁止探测互联网上的 IP ，严格遵守网络安全相关法律法规
- [x] 完成以下扫描技术的编程实现
  - TCP connect scan / TCP stealth scan
  - TCP Xmas scan / TCP fin scan / TCP null scan
  - UDP scan
- [x] 上述每种扫描技术的实现测试均需要测试端口状态为：`开放`、`关闭` 和 `过滤` 状态时的程序执行结果
- [x] 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；
- [x] 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
- [x] （可选）复刻 `nmap` 的上述扫描技术实现的命令行参数开关

## 实验过程

#### TCP connect scan

- 实验预期获得的结果：倘若攻击者向靶机发送SYN包，能完成三次握手，收到ACK,则端口为开发状态；若只收到一个RST包，则端口为关闭状态；倘若什么都没收到，即为端口过滤状态。
-  scapy编程python代码实现 
- 第一次执行代码后，显示端口为关闭，同时我们进行抓包并对抓包结果进行观察
- 接着将该端口开启后再执行代码，同时进行抓包
- 用nmap进行扫描
- 在靶机中过滤80端口的tcp包
- 再执行该代码，得到端口为过滤状态，抓包结果中确实只有一个接收到的TCP包

**CODE**

```
from scapy.all import *

dst_ip = "172.16.111.102"
dst_port = 80

resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="S"),timeout=10)

if resp is None:
    print("Filtered")
elif(resp.haslayer(TCP)):
    if(resp.getlayer(TCP).flags==0x12):
        send_ret = sr(IP(dst=dst_ip)/TCP(dport=dst_port,flags="AR"),timeout=10)
        print("Open")
    elif(resp.getlayer(TCP).flags==0x14):
        print("closed")
```

![](img/evn.png)

##### 端口关闭

- 使用scapy

  ![](img/tcp-connect-scapy.png)

- 使用nmap

  ![](img/tcp-connect-nmap.png)

##### 端口开启

- 使用scapy

  ![](img/tcp-connect-scapy-open.png)

- 使用nmap

  ![](img/tcp-connect-nmap-open.png)

##### 端口过滤

- 使用scapy

  ![](img/tcp-connect-scapy-filter.png)

- 使用nmap

  ![](img/tcp-connect-nmap-filter.png)

#### TCP stealth scan

**CODE**

```
from scapy.all import *

dst_ip = "172.16.111.102"
dst_port = 80

resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="S"),timeout=10)

if resp is None:
    print("Filtered")
elif(resp.haslayer(TCP)):
    if(resp.getlayer(TCP).flags==0x12):
        send_ret = sr(IP(dst=dst_ip)/TCP(dport=dst_port,flags="R"),timeout=10)
        print("Open")
    elif(resp.getlayer(TCP).flags==0x14):
        print("closed")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print("Filtered")
```

##### 端口关闭

- 使用scapy

  ![](img/tcp-ste-scapy-close.png)

- 使用nmap

  ![](img/tcp-ste-nmap-close.png)

##### 端口开启

- 使用scapy

  ![](img/tcp-ste-scapy-open.png)

- 使用nmap

  ![](img/tcp-ste-nmap-open.png)

##### 端口过滤

- 使用scapy

  ![](img/tcp-ste-scapy-filter.png)

- 使用nmap

  ![](img/tcp-ste-nmap-filter.png)

#### TCP Xmas scan

**CODE**

```
from scapy.all import *

dst_ip = "172.16.111.102"
dst_port = 80

resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="FPU"),timeout=10)

if resp is None:
    print("Open|Filtered")
elif(resp.haslayer(TCP)):
    if(resp.getlayer(TCP).flags==0x14):
        print("Closed")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print("Filtered")
```

##### 端口关闭

- 使用scapy

  ![](img/tcp-xmas-scapy-close.png)

- 使用nmap

  ![](img/tcp-xmas-nmap-close.png)

##### 端口开启

- 使用scapy

  ![](img/tcp-xmas-scapy-open.png)

- 使用nmap

  ![](img/tcp-xmas-nmap-open.png)

##### 端口过滤

- 使用scapy

  ![](img/tcp-xmas-scapy-filter.png)

- 使用nmap

  ![](img/tcp-xmas-nmap-filter.png)

#### TCP fin scan

**CODE**

```python
from scapy.all import *

dst_ip = "172.16.111.102"
dst_port = 80

resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="F"),timeout=10)

if resp is None:
    print("Open|Filtered")
elif(resp.haslayer(TCP)):
    if(resp.getlayer(TCP).flags==0x14):
        print("Closed")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print("Filtered")
```

##### 端口关闭

- 使用scapy

  ![](img/tcp-fin-scapy-close.png)

- 使用nmap

  ![](img/tcp-fin-nmap-close.png)

##### 端口开启

- 使用scapy

  ![](img/tcp-fin-scapy-open.png)

- 使用nmap

  ![](img/tcp-fin-namp-open.png)

##### 端口过滤

- 使用scapy

  ![](img/tcp-fin-scapy-filter.png)

- 使用nmap

  ![](img/tcp-fin-nmap-filter.png)

#### TCP null scan

**CODE**

```python
from scapy.all import *

dst_ip = "172.16.111.102"
dst_port = 80

resp = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags=""),timeout=10)

if resp is None:
    print("Open|Filtered")
elif(resp.haslayer(TCP)):
    if(resp.getlayer(TCP).flags==0x14):
        print("Closed")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]):
            print("Filtered")
```

##### 端口关闭

- 使用scapy

  ![](img/tcp-null-scapy-close.png)

- 使用nmap

- ![](img/tcp-null-nmap-close.png)

##### 端口开启

- 使用scapy

  ![](img/tcp-null-scapy-open.png)

- 使用nmap

  ![](img/tcp-null-nmap-open.png)

##### 端口过滤

- 使用scapy

  ![](img/tcp-null-scapy-filter.png)

- 使用nmap

  ![](img/tcp-null-nmap-filter.png)

#### UDP scan

**CODE**

```python
from scapy.all import 

dst_ip = "172.16.111.102"
dst_port = 53

resp = sr1(IP(dst=dst_ip)/UDP(dport=dst_port)/DNS(opcode=2),timeout=10) #受害者主机开启了DNS服务，想要判断开启状态需要发送DNS询问包

if resp is None:
    print("Open|Filtered")
elif(resp.haslayer(UDP)):
    print("Open")
elif(resp.haslayer(ICMP)):
    if(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code==3)):
        print("Closed")
    elif(int(resp.getlayer(ICMP).type)==3 and int(resp.getlayer(ICMP).code) in [1,2,9,10,13]):
        print("Filtered")
```

##### 端口关闭

- 使用scapy

  ![](img/udp-scapy-close.png)

- 使用nmap

  ![](img/udp-nmap-close.png)

##### 端口开启

- 使用scapy

  ![](img/udp-scapy-open.png)

- 使用nmap

  ![](img/udp-nmap-open.png)

##### 端口过滤

- 使用scapy

  ![](img/udp-scapy-filter.png)

- 使用nmap

  ![](img/udp-nmap-filter.png)

## 实验总结

- 扫描实验中，TCP connect/TCP stealth扫描属于开放扫描，TCP Xmas/TCP FIN/TCP NULL扫描属于隐蔽扫描，UDP 扫描也属于开放扫描。 

- 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；

  - 基本相符，但是在使用编程扫描的时候比课本中实现的更详细，例如UDP扫描中,我们可以看到课本中提供的只是在收到UDP包的情况下可以确认端口为开放状态下，编程实现的过程中还可以通过ICMP包中是否包含UDP包进行判断端口是否为开放状态。这是由于编程实现可以更加详细的分析抓到的包。

-  总结各种扫描方式所对应的状态 

  |     扫描方式/端口状态     |              开放               |      关闭       |      过滤       |
  | :-----------------------: | :-----------------------------: | :-------------: | :-------------: |
  |  TCP connect/TCP stealth  | 完整的三次握手，能抓到ACK&RST包 | 只收到一个RST包 | 收不到任何TCP包 |
  | TCP Xmas/TCP FIN/TCP NULL |         收不到TCP回复包         |  收到一个RST包  | 收不到TCP回复包 |
  |            UDP            |          收到UDP回复包          | 收不到UDP回复包 | 收不到UDP回复包 |

## 参考资料

[网络扫描实验报告](https://blog.csdn.net/lemonalla/article/details/105592229) 