
## 七、确保Web安全的HTTPS

在 HTTP 协议中有可能存在信息窃听或身份伪装等安全问题。使用
HTTPS 通信机制可以有效地防止这些问题。本章我们就了解一下
HTTPS。

### 1. HTTP的缺点

到现在为止，我们已了解到 HTTP 具有相当优秀和方便的一面，然而
HTTP 并非只有好的一面，事物皆具两面性，它也是有不足之处的。

HTTP 主要有这些不足，例举如下。

- 通信使用明文（不加密），内容可能会被窃听
- 不验证通信方的身份，因此有可能遭遇伪装
- 无法证明报文的完整性，所以有可能已遭篡改

这些问题不仅在 HTTP 上出现，其他未加密的协议中也会存在这类问
题。

除此之外，HTTP 本身还有很多缺点。而且，还有像某些特定的 Web
服务器和特定的 Web 浏览器在实际应用中存在的不足（也可以说成
是脆弱性或安全漏洞），另外，用 Java 和 PHP 等编程语言开发的
Web 应用也可能存在安全漏洞。

**通信使用明文可能会被窃听**

由于 HTTP 本身不具备加密的功能，所以也无法做到对通信整体（使
用 HTTP 协议通信的请求和响应的内容）进行加密。即，HTTP 报文
使用明文（指未经过加密的报文）方式发送。

- TCP/IP 是可能被窃听的网络  
如果要问为什么通信时不加密是一个缺点，这是因为，按
TCP/IP 协议族的工作机制，通信内容在所有的通信线路上都有
可能遭到窥视。

所谓互联网，是由能连通到全世界的网络组成的。无论世界哪个
角落的服务器在和客户端通信时，在此通信线路上的某些网络设
备、光缆、计算机等都不可能是个人的私有物，所以不排除某个
环节中会遭到恶意窥视行为。

即使已经过加密处理的通信，也会被窥视到通信内容，这点和未
加密的通信是相同的。只是说如果通信经过加密，就有可能让人
无法破解报文信息的含义，但加密处理后的报文信息本身还是会
被看到的。

![](./asserts/052.png)

图：互联网上的任何角落都存在通信内容被窃听的风险

窃听相同段上的通信并非难事。只需要收集在互联网上流动的数
据包（帧）就行了。对于收集来的数据包的解析工作，可交给那
些抓包（Packet Capture）或嗅探器（Sniffer）工具。

下面的图片示例就是被广泛使用的抓包工具 Wireshark。它可以
获取 HTTP 协议的请求和响应的内容，并对其进行解析。

像使用 GET 方法发送请求、响应返回了 200 OK，查看 HTTP 响
应报文的全部内容等一系列的事情都可以做到。

- 加密处理防止被窃听  
在目前大家正在研究的如何防止窃听保护信息的几种对策中，最
为普及的就是加密技术。加密的对象可以有这么几个。
  - 通信的加密  
    一种方式就是将通信加密。HTTP 协议中没有加密机制，但可以
    通过和 SSL（Secure Socket Layer，安全套接层）或
    TLS（Transport Layer Security，安全层传输协议）的组合使用，
    加密 HTTP 的通信内容。
    用 SSL 建立安全通信线路之后，就可以在这条线路上进行 HTTP
    通信了。与 SSL 组合使用的 HTTP 被称为 HTTPS（HTTP
    Secure，超文本传输安全协议）或 HTTP over SSL。
    ![](./asserts/053.png)
  - 内容的加密  
    还有一种将参与通信的内容本身加密的方式。由于 HTTP 协议中
    没有加密机制，那么就对 HTTP 协议传输的内容本身加密。即把
    HTTP 报文里所含的内容进行加密处理。
    在这种情况下，客户端需要对 HTTP 报文进行加密处理后再发送
    请求。
    ![](./asserts/054.png)
    诚然，为了做到有效的内容加密，前提是要求客户端和服务器同
    时具备加密和解密机制。主要应用在 Web 服务中。有一点必须
    引起注意，由于该方式不同于 SSL 或 TLS 将整个通信线路加密
    处理，所以内容仍有被篡改的风险。稍后我们会加以说明。

