# std::net

TCP/UDP 通信的网络原语

此模块提供传输控制和用户数据报协议的网络功能，以及IP和套接字地址类型。

组成：

- `TcpListener`  `TcpStream` 提供TCP通信功能
- `UdpSocket`   提供通过UDP进行通信的功能
- `IpAddr`  表示IPv4 IPv6 的地址：Ipv4Addr和Ipv6Addr分别是IPv4和IPv6地址
- `SocketAddr`  表示IPv4或IPv6的套接字地址；SocketAddrV4和SocketAddrV6分别是IPv4和IPv6套接字地址
- `ToSocketAddrs` 是与网络对象(如TcpListener、TcpStream或UdpSocket)交互时用于通用地址解析的trait
- 或其他类型的返回方法

### Traits

`ToSocketAddrs`  对象的一种特征，可将其转换或解析为一个或多个SocketAddr值。

### Enums

| IpAddr             | IP 地址 包含 IPv4  IPv6                      |
| ------------------ | -------------------------------------------- |
| Shutdown           | 可以传递给TcpStream:：shutdown方法的可能值。 |
| SocketAddr         | internet套接字地址，IPv4或IPv6。             |
| Ipv6MulticastScope |                                              |

### structs

| AddrParseError | 解析IP地址或套接字地址时可以返回的错误 |
| -------------- | -------------------------------------- |
| Incoming       | 无限接受迭代器或连接。                 |
| Ipv4Addr       | IPv4地址                               |
| Ipv6Addr       | IPv6地址                               |
| SocketAddrV4   | IPv4 套子地址                          |
| SocketAddrV6   | IPv6套接字地址                         |
| TcpListener    | TCP套接字服务器，侦听连接              |
| TcpStream      | 本地和远程套接字之间的TCP流。          |
| UdpSocket      | UDP套接字。                            |

