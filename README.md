# Surge 使用手册

Surge 这个是 iOS9 上的神器，作者虽然在[官网](http://surge.run/manual/)说明 Surge 是开发者的网络调试工具，但是这个工具自打上架以来，其最广泛的应用场景绝对包括 **iOS9梯子** 这一刚需，iOS9 上有了 Surge 之后，iPhone 越狱的理由又少了一条。

### 主要功能

Surge 的主要功能包括以下：

- 截获 iOS 上的所有应用程序的 HTTP/HTTPS/TCP 流量，将其重定向到 HTTP/HTTPS/SOCK5 代理服务器转发。
- 重载 iOS 蜂窝网络下的 DNS 服务器配置，使得 iOS 设备工作在蜂窝网络之下时，开发者也能够配置 DNS 服务器。
- 记录设备所发出/接收的 HTTP 请求/应答首部信息。
- 配置基于 domain, domain suffix, domain keyword, CIDR IP 以及 GeoIP 的过滤规则。
- 对设备上 WIFI、蜂窝网络以及代理连接会话的流量进行统计以及测速。
- 从 URL 或 iTunes 导入/导出配置文件。
- 重度使用场景下有很高的性能以及适应性。
- 基于 domain 规则屏蔽广告。
- 完全兼容蜂窝网络。

与 iOS9之前需要越狱才能使用的 Shadowsocks 比较，Surge 在 iPhone 有强大的可配置性（过滤规则）、数据统计以及日志，较为遗憾的是由于 iOS 沙盘的限制，Surge 无法实现 ``应用内代理`` 功能，希望后续有对应的越狱插件可以弥补这个不足。

### 软件构架

为了让使用者能够更好地理解 Surge 的各项配置的作用，作者在官网上特意花了一个章节描述其软件构架

![Surge Architecture](http://surge.run/manual/Surge-Architecture.png)

Surge 主要包括 **Surge proxy server** 与 **Surge TUN interface** 这两个组件，分别负责 HTTP/HTTPS 代理以及 IP 代理。

- Surge Proxy server: Surge 功能开启后将自动代理 iOS 设备上所有的 HTTP/HTTPS ，同时对所有的 HTTP/HTTPS 代理使用同一个代理会话，以最高限度提高 Surge 的代理性能。Surge 可以通过 ``skip-proxy`` 指定哪些流量不被 proxy server 所代理

- Surge TUN interface: iOS 上大部分应用程序的网络交互使用 HTTP/HTTPS，但也存在部分应用程序(如:iOS邮件客户端、Facebook客户端）等应用程序使用其他的通讯方式（如：SPDY等），这些应用无法被 Proxy server 所代理，这就需要使用更为底层的TUN interface 隧道方式。Surge 可以通过 ``bypass-tun`` 指定哪些数据流量不送 TUN interface 处理

		> 注意: 当前 Surge TUN interface 只能处理 TCP 流量，对于 ICMP、UDP 流量将被直接丢弃，因此需要通过配置 ``bypass-tun`` 选项来放行这些流量。

### 配置文件

		Surge 可以通过 ``URL下载`` 和 ``iTunes更新`` 这两种方式导入配置文件，配置文件由多个段(Section)构成，主要包括:General, Proxy, Rule等几个部分。

#### 通用配置

		通用配置段``[general]``的主要配置如下:

		```
		[General]
		loglevel = verbose
		bypass-system = true
		skip-proxy = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12, localhost, *.local
		bypass-tun = 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
		dns-server = 8.8.8.8, 8.8.4.4
		```

##### loglevel 

Surge输出的日志信息级别，按日志输出从多到少分别为 verbose,info,notify与warning，默认为 notify，一般情况下不要设置为 verbose ，这会大幅度降低 Surge 的性能，如果只是做为梯子使用，建议设置为输出最少的 warning 级别。

##### bypass-system

全局忽略配置，此配置列表中指定的服务器请求流量将不会被 Surge 所处理（包括 Proxy server 与 TUN interface），在一般使用中，请将以下服务器列入 bypass-system ，否则可能会出现一些奇怪问题（比如：应用程序的推送被延迟），添加 Apple 常用的服务器域名清单：

```
api.smoot.apple.com
configuration.apple.com
xp.apple.com
smp-device-content.apple.com
guzzoni.apple.com
captive.apple.com
*.ess.apple.com
*.push.apple.com
*.push-apple.com.akadns.net
```

同时也将 Apple 的服务器 IP 地址段过滤规则加上 `IP-CIDR, 17.0.0.0/8, DIRECT, no-resolve` 。

##### skip-proxy

skip-proxy 用于指定哪些服务器的**域名和IP**不被 Proxy server 代理，常用的有以下配置方式:

- 顶级域名，如 apple.com
- 包含特定关键字的域名，如 *apple.com
- 特定子域名，如 store.apple.com
- 特定的IP地址以及IP地址段，如 192.168.2.11,192.168.2.*,192.168.2.0/24

做梯子使用时，可以考虑在这里添加常用的应用的域名以及整个中国区域的 IP 地址段。

> 需要特别说明的是主流应用程序，特别是现象级的应用，一般都广泛采用 CDN 技术对服务器请求进行加速，这种情况下同个域名在不同的地区及同地区不同时间都可能会请求到不同 IP 地址，因此要通过添加域名的方式匹配，而不能采用简单地添加指定 IP 地址。

##### bypass-tun

bypass-tun 用于指定哪些服务器的域名/IP不被 TUN interface 转发，配置的方式同 skip-proxy，需要特别注意的是 Proxy server 所代理的流量不会被 TUNN interface 代理，因此在配置 bypass-tun 时需要结合 skip-proxy 进行综合考虑。

##### dns-server

dns-server 用于指定 iOS 设备所使用的域名解析服务器，iOS 本身是不允许修改蜂窝网络下的域名解析服务器，通过使用 Surge 的 dns-server 配置则可以达到修改蜂窝网络域名解析服务器的目的。

#### 代理配置

代理配置段 ``[Proxy]`` 用于配置 HTTP, HTTPS 以及 SOCKS5 代理服务器（梯子的关键），配置的例子如下:

```
[Proxy]
ProxyA = socks5, example.server.com, 3129
ProxyB = http, example.server.com, 3128
ProxyC = https, example.server.com, 443, username, password
```

这个例子中，配置了三个服务器，分别对应于 HTTP, HTTPS 以及 SOCKS5 这三种代理机制，Surge 可以针对这三个代理指定多套不同的配置：

- 指定 HTTP/HTTPS 代理的用户名与密码（可选）
- 指定 HTTPS 代理的加密机制，如 ``ProxyC = https, example.server.com, 443, username, password, cipher=TLS_RSA_WITH_AES_128_GCM_SHA256``
- 指定 TLS 会话重传，HTTPS 代理默认打开 ``False Start``与``ECN``功能。（未来版本将支持 on/off 配置）

#### 过滤规则

``[Rule]``段用来配置 Surge 的过滤规则，规则按序存储，匹配时从上到下，先匹配先生效，因此对于常用的规则应该放在前头，以提高性能，配置的例子如下:

```
[Rule]
DOMAIN-SUFFIX,company.com,ProxyA
DOMAIN-KEYWORD,google,DIRECT
GEOIP,US,DIRECT
IP-CIDR,192.168.0.0/16,DIRECT
FINAL ProxyB
```

Surge 支持 DOMAIN,DOMAIN-SUFFIX,DOMAIN-KEYWORD,GEOIP,IP-CIDR与FINAL 这六种规则，每条规则都由 类型，匹配域以及行为 这三个部分构成（除了 FINAL 类型外），每个部分使用逗号隔开，每条规则可指定的行为包括：代理(Proxy)，不代理（DIRECT）和丢弃（REJECT）这三种。

六种过滤规则的配置如下：

- DOMAIN，精确指定域名，`` DOMAIN,www.apple.com,Proxy ``，匹配所有发往 www.apple.com 的流量
- DOMAIN-SUFFIX，按域名后缀匹配，``DOMAIN-SUFFIX,apple.com,Proxy``，匹配所有发往以 apple.com 结尾的流量，如 store.apple.com,mail.apple.com等，但不包括 issue-apple.com
- DOMAIN-KEWORD，按域名关键字匹配，``DOMAIN-KEYWORD,google,Proxy``，匹配所有域名中包含 google 的流量，如: www.google.com, issue-google.cn 等
- IP-CIDR，按 IP 地址无类匹配，`` IP-CIDR, 192.168.0.0/16,DIRECT``，匹配所有到内网 192.168.0.0/16 的流量
- GEOIP，按地理位置IP匹配，``GEOIP,US,DIRECT`` 匹配所有美国IP的流量
- FINAL，最终匹配的行为，这个必须放在最后，指定不能在前面任意一个规则匹配的流量行为

> 注：REJECT 过滤规则结合广告域名匹配可以过滤掉应用程序内部广告，这个真的很棒。

##### no-resolve

当有 Rule 中有 GEOIP 和 IP-CIDR 配置时，Surge 将对所有基于域名的请求流量进行域名解析代理，通过可选配置``no-resolve`` 可以指定特定流量不运行域名解析代理，如：

```
GEOIP,US,DIRECT,no-resolve
IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
```

##### force-remote-dns

基于 TCP 协议的应用程序在 TUN interface 模式中需要请求服务时，首先会发送域名解析请求，之后再发送 TCP 数据，如果域名无法在本地被解析，可以通过 ``force-remote-dns`` 配置域名在远程代理上进行解析，如：

```
DOMAIN,www.apple.com,Proxy,force-remote-dns
DOMAIN-SUFFIX,apple.com,Proxy,force-remote-dns
DOMAIN-KEYWORD,google,Proxy,force-remote-dns
```

> 注意: 这个仅对 TUN interface 有效，对于 Proxy Server 而言，所有的请求的 DNS 都是在远程代理上解析的。

#### 保护配置

所有以``#!REQUIRE-PROTECTED``开头的配置都是保护配置，保护配置在 Surge 软件中将不会被显示，在你将私人配置发送给其他人共享时可以将私有信息设为保护配置。

> 注：这其也没啥用，配置文件本身是可读的，下载文件直接看就是了。

### URL Scheme

Surge 支持 URL Scheme 满足重度/效率使用者的需求，不过按我估计，大概是作者不想被喷，于是就做了最基本的 URL Scheme，不然怎么会不支持切换配置文件的 URL Scheme，当前 Surge 仅支持三种行为以及一个选项：

- 行为:start,stop,toggle
- surge:///start，使用预先选定的服务器配置开启 Surge 服务
- surge:///stop，关闭 Surge 服务
- surge:///toggle，打开/关闭 Surge 服务
- 选项:autoclose=true，当行为执行完毕时，自动退出 Surge 软件 ``surge:///toggle?autoclose=true`` 