**不验证通信方的身份就可能遭遇伪装**

HTTP 协议中的请求和响应不会对通信方进行确认。也就是说存在“服
务器是否就是发送请求中 URI 真正指定的主机，返回的响应是否真的
返回到实际提出请求的客户端”等类似问题。

- 任何人都可发起请求  
  在 HTTP 协议通信时，由于不存在确认通信方的处理步骤，任何
  人都可以发起请求。另外，服务器只要接收到请求，不管对方是
  谁都会返回一个响应（但也仅限于发送端的 IP 地址和端口号没
  有被 Web 服务器设定限制访问的前提下）。
  HTTP 协议的实现本身非常简单，不论是谁发送过来的请求都会
  返回响应，因此不确认通信方，会存在以下各种隐患。
  - 无法确定请求发送至目标的 Web 服务器是否是按真实意
    图返回响应的那台服务器。有可能是已伪装的 Web 服务
    器。
  - 无法确定响应返回到的客户端是否是按真实意图接收响
    应的那个客户端。有可能是已伪装的客户端。
  - 无法确定正在通信的对方是否具备访问权限。因为某些
    Web 服务器上保存着重要的信息，只想发给特定用户通
    信的权限。
  - 无法判定请求是来自何方、出自谁手
  - 即使是无意义的请求也会照单全收。无法阻止海量请求
    下的 DoS 攻击（Denial of Service，拒绝服务攻击）

- 查明对手的证书  
虽然使用 HTTP 协议无法确定通信方，但如果使用 SSL 则可以。
SSL 不仅提供加密处理，而且还使用了一种被称为证书的手段，
可用于确定方。

证书由值得信任的第三方机构颁发，用以证明服务器和客户端是
实际存在的。另外，伪造证书从技术角度来说是异常困难的一件
事。所以只要能够确认通信方（服务器或客户端）持有的证书，
即可判断通信方的真实意图。

通过使用证书，以证明通信方就是意料中的服务器。这对使用者
个人来讲，也减少了个人信息泄露的危险性。

另外，客户端持有证书即可完成个人身份的确认，也可用于对
Web 网站的认证环节。

**无法证明报文完整性，可能已遭篡改**

所谓完整性是指信息的准确度。若无法证明其完整性，通常也就意味
着无法判断信息是否准确。

- 接收到的内容可能有误  
  由于 HTTP 协议无法证明通信的报文完整性，因此，在请求或响
  应送出之后直到对方接收之前的这段时间内，即使请求或响应的
  内容遭到篡改，也没有办法获悉。
  换句话说，没有任何办法确认，发出的请求 / 响应和接收到的请
  求 / 响应是前后相同的。
  ![](./asserts/055.png)
  比如，从某个 Web 网站上下载内容，是无法确定客户端下载的
  文件和服务器上存放的文件是否前后一致的。文件内容在传输途
  中可能已经被篡改为其他的内容。即使内容真的已改变，作为接
  收方的客户端也是觉察不到的。
  像这样，请求或响应在传输途中，遭攻击者拦截并篡改内容的攻
  击称为中间人攻击（Man-in-the-Middle attack，MITM）。
  ![](./asserts/056.png)

- 如何防止篡改  
  虽然有使用 HTTP 协议确定报文完整性的方法，但事实上并不便
  捷、可靠。其中常用的是 MD5 和 SHA-1 等散列值校验的方法，
  以及用来确认文件的数字签名方法。
  提供文件下载服务的 Web 网站也会提供相应的以 PGP（Pretty
  Good Privacy，完美隐私）创建的数字签名及 MD5 算法生成的散
  列值。PGP 是用来证明创建文件的数字签名，MD5 是由单向函
  数生成的散列值。不论使用哪一种方法，都需要操纵客户端的用
  户本人亲自检查验证下载的文件是否就是原来服务器上的文件。
  浏览器无法自动帮用户检查。
  可惜的是，用这些方法也依然无法百分百保证确认结果正确。因
  为 PGP 和 MD5 本身被改写的话，用户是没有办法意识到的。
  为了有效防止这些弊端，有必要使用 HTTPS。SSL 提供认证和加
  密处理及摘要功能。仅靠 HTTP 确保完整性是非常困难的，因此
  通过和其他协议组合使用来实现这个目标。下节我们介绍
  HTTPS 的相关内容

### 2. HTTP+ 加密 + 认证 + 完整性保护 = HTTPS

**HTTP加上加密处理和认证以及完整性保护后即是HTTPS**

如果在 HTTP 协议通信过程中使用未经加密的明文，比如在 Web 页
面中输入信用卡号，如果这条通信线路遭到窃听，那么信用卡号就暴
露了。

另外，对于 HTTP 来说，服务器也好，客户端也好，都是没有办法确
认通信方的。因为很有可能并不是和原本预想的通信方在实际通信。
并且还需要考虑到接收到的报文在通信途中已经遭到篡改这一可能
性。

为了统一解决上述这些问题，需要在 HTTP 上再加入加密处理和认证
等机制。我们把添加了加密及认证机制的 HTTP 称为 HTTPS（HTTP
Secure）。

![](./asserts/057.png)

经常会在 Web 的登录页面和购物结算界面等使用 HTTPS 通信。使用
HTTPS 通信时，不再用 http://，而是改用 https://。另外，当浏览器访
问 HTTPS 通信有效的 Web 网站时，浏览器的地址栏内会出现一个带
锁的标记。对 HTTPS 的显示方式会因浏览器的不同而有所改变。

**HTTPS是身披SSL外壳的 HTTP**

HTTPS并非是应用层的一种新协议。只是 HTTP 通信接口部分用
SSL（Secure Socket Layer）和 TLS（Transport Layer Security）协议代
替而已。

通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通
信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披
SSL 协议这层外壳的 HTTP。

![](./asserts/058.png)

在采用 SSL 后，HTTP 就拥有了 HTTPS 的加密、证书和完整性保护
这些功能。

SSL 是独立于 HTTP 的协议，所以不光是 HTTP 协议，其他运行在应
用层的 SMTP 和 Telnet 等协议均可配合 SSL 协议使用。可以说 SSL 是
当今世界上应用最为广泛的网络安全技术。

**相互交换密钥的公开密钥加密技术**

在对 SSL 进行讲解之前，我们先来了解一下加密方法。SSL 采用一种
叫做公开密钥加密（Public-key cryptography）的加密处理方式。

近代的加密方法中加密算法是公开的，而密钥却是保密的。通过这种
方式得以保持加密方法的安全性。

加密和解密都会用到密钥。没有密钥就无法对密码解密，反过来说，
任何人只要持有密钥就能解密了。如果密钥被攻击者获得，那加密也
就失去了意义。

- 共享密钥加密的困境  
  加密和解密同用一个密钥的方式称为共享密钥加密（Common key
  crypto system），也被叫做对称密钥加密。
  ![](./asserts/059.png)
  以共享密钥方式加密时必须将密钥也发给对方。可究竟怎样才能
  安全地转交？在互联网上转发密钥时，如果通信被监听那么密钥
  就可会落入攻击者之手，同时也就失去了加密的意义。另外还得
  设法安全地保管接收到的密钥。
  ![](./asserts/060.png)
- 使用两把密钥的公开密钥加密  
  公开密钥加密方式很好地解决了共享密钥加密的困难。
  公开密钥加密使用一对非对称的密钥。一把叫做私有密钥
  （private key），另一把叫做公开密钥（public key）。顾名思
  义，私有密钥不能让其他任何人知道，而公开密钥则可以随意发
  布，任何人都可以获得。
  使用公开密钥加密方式，发送密文的一方使用对方的公开密钥进
  行加密处理，对方收到被加密的信息后，再使用自己的私有密钥
  进行解密。利用这种方式，不需要发送用来解密的私有密钥，也
  不必担心密钥被攻击者窃听而盗走
  另外，要想根据密文和公开密钥，恢复到信息原文是异常困难
  的，因为解密过程就是在对离散对数进行求值，这并非轻而易举
  就能办到。退一步讲，如果能对一个非常大的整数做到快速地因
  式分解，那么密码破解还是存在希望的。但就目前的技术来看是
  不太现实的。
  ![](./asserts/061.png)
- HTTPS采用混合加密机制
  HTTPS 采用共享密钥加密和公开密钥加密两者并用的混合加密
  机制。若密钥能够实现安全交换，那么有可能会考虑仅使用公开
  密钥加密来通信。但是公开密钥加密与共享密钥加密相比，其处
  理速度要慢。
  所以应充分利用两者各自的优势，将多种方法组合起来用于通
  信。在交换密钥环节使用公开密钥加密方式，之后的建立通信交
  换报文阶段则使用共享密钥加密方式。
  ![](./asserts/062.png)

**证明公开密钥正确性的证书**

遗憾的是，公开密钥加密方式还是存在一些问题的。那就是无法证明
公开密钥本身就是货真价实的公开密钥。比如，正准备和某台服务器
建立公开密钥加密方式下的通信时，如何证明收到的公开密钥就是原
本预想的那台服务器发行的公开密钥。或许在公开密钥传输途中，真
正的公开密钥已经被攻击者替换掉了。

为了解决上述问题，可以使用由数字证书认证机构（CA，Certificate
Authority）和其相关机关颁发的公开密钥证书。

数字证书认证机构处于客户端与服务器双方都可信赖的第三方机构的
立场上。威瑞信（VeriSign）就是其中一家非常有名的数字证书认证
机构。我们来介绍一下数字证书认证机构的业务流程。首先，服务器
的运营人员向数字证书认证机构提出公开密钥的申请。数字证书认证
机构在判明提出申请者的身份之后，会对已申请的公开密钥做数字签
名，然后分配这个已签名的公开密钥，并将该公开密钥放入公钥证书
后绑定在一起。

服务器会将这份由数字证书认证机构颁发的公钥证书发送给客户端，
以进行公开密钥加密方式通信。公钥证书也可叫做数字证书或直接称
为证书。

接到证书的客户端可使用数字证书认证机构的公开密钥，对那张证书
上的数字签名进行验证，一旦验证通过，客户端便可明确两件事：
一，认证服务器的公开密钥的是真实有效的数字证书认证机构。二，
服务器的公开密钥是值得信赖的。

此处认证机关的公开密钥必须安全地转交给客户端。使用通信方式
时，如何安全转交是一件很困难的事，因此，多数浏览器开发商发布
版本时，会事先在内部植入常用认证机关的公开密钥。

![](./asserts/063.png)

**可证明组织真实性的 EV SSL 证书**

证书的一个作用是用来证明作为通信一方的服务器是否规范，另
外一个作用是可确认对方服务器背后运营的企业是否真实存在。
拥有该特性的证书就是 EV SSL 证书（Extended Validation SSL
Certificate）

EV SSL 证书是基于国际标准的认证指导方针颁发的证书。其严
格规定了对运营组织是否真实的确认方针，因此，通过认证的
Web 网站能够获得更高的认可度。

持有 EV SSL 证书的 Web 网站的浏览器地址栏处的背景色是绿色
的，从视觉上就能一眼辨别出。而且在地址栏的左侧显示了 SSL
证书中记录的组织名称以及颁发证书的认证机构的名称。

上述机制的原意图是为了防止用户被钓鱼攻击（Phishing），但
就效果上来讲，还得打一个问号。很多用户可能不了解 EV SSL
证书相关的知识，因此也不太会留意它。

**用以确认客户端的客户端证书**

HTTPS 中还可以使用客户端证书。以客户端证书进行客户端认
证，证明服务器正在通信的对方始终是预料之内的客户端，其作
用跟服务器证书如出一辙。

但客户端证书仍存在几处问题点。其中的一个问题点是证书的获
取及发布。

想获取证书时，用户得自行安装客户端证书。但由于客户端证书
是要付费购买的，且每张证书对应到每位用户也就意味着需支付
和用户数对等的费用。另外，要让知识层次不同的用户们自行安
装证书，这件事本身也充满了各种挑战。

现状是，安全性极高的认证机构可颁发客户端证书但仅用于特殊
用途的业务。比如那些可支撑客户端证书支出费用的业务。

例如，银行的网上银行就采用了客户端证书。在登录网银时不仅
要求用户确认输入 ID 和密码，还会要求用户的客户端证书，以
确认用户是否从特定的终端访问网银。

客户端证书存在的另一个问题点是，客户端证书毕竟只能用来证
明客户端实际存在，而不能用来证明用户本人的真实有效性。也
就是说，只要获得了安装有客户端证书的计算机的使用权限，也
就意味着同时拥有了客户端证书的使用权限。

**认证机构信誉第一**

SSL 机制中介入认证机构之所以可行，是因为建立在其信用绝对
可靠这一大前提下的。然而，2011 年 7 月，荷兰的一家名叫
DigiNotar 的认证机构曾遭黑客不法入侵，颁布了 google.com 和
twitter.com 等网站的伪造证书事件。这一事件从根本上撼动了
SSL 的可信度。

因为伪造证书上有正规认证机构的数字签名，所以浏览器会判定
该证书是正当的。当伪造的证书被用做服务器伪装之时，用户根
本无法察觉到。

虽然存在可将证书无效化的证书吊销列表（Certificate Revocation
List，CRL）机制，以及从客户端删除根证书颁发机构（Root
Certificate Authority，RCA）的对策，但是距离生效还需要一段
时间，而在这段时间内，到底会有多少用户的利益蒙受损失就不
得而知了

**由自认证机构颁发的证书称为自签名证书**

如果使用 OpenSSL 这套开源程序，每个人都可以构建一套属于
自己的认证机构，从而自己给自己颁发服务器证书。但该服务器
证书在互联网上不可作为证书使用，似乎没什么帮助。

独立构建的认证机构叫做自认证机构，由自认证机构颁发的“无
用”证书也被戏称为自签名证书。

浏览器访问该服务器时，会显示“无法确认连接安全性”或“该网
站的安全证书存在问题”等警告消息。

由自认证机构颁发的服务器证书之所以不起作用，是因为它无法
消除伪装的可能性。自认证机构能够产生的作用顶多也就是自己
对外宣称“我是○○”的这种程度。即使采用自签名证书，通过 SSL
加密之后，可能偶尔还会看见通信处在安全状态的提示，可那也
是有问题的。因为 就算加密通信，也不能排除正在和已经过伪
装的假服务器保持通信。

值得信赖的第三方机构介入认证，才能让已植入在浏览器内的认
证机构颁布的公开密钥发挥作用，并借此证明服务器的真实性。

中级认证机构的证书可能会变成自认证证书

多数浏览器内预先已植入备受信赖的认证机构的证书，但也有一
小部分浏览器会植入中级认证机构的证书。

对于中级认证机构颁发的服务器证书，某些浏览器会以正规的证
书来对待，可有的浏览器会当作自签名证书

**HTTPS 的安全通信机制**

为了更好地理解 HTTPS，我们来观察一下 HTTPS 的通信步骤。

![](./asserts/064.png)

步骤 1： 客户端通过发送 Client Hello 报文开始 SSL 通信。报文中包
含客户端支持的 SSL 的指定版本、加密组件（Cipher Suite）列表（所
使用的加密算法及密钥长度等）。

步骤 2： 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应
答。和客户端一样，在报文中包含 SSL 版本以及加密组件。服务器的
加密组件内容是从接收到的客户端加密组件内筛选出来的。

步骤 3： 之后服务器发送 Certificate 报文。报文中包含公开密钥证
书。

步骤 4： 最后服务器发送 Server Hello Done 报文通知客户端，最初阶
段的 SSL 握手协商部分结束。

步骤 5： SSL 第一次握手结束之后，客户端以 Client Key Exchange 报
文作为回应。报文中包含通信加密中使用的一种被称为 Pre-master
secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密。

步骤 6： 接着客户端继续发送 Change Cipher Spec 报文。该报文会提
示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密。

步骤 7： 客户端发送 Finished 报文。该报文包含连接至今全部报文的
整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确
解密该报文作为判定标准。

步骤 8： 服务器同样发送 Change Cipher Spec 报文。

步骤 9： 服务器同样发送 Finished 报文。

步骤 10： 服务器和客户端的 Finished 报文交换完毕之后，SSL 连接
就算建立完成。当然，通信会受到 SSL 的保护。从此处开始进行应用
层协议的通信，即发送 HTTP 请求。

步骤 11： 应用层协议通信，即发送 HTTP 响应。

步骤 12： 最后由客户端断开连接。断开连接时，发送 close_notify 报
文。上图做了一些省略，这步之后再发送 TCP FIN 报文来关闭与 TCP
的通信。

在以上流程中，应用层发送数据时会附加一种叫做 MAC（Message
Authentication Code）的报文摘要。MAC 能够查知报文是否遭到篡
改，从而保护报文的完整性。

下面是对整个流程的图解。图中说明了从仅使用服务器端的公开密钥
证书（服务器证书）建立 HTTPS 通信的整个过程。

![](./asserts/065.png)

- SSL和TLS  
  SSL 技术最初是由浏览器开发商网景通信公司率先倡导的，开发
  过 SSL3.0 之前的版本。目前主导权已转移到 IETF（Internet
  Engineering Task Force，Internet 工程任务组）的手中。  
  IETF 以 SSL3.0 为基准，后又制定了 TLS1.0、TLS1.1 和
  TLS1.2。TSL 是以 SSL 为原型开发的协议，有时会统一称该协议
  为 SSL。当前主流的版本是 SSL3.0 和 TLS1.0。  
  由于 SSL1.0 协议在设计之初被发现出了问题，就没有实际投入
  使用。SSL2.0 也被发现存在问题，所以很多浏览器直接废除了
  该协议版本
- SSL速度慢吗  
  HTTPS 也存在一些问题，那就是当使用 SSL 时，它的处理速度
  会变慢。
  ![](./asserts/066.png)
  SSL 的慢分两种。一种是指通信慢。另一种是指由于大量消耗
  CPU 及内存等资源，导致处理速度变慢。  
  和使用 HTTP 相比，网络负载可能会变慢 2 到 100 倍。除去和
  TCP 连接、发送 HTTP 请求 • 响应以外，还必须进行 SSL 通信，
  因此整体上处理通信量不可避免会增加。  
  另一点是 SSL 必须进行加密处理。在服务器和客户端都需要进行
  加密和解密的运算处理。因此从结果上讲，比起 HTTP 会更多地
  消耗服务器和客户端的硬件资源，导致负载增强。  
  针对速度变慢这一问题，并没有根本性的解决方案，我们会使用
  SSL 加速器这种（专用服务器）硬件来改善该问题。该硬件为
  SSL 通信专用硬件，相对软件来讲，能够提高数倍 SSL 的计算速
  度。仅在 SSL 处理时发挥 SSL 加速器的功效，以分担负载。
  > 为什么不一直使用 HTTPS  
    既然 HTTPS 那么安全可靠，那为何所有的 Web 网站不一直使用
    HTTPS ？  
    其中一个原因是，因为与纯文本通信相比，加密通信会消耗更多的
    CPU 及内存资源。如果每次通信都加密，会消耗相当多的资源，平
    摊到一台计算机上时，能够处理的请求数量必定也会随之减少。  
    因此，如果是非敏感信息则使用 HTTP 通信，只有在包含个人信息
    等敏感数据时，才利用 HTTPS 加密通信。  
    特别是每当那些访问量较多的 Web 网站在进行加密处理时，它们
    所承担着的负载不容小觑。在进行加密处理时，并非对所有内容都
    进行加密处理，而是仅在那些需要信息隐藏时才会加密，以节约资
    源。
    除此之外，想要节约购买证书的开销也是原因之一。  
    要进行 HTTPS 通信，证书是必不可少的。而使用的证书必须向认
    证机构（CA）购买。证书价格可能会根据不同的认证机构略有不
    同。通常，一年的授权需要数万日元（现在一万日元大约折合 600
    人民币）  
    那些购买证书并不合算的服务以及一些个人网站，可能只会选择采
    用 HTTP 的通信方式。