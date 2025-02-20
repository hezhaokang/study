#   17、http协议和apache服务

## 静动态资源

动静资源的区别

```
	客户端通过 URI 锚定服务端的一个唯一资源，WEB SERVER 服务将该资源返回给客户端，资源可以分为静态资源和动态资源两类，在 Web 服务中，静态资源和动态资源是根据其内容和 "生成方式" 来定义的，它们在处理和提供方式上有一些显著的区别。
 	在实际的 Web 应用中，通常会同时使用静态资源和动态资源，以充分利用它们各自的优势，例如，静态资源可以用于提供页面的基本框架、样式和脚本，而动态资源可以用于提供个性化的、与用户交互的内容。
```

静态资源 - 是啥就是啥

```
- 定义：静态资源是指在服务器上事先存在，不需要在请求时进行动态生成的文件，这些文件包括 HTML、CSS、JavaScript、图像（如 JPEG、PNG）等
- 生成方式：静态资源是直接从服务器上的文件系统提供的，没有经过服务器端的处理或计算，它们的内容在创建时已经确定，不会根据每个请求而变化
- 特点：静态资源适用于那些内容较为固定，不经常变化的情况，它们可以被直接缓存，以提高访问速度，并减轻服务器的负载
静态资源是指那些内容在服务器上存储时就已经固定的资源，客户端请求时直接返回，没有任何变化。它们在服务器端通常不需要进行任何处理或动态计算。
特点：
内容固定，不会根据用户请求而改变。
请求时，服务器直接返回文件内容。
一般包括图片（.jpg, .png），CSS样式表，JavaScript文件，字体文件等。
响应速度较快，因为文件已经存在，不需要动态生成。
```

动态资源

```
- 定义：动态资源是在服务器上根据用户请求动态生成的内容。这通常涉及到使用服务器端脚本（如 PHP、Python、Node.js）进行处理，并根据用户请求的参数生成不同的内容
- 生成方式：动态资源的内容在请求时动态生成，可能涉及到数据库查询、计算或其他与用户请求相关的操作，每次请求可能产生不同的结果
- 特点：动态资源适用于需要根据用户输入或其他动态条件生成内容的情况。它们通常涉及更多的计算和服务器端的处理，可能会引起更高的服务器负载
动态资源是根据用户的请求或某些条件，实时生成或处理后再返回的内容。服务器通常会根据请求内容，调用数据库或进行计算，生成一个响应并返回给客户端。
特点：
内容是动态生成的，根据请求内容或时间等因素变化。
需要在服务器端进行处理，通常会涉及到脚本语言（如PHP, Python, Node.js等）或者应用程序。
响应速度较慢，因为需要进行计算、查询数据库等操作。
```

## http的工作机制

```
1 建立连接
 	客户端（通常是浏览器）通过TCP/IP协议与服务器建立连接。这是HTTP通信的基础，确保数据能够在客户端和服务器之间可靠地传输。
2. 发送请求
 	一旦连接建立，客户端会向服务器发送一个HTTP请求。这个请求由多个部分组成，包括：
 		请求行：包含
 			请求方法（如GET、POST等）、
 			请求URI（统一资源标识符，指定了要访问的资源的位置）
 			HTTP版本号（表示请求所使用的HTTP协议版本）等。
 		头部字段：使用key-value形式传递一些请求信息，如
 			Accept（客户端可以接受的响应内容类型）、
 			User-Agent（客户端的浏览器信息）、
 			Referer（客户端从哪个页面跳转而来）等。
 		正文：可选部分，用于向服务器传递一些数据。例如，
 			当客户端向服务器提交表单时，表单数据就可以放在请求的正文中。
3. 处理请求
 	服务器接收到客户端的请求后，会解析请求行和头部字段，查找所请求的资源，并准备相应的响应。
4. 发送响应
 	服务器将处理后的响应发送回客户端。响应也由多个部分组成，包括：
 		状态行：包含HTTP版本号、状态码和状态短语。
 			状态码是一个三位数字，用于表示服务器对请求的处理结果
 			（如200表示成功，404表示未找到资源，500表示服务器内部错误等）。
 			状态短语是对状态码的简短描述。
		 头部字段：使用key-value形式传递一些响应信息，如
 			Content-Type（响应内容的类型）、
 			Content-Length（响应内容的长度）、
 			Set-Cookie（服务器要求客户端保存一个Cookie等）。
 		正文：响应的实际内容。
 			例如，当客户端请求一个网页时，网页的HTML代码就可以放在响应的正文中。
5. 关闭连接
 	连接在请求和响应之后通常会被关闭，以释放系统资源。然而，在HTTP/1.1中引入了持久连接（也称为连接重用），允许在同一个TCP连接上发送和接收多个HTTP请求和响应，从而提高了网络传输的效率。
```

```
当你在 Web 浏览器中输入网址并访问一个网站时，浏览器会通过一系列步骤发起 HTTP 请求 来获取网页。以下是浏览器发起 HTTP 请求访问网站的过程：
1. URL 解析
	用户在浏览器中输入一个 URL（Uniform Resource Locator）地址，例如 https://www.example.com/index.html。
	浏览器首先解析该 URL，获取各个组成部分：协议（如 HTTP 或 HTTPS）
		主机名（www.example.com）
		路径（/index.html）
		端口（如果没有指定，HTTP 默认使用 80 端口，HTTPS 使用 443 端口）
		如果 URL 使用 HTTPS 协议，浏览器会启动一个 TLS/SSL 握手过程，以确保安全连接。

2. DNS 解析（域名解析）
	浏览器需要将域名（如 www.example.com）转换为 IP 地址，因为计算机通过 IP 地址与服务器进行通信。
		浏览器向操作系统发出请求，操作系统会检查本地的 DNS 缓存，如果没有缓存结果，操作系统会向 DNS 服务器 查询该域名对应的 IP 地址。
		DNS 服务器返回相应的 IP 地址，浏览器得到服务器的 IP 地址后，可以与服务器建立连接。

3. 建立 TCP 连接
	浏览器使用 TCP（三次握手） 协议与目标服务器建立连接。这个过程包括：客户端（浏览器）发送 SYN：浏览器向服务器发送一个同步请求（SYN）。
		服务器发送 SYN-ACK：服务器响应客户端的同步请求，发送一个确认消息（SYN-ACK）。
		客户端发送 ACK：浏览器再次确认，三次握手完成，TCP 连接建立。
		这时，浏览器与目标服务器之间已经建立了 TCP 连接，准备进行数据交换。

4. 浏览器发送 HTTP 请求
	一旦 TCP 连接建立，浏览器会根据用户输入的 URL 生成 HTTP 请求，并将其发送到服务器。典型的 HTTP 请求包括以下部分：请求行（Request Line）：包括 HTTP 方法（如 GET、POST）、请求路径和 HTTP 版本。例如：
GET /index.html HTTP/1.1
		请求头部（Request Headers）：包含关于客户端环境、请求类型等信息。例如：
		Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
请求体（Request Body）：对于 POST 或 PUT 请求，通常包含请求体，用于发送数据；但对于 GET 请求，通常没有请求体。

5. 服务器处理请求
服务器接收到浏览器发送的 HTTP 请求后，解析请求并进行相应的处理。例如，服务器可能会读取指定的文件、执行数据库查询或调用应用程序，生成动态网页。
服务器处理完请求后，会构建一个 HTTP 响应，将处理结果返回给浏览器。HTTP 响应通常包含：响应行（Response Line）：包括 HTTP 版本、状态码和状态消息。例如：
	HTTP/1.1 200 OK
	响应头部（Response Headers）：包含关于响应的各种信息，如内容类型、长度、缓存控制等。例如：css
		Content-Type: text/html; charset=UTF-8
		Content-Length: 1234
	响应体（Response Body）：包含实际的网页内容（HTML、CSS、JavaScript 文件等）。

6. 浏览器接收并解析 HTTP 响应
	浏览器接收到 HTTP 响应后，会根据响应头中的 Content-Type 来确定如何处理响应体。例如，如果响应是 HTML，浏览器会将其解析为网页内容并呈现给用户。
	如果响应包含其他资源（如 CSS、JavaScript 文件、图片等），浏览器会继续发起相应的 HTTP 请求来获取这些资源。
```

```
HTTP1.0 定义了三种请求方法： GET，POST，HEAD
HTTP1.1 新增了六种请求方法：OPTIONS，PUT，PATCH，DELETE，TRACE，CONNECT
在后续其它版本的 HTTP 协议中，还陆续增加了 MKCOL，COPY，MOVE，LOCK，UNLOCK 等方法
```

```
常用的http的请求资源的方法：
    GET  - 获取资源
    POST  -- 发送完整数据
    PUT   -- 请求修改部分数据
    DELETE -- 请求删除一个数据
    HEAD  -- 只要 请求头|响应头信息
```

| 状态码 | 状态短语            | 说明                                                         |
| ------ | ------------------- | ------------------------------------------------------------ |
| 200    | OK                  | 请求成功，一般用于GET与POST请求                              |
| 301    | Moved Permanently   | 永久重定向，客户端请求的资源被移动到新的URI，返回信息中包含新的URI，客户端自动请求 |
| 302    | Found               | 临时重定向，其它与301相同                                    |
| 403    | Forbidden           | 服务器理解请求客户端的请求，但是拒绝执行此请求，原因有多样，通常是没有权限 |
| 404    | Not Found           | 服务器上没有找到客户端需要访问的资源                         |
| 502    | Bad Gateway         | 服务端网关错误                                               |
| 503    | Service Unavailable | 服务器当前很忙，暂时无法响应请求（网络服务正忙，请稍后重试） |

## http特点

```
无连接
限制每次连接只处理一个请求。
 	服务器处理完客户的请求，并收到客户的应答后，即断开连接。
 	这种方式可以节省传输时间。

无状态
HTTP协议是无状态协议，对于事务处理没有记忆能力。
```



## HTTP服务通信过程

![image-20241201123302014](D:\桌面\mage.md\笔记\5day-png\17HTTP通信过程.png)

## 串行和并行连接

![image-20241201123334417](D:\桌面\mage.md\笔记\5day-png\17串行并行.png)

## 串行，持久连接和管道

![image-20241201123421204](D:\桌面\mage.md\笔记\5day-png\17串行，持久连接和管道.png)

## **Cookie和Session**

```
cookie
	一种在客户端保存状态信息的机制。服务器可以通过Set-Cookie头部向客户端发送一个Cookie，客户端在下一次请求时将该Cookie发送回服务器。
	服务器可以根据Cookie的内容来识别客户端的身份，从而实现状态管理。
	
session
	一种在服务器保存状态信息的机制。
 	服务器在接收到客户端的请求时，为该客户端创建一个Session对象，并将该对象的ID保存在一个Cookie中发送给客户端。
 	客户端在下一次请求时将该Cookie发送回服务器，服务器根据Cookie中的Session ID来查找该客户端对应的Session对象，从而实现状态管理。
```

## IO模型

![image-20241201134141960](D:\桌面\mage.md\笔记\5day-png\17服务端网络io处理.png)

```
1. 客户端发送请求
2. 将请求数据从网卡缓冲区拷贝到内核缓冲区，这一步由硬件通过DMA直接完成
3. 将请求数据从内核缓冲区拷贝到用户空间的WEB服务器缓冲空间，这一步由操作系统完成
4. 用户空间WEB服务进程处理请求，构建响应数据
5. 将响应数据从WEB服务器缓冲区拷贝到内核缓冲区，这一步由操作系统完成
6. 将响应数据从内核缓冲区拷贝到网卡缓冲区，这一步由DMA直接完成
7. 返回响应数据给客户端
```

```
内核缓冲区和应用进程缓存区都位于内存中，但分别属于内核空间和用户空间。网卡缓冲区则位于网卡硬件内部，不属于内存。
这些缓冲区在数据传输过程中起着重要的作用，它们之间的数据复制和传输是由操作系统和硬件共同协作完成的。
```

磁盘IO和网络IO，每次的IO过程，都有两个步骤：

```
1. 将数据从文件先加载至内核内存空间（缓冲区），等待数据准备完成，时间较长
2. 将数据从内核缓冲区复制到用户空间的进程的内存中，时间较短
```

### **消息通信**

```
同步和异步关注的是消息的通信机制，即调用者在等待一件事情的处理结果时，被调用者是否提供完成状态的通知
- 同步：sychronous，
 被调用者并不提供事件的处理结果相关的通知消息，需要调用者主动询问事件是否处理完成
- 异步：asynchronous，
 被调用者通过状态，通知或者回调机制主动通知发起调用者相关的运行状态
```

阻塞和非阻塞 - 发起者在等待结果时的状态

```
阻塞和非阻塞关注的是调用发起者在等待结果返回之前所处的状态
- 阻塞：blocking，
 指IO操作需要彻底完成后才返回到用户空间，调用结果返回之前，调用者被挂起，干不了别的事情。
 示例：心脑血管阻塞的时候，你只能在手术台上等着他们做手术。
- 非阻塞：nonblocking，
 指IO操作被调用后立即返回给用户一个状态值，而无需等到IO操作彻底完成，在最终的调用结果返回之前，调用者不会被挂起，可以去做别的事情。
 示例：关将军刮骨疗毒，同时下棋，等待手术完成后，知道了即可。
```

### **五种IO**

#### **阻塞IO**

![image-20241201135057476](D:\桌面\mage.md\笔记\5day-png\17阻塞IO.png)

```
阻塞IO模型是最简单的I/O模型，用户线程在内核进行IO操作时被阻塞

用户线程通过系统调用 read 发起I/O读操作，由用户空间转到内核空间。内核等到数据包到达后，然后将接收的数据拷贝到用户空间，完成 read 操作，用户需要等待 read 将数据读取到buffer后，才继续处理接收的数据。整个I/O请求的过程中，用户线程是被阻塞的，这导致用户在发起IO请求时，不能做任何事情，对CPU的资源利用率不够。
```

#### 非阻塞IO

![image-20241201135238286](D:\桌面\mage.md\笔记\5day-png\17非阻塞IO.png)

```
用户线程发起IO请求时立即返回，但并未读取到任何数据，用户线程需要不断地发起IO请求，直到数据
到达后，才真正读取到数据，继续执行。
```

轮询机制的两个问题

```
	1 如果有大量文件描述符都要等，那么就得一个一个的 read。这会带来大量的 Context Switch（read是系统调用，每调用一次就得在用户态和核心态切换一次）。
	2 轮询的时间不好把握，这里是要猜多久之后数据才能到。等待时间设的太长，程序响应延迟就过大；设的太短，就会造成过于频繁的重试，干耗CPU而已，是比较浪费CPU的方式，一般很少直接使用这种模型，而是在其他IO模型中使用非阻塞IO这一特性。
```

#### 多路复用IO

![image-20241201135423634](D:\桌面\mage.md\笔记\5day-png\17多路复用IO.png)

```
多路复用IO指一个线程可以同时（实际是交替实现，即并发完成）监控和处理多个文件描述符对应各自的IO，即复用同一个线程。
```

```
	一个线程之所以能实现同时处理多个IO，是因为这个线程调用了内核中的SELECT，POLL 或 EPOLL 等系统调用，从而实现多路复用IO，这三个系统调用的好处就在于单个 process 可以同时处理多个网络连接IO，其基本原理是不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。
	然后用户进程再调用 read 操作，将数据从 kernel 拷贝到用户进程，然后进行后续的动作。
```

```
	当用户进程调用了select，那么整个进程会被 block，而同时，kernel会 监视 所有 select 负责的 socket，当任何一个 socket 中的数据准备好了，select 就会返回。这个时候用户进程再调用 read 操作，将数据从 kernel 拷贝到用户进程
	Apache prefork是此模式的 select，worker 是 poll 模式。
```

#### 信号驱动IO

![image-20241201135651276](D:\桌面\mage.md\笔记\5day-png\17信号驱动IO.png)

```
信号驱动I/O的意思就是进程现在不用傻等着，也不用去轮询。而是让内核在数据就绪时，发送信号通知进程

	调用的步骤是，通过系统调用 sigaction ，并注册一个信号处理的回调函数，该调用会立即返回，然后主程序可以继续向下执行，
 	当有IO操作准备就绪,即内核数据就绪时，内核会为该进程产生一个 SIGIO 信号，并回调注册的信号回调函数，这样就可以在信号回调函数中系统调用 recvfrom 获取数据，将用户进程所需要的数据从内核空间拷贝到用户空间，然后进行后续的动作。
```

```
优点：
 	线程并没有在等待数据时被阻塞，内核直接返回调用接收信号，不影响进程继续处理其他请求，因此可以提高资源的利用率。
缺点：
 	信号 IO 在大量 IO 操作时可能会因为信号队列溢出导致没法通知。
```

#### 异步IO

![image-20241201135900482](D:\桌面\mage.md\笔记\5day-png\17异步IO.png)

```
异步IO 与 信号驱动IO最大区别在于，信号驱动是内核通知用户进程何时开始一个IO操作，而异步IO是由内核通知用户进程IO操作何时完成，两者有本质区别。
```

```
	相对于同步IO，异步IO不是顺序执行。用户进程进行 aio_read 系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。
 	等到 socket 数据准备好了，内核 "直接" 复制数据给进程，然后从内核向进程发送通知。
 	IO 两个阶段，进程都是非阻塞的
	信号驱动IO当内核通知触发信号处理程序时，信号处理程序还需要阻塞在从内核空间缓冲区拷贝数据到用户空间缓冲区这个阶段，而异步IO直接是在第二个阶段完成后，内核直接通知用户线程可以进行后续操作了。
```

```
优点：
 	异步 IO 能够充分利用 DMA 特性，让 IO 操作与计算重叠
缺点：
 	要实现真正的异步 IO，操作系统需要做大量的工作，目前 Windows 下通过 IOCP 实现了真正的异步
IO，在 Linux 系统下，Linux 2.6才引入，目前 AIO 并不完善，因此在 Linux 下实现高并发网络编程时以 IO 复用模型模式 + 多线程任务的架构基本可以满足需求。
```

#### 对比

![image-20241201140105067](D:\桌面\mage.md\笔记\5day-png\17IO对比.png)

## apache2软件特性功能

```
特性
- 高度模块化：core + modules
- DSO：Dynamic Shared Object 动态加载/卸载
- MPM：multi-processing module 多路处理模块
```

```
功能
- 虚拟主机：IP，Port，FQDN
- CGl：Common Gateway lnterface，通用网关接口
- 反向代理、负载均衡、路径别名
- 丰富的用户认证机制：basic，digest
- 支持第三方模块
```

## 工作模式

### prefork模型

```
	预派生模式，有一个主控制进程，然后生成多个子进程，每个子进程有一个独立的线程响应用户请求，相对比较占用内存，但是比较稳定，可以设置最大和最小进程数，是最古老的一种模式，也是最稳定的模式，适用于访问量不是很大的场景。
```

特点

```
优点：工作稳定
缺点：每个用户请求需要对应开启一个进程，占用资源较多，并发性差，不适用于高并发场景
```

![image-20241201142901675](D:\桌面\mage.md\笔记\5day-png\17prefork模型.png)

### worker模型

```
一种多进程和多线程混合的模型，有一个控制进程，启动多个子进程，每个子进程里面包含固定的线程，使用线程来处理请求，当线程不够使用的时候会再启动一个新的子进程，然后在进程里面再启动线程处理请求，由于其使用了线程处理请求，因此可以承受更高的并发。
```

特点

```
优点：相比prefor模型，其占用的内存较少，可以同时处理更多的请求
缺点：使用keepalive的长连接方式，某个线程会一直被占据，即使没有传输数据，也需要一直等待到超时才会被释放。如果过多的线程被这样占据，也会导致在高并发场景下的无服务线程可用。（该问题在prefork模式下，同样会发生）
```

![image-20241201143048829](D:\桌面\mage.md\笔记\5day-png\17worker模型.png)

### event模型

```
event 模型，属于事件驱动模型(epoll)，每个进程响应多个请求，在现在版本里的已经是稳定可用的模式，它和worker模式很像。最大的区别在于，它解决了keepalive场景下，长期被占用的线程的资源浪费问题（某些线程因为被keepalive，空挂在哪里等待，中间几乎没有请求过来，甚至等到超时）。event MPM中，会有一个专门的线程来管理这些keepalive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放。这样增强了高并发场景下的请求处理能力。
```

特点

```
优点：单线程响应多请求，占据更少的内存，高并发下表现更优秀，会有一个专门的线程来管理keepalive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放
缺点：没有线程安全控制
```

![image-20241201143424989](D:\桌面\mage.md\笔记\5day-png\17envent模型.png)

## httpd&apache2

```
安装软件
rocky
yum -y install httpd
ubuntu
apt -y install apache2
```

**配置文件**

```shell
rocky
/etc/httpd/conf/
root@rocky9-14:httpd # ll /etc/httpd/
total 4
drwxr-xr-x. 2 root root   37 Dec  1 16:15 conf					#主配置文件
drwxr-xr-x. 2 root root   98 Dec  1 16:53 conf.d				#子配置文件目录
drwxr-xr-x. 2 root root 4096 Dec  1 16:15 conf.modules.d		#可用的模块配置文件
lrwxrwxrwx. 1 root root   19 Nov  4 01:31 logs -> ../../var/log/httpd
lrwxrwxrwx. 1 root root   29 Nov  4 01:31 modules -> ../../usr/lib64/httpd/moduleslrwxrwxrwx. 1 root root   10 Nov  4 01:31 run -> /run/httpd
lrwxrwxrwx. 1 root root   19 Nov  4 01:31 state -> ../../var/lib/httpd
root@rocky9-14:httpd # ll /etc/httpd/conf
total 28
-rw-r--r--. 1 root root 12005 Nov  4 01:28 httpd.conf
-rw-r--r--. 1 root root 13430 Nov  4 01:30 magic
```

```shell
ubuntu
/etc/apache2/
root@ubuntu42:~# ll /etc/apache2/
-rw-r--r--   1 root root  ... apache2.conf 		#主配置文件
drwxr-xr-x   2 root root  ... conf-available/ 	#子配置文件目录
drwxr-xr-x   2 root root  ... conf-enabled/ 	#生效的子配置文件，链接到 conf_available中
-rw-r--r--   1 root root  ... envvars 			#全局环境变量配置文件
-rw-r--r--   1 root root  ... magic 			#配合 mod_mime_magic模块判断 MIME 类型的配置文件
drwxr-xr-x   2 root root  ... mods-available/ 	#可用的模块配置文件
drwxr-xr-x   2 root root  ... mods-enabled/ 	#生效的模块配置文件，链接到 mods_available中
-rw-r--r--   1 root root  ... ports.conf 		#默认端口配置文件
drwxr-xr-x   2 root root  ... sites-available/ 	#可用的虚拟主机配置文件
drwxr-xr-x   2 root root  ... sites-enabled/ 	#生效的虚似主机配置文件，链接到 sites_available中

主配置文件中的包含
root@ubuntu42:~# cat /etc/apache2/apache2.conf | grep "^Include"
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
Include ports.conf
IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf

全局配置
root@ubuntu42:~# ll -L /etc/apache2/conf-enabled/ 
-rw-r--r-- 1 root root  315 ... charset.conf 					#默认编码，全都注释
-rw-r--r-- 1 root root 3224 ... localized-error-pages.conf 		#自定义错误页面，全
注释
-rw-r--r-- 1 root root  189 ... other-vhosts-access-log.conf 	#未定义的虚拟主机访
问日志 
-rw-r--r-- 1 root root 2174 ... security.conf 					#安全设置配置文件
-rw-r--r-- 1 root root  455 ... serve-cgi-bin.conf 				#CGI 配置
```

配置语法结构的检测

```shell
apachectl -t
```

目录配置段

```
<Directory "/path/to/dir"> ... </Directory>
 - 用于指定服务器上一个具体目录的各种访问控制
<DirectoryMatch "^/path/to/dir/.*$"> ... </DirectoryMatch>
 - 使用正则表达式来匹配目录路径。这允许你对符合特定模式的多个目录应用相同的配置。
```

文件配置段

```
<Files "config.file"> ... </Files>
 - 用于指定服务器上特定文件的配置。你可以在这个容器内设置访问控制、MIME类型等指令。
<FilesMatch "\.(php|html)$"> ... </FilesMatch>
 - 使用正则表达式来匹配文件名。这允许你对符合特定模式的多个文件应用相同的配置。
```

URL配置段

```
<Location "/admin"> ... </Location>
 - 用于指定基于URL路径的配置，这对于处理别名、重定向或基于URL的访问控制非常有用。
<LocationMatch "^/secure/.*$"> ... </LocationMatch>
 - 使用正则表达式来匹配URL路径。这允许你对符合特定模式的多个URL路径应用相同的配置。
```

![image-20241201171057530](D:\桌面\mage.md\笔记\5day-png\17常见配置段.png)

## 压缩实践

```shell
root@ubuntu-42:source # cat /etc/apache2/mods-enabled/deflate.conf
<Ifmodule mod_deflate.c>
        <IfModule mod_filter.c>
                AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css text/javascript
                AddOutputFilterByType DEFLATE application/x-javascript application/javascript application/ecmascript
                AddOutputFilterByType DEFLATE application/rss+xml
                AddOutputFilterByType DEFLATE application/wasm
                AddOutputFilterByType DEFLATE application/xml
                AddOutputFilterByType DEFLATE image/png
                DeflateCompressionLevel 9
        </IfModule>
</IfModule>

root@ubuntu-42:source # systemctl restart apache2.service
```

## 软件信息提醒

```shell
root@ubuntu-42:source # vim /etc/apache2/conf-available/security.conf
修改下面行：
ServerTokens Prod

root@ubuntu-42:source # systemctl restart apache2.service
```

## 持久连接

```shell
root@ubuntu-42:source # grep "KeepAlive" /etc/apache2/apache2.conf | grep -v "#"
KeepAlive On				#默认开启了持久连接
MaxKeepAliveRequests 100	#在一个持久连接内，最多可以累计处理100次请求
KeepAliveTimeout 15			#持久连接保持时间
```

## Alias别名实践

```shell
注意：
 如果 alias 定义的目录在 DocumentRoot 之外，则需要单独授权，否则不可用
 <Directory "/path/to/alias/dir">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
 </Directory>
```

```shell
root@ubuntu-42:html # mkdir /data/server/apache/www -p
root@ubuntu-42:source # vim /etc/apache2/apache2.conf
Alias "/apache" "/data/server/apache/www"

<Directory "/data/server/apache/www">		# 指定要配置访问控制的目录路径
        Options Indexes FollowSymlinks		# 设置该目录下的可用选项：Indexes（允许目录浏览）
        									# 和FollowSymLinks（允许跟随符号链接）
        AllowOverride None					# 禁止在该目录下使用.htaccess文件进行配置覆盖
        Require all granted					# 允许所有请求访问该目录，不进行任何限制
</Directory>
```

## 属性访问权限

### options选项配置

```shell
options选项配置示例1
 	<Directory /path/>
 	--- 样式1：第二个Options生效
 		Options Includes FollowSymLinks 	#这条无效，被下面的覆盖 
 		Options Indexes     				#这条生效
 	<Directory /path/>
```

```shell
options选项配置示例2
 	<Directory /path/>
 	--- 样式2：所有条目都生效
 		Options +Indexes 
 		Options +Includes +FollowSymLinks 			#三个值都生效，并集
 	<Directory /path/>
```

```shell
options选项配置示例3
 	--- 样式3：子目录不使用父级目录的权限
 	<Directory /path/> 
 		Options Indexes FollowSymLinks 		# 同时启用目录浏览功能 和 允许跟随符号链接功能 
 	<Directory /path/>
 	<Directory /path/child/> 
 		Options +Includes -Indexes 			# 增加动态网页功能，去除目录浏览功能
 	<Directory /path/>
```

### llowOverride选项配置

AllowOverride指令用于控制 .htaccess 文件中的指令是否可以覆盖服务器配置文件中的指令。

```shell
配置格式：
 AllowOverride All|None|directive-type [directive-type] ...
 
选项解析：
 	ALL   	#所有指令都可以在 .htaccess 文件中生效
 	None  	#不使用 .htaccess 文件中的指令
 	directive-type 	#是如下指令：AuthConfig，FileInfo，Indexes，Limit，Options[=选项,...]
 		- AuthConfig  	#只能在 .htaccess 文件中配置 AuthConfig 相关指令
 		- Options=FollowSmlinks Indexes 	#可以在 .htaccess 文件中配置 FollowSmlinks 
Indexes
```

```
注意：
 	作用域范围：directory，在<Location>, <DirectoryMatch>, <Files>配置段中都是无效的。
 	控制的文件指令名称默认是 .htaccess，当然文件名也可以被 AccessFileName 选项指定
```

### Require配置

```shell
配置格式：
 	Require [not] entity-name [entity-name] ...
常见使用样例：
 	Require all granted 							#所有用户都可以访问
 	Require all denied 								#所有用户都不可以访问
 	Require method http-method [http-method] ... 	#特定请请求方法可以访问
 	Require user userid [userid] ... 				#特定用户可以访问
 	Require group group-name [group-name] ... 		#特定组中的用户可以访问
 	Require valid-user 								#所有有效用户可以访问
 	Require ip 10 172.20 192.168.2 					#指定IP可以访问
 	Require forward-dns dynamic.example.org 		#指定主机名或域名可以访问    
```

```
注意：
 	作用域 directory, .htaccess
 	某些情况下需要配合AuthName，AuthType，AuthUserFile，AuthGroupFile指令一起使用
```

```
从语法上看，允许在 <Directory> 段中使用的指令当然也可以在 <DirectoryMatch>，<Files>，<FilesMatch>，<Location>，<LocationMatch>，<Proxy>，<ProxyMatch>段中使用，但也有例外：

	- AllowOverride 指令只能出现在<Directory> 段中
    - Options指令不能用于 <Files> 和 <FilesMatch> 段
    - Options 中的 FollowSymLinks 和 SymLinksIfOwnerMatch 
   	- 只能出现在 <Directory> 段或者 .htaccess 文件中
```

### 目录访问控制

```shell
目录访问控制
vim /etc/apache2/apache2.conf
#基于正则，文件后缀名控制，禁止访问图片
<FilesMatch ".+\.(gif|jpe?g|png)$">
    Require all denied
</FilesMatch>

#基于URL路径控制
<Location "/dira">
    Require all granted
</Location>

# 基于URL路径控制，dirb中去掉 Indexes（目录列表功能）
<Location "/dirb">
    Options -Indexes
    Require all granted
</Location>

# 基于正则，URL路径控制，txt文件无法访问
<LocationMatch "/dira/.+\.txt$">
    Require all denied
</LocationMatch>
```

```shell
在浏览器中测试
http://10.0.0.13/ 				#能列出目录内容
http://10.0.0.13/dira/ 			#能列出test.log，不会列出 test.txt
http://10.0.0.13/dira/test.log 	#可以访问
http://10.0.0.13/dira/test.txt 	#报403
http://10.0.0.13/dirb/ 			#报403
http://10.0.0.13/dirb/img 		#可以访问
```

### IP访问限制

```shell
root@ubuntu-42:apache2 # vim apache2.conf
#基于URL路径控制,禁止IP访问
<Location "/dira/*">
    <RequireAll>
        Require all granted
        Require not ip 10.0.0.14
    </RequireAll>
</Location>

# 基于文件目录方式控制，仅允许IP访问
<Directory "/var/www/html/dirb/">
    <RequireAny>
        Require all denied
        Require ip 10.0.0.12
    </RequireAny>
</Directory>

# 在Directory配置段中，针对dira目录，可以进行任何请求
<Directory /var/www/html/dira>
    Require all granted
</Directory>

# 在Directory配置段中，针对dirb目录，仅限于POST请求
<Directory /var/www/html/dirb>
    Require method POST
</Directory>
```

### .htaccess配置基础

```shell
# 开启重写引擎  
RewriteEngine On  
# 禁止直接访问某些文件类型  
<FilesMatch "\.(htaccess|htpasswd|ini|log|sh|bak|tmp|old|sql|bkp|orig)$">  
    Order Allow,Deny  
    Deny from all  
    Satisfy All  
</FilesMatch>  
  
# 禁止目录浏览  
Options All -Indexes  
  
# 设置默认文档  
DirectoryIndex index.php index.html index.htm  
  
# 自定义404错误页面  
ErrorDocument 404 /error404.html  
  
# 自定义500错误页面  
ErrorDocument 500 /error500.html  
```

```shell
定制访问页面的配置整体控制
root@ubuntu42:~# cat >> /etc/apache2/apache2.conf <<-eof
# 在Directory配置段中，定制启用.htaccess能力
<Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
eof
```

```shell
root@ubuntu42:~# echo "Options -Indexes" > /var/www/html/dirb/.htaccess
```

```
将.htaccess配置文件的配置属性内容清空
root@ubuntu42:~# > /var/www/html/dirb/.htaccess
在不重启apache的前提下，直接去浏览器确认效果
- 首先是性能问题：
 	如果 AllowOverride 启用了 .htaccess 文件，则 Apache2 需要在每个目录中查找 .htaccess 文件，另外，每一次请求，都需要读取一次 .htaccess 文件
- 其次还是性能问题：
 	在启用 .htaccess 文件后，如果客户端访问的资源路径较深，则 apache2 需要读取上级目录中的.htaccess 文件，因为要保证所有的指令生效
- 另外还有安全问题：
 	如果使用 .htaccess 文件，则意味着允许apache2用户能自由的修改服务器配置
```

## 日志配置

```
root@ubuntu-42:apache2 # grep -Ev '^$|#' /etc/apache2/apache2.conf | grep -i log
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
```

```
%v：服务器名称（虚拟主机名）。
%p：服务器端口号。
%h：客户端的IP地址。
%l：远程日志名（通常是从identd获取的，但现在很少使用）。
%u：经过身份验证的用户名（如果适用）。
%t：请求到达的日期和时间。 # 通过 man 3 strftime 来获取时间信息
"%r"：请求的行（包括方法、请求的URI和协议版本）。
%>s：发送给客户端的状态码。
%O：响应的大小（以字节为单位）。
"%{Referer}i"：请求头中的Referer字段 -- 从哪一个页面跳转过来的。
"%{User-Agent}i"：请求头中的User-Agent字段 -- 浏览器头。
vhost_combined：这是为此格式指定的名称，可以在CustomLog指令中引用。
```

## 虚拟主机

### 定制多端口的虚拟主机配置

```shell
#查看默认的配置文件
root@ubuntu42:~# grep -Ev "^.*#|^$" /etc/apache2/sites-enabled/000-default.conf
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
￥确认主配置文件
root@ubuntu42:~# cat /etc/apache2/apache2.conf
root@ubuntu42:~# grep -Ev "^.*#|^$" /etc/apache2/apache2.conf
...
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

root@ubuntu-42:apache2 # mkdir /data/server/apache/web{1..3} -p
echo "base 10086 port web" > /data/server/apache/web1/index.html
echo "base 10087 port web" > /data/server/apache/web2/index.html
echo "base 10088 port web" > /data/server/apache/web3/index.html
root@ubuntu-42:apache2 # vim /etc/apache2/sites-enabled/vhost.conf
root@ubuntu-42:apache2 # cat /etc/apache2/sites-enabled/vhost.conf
Listen 10086
Listen 10087
Listen 10088

<virtualhost *:10086>
  DocumentRoot /data/server/apache/web1/
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost *:10087>
  DocumentRoot /data/server/apache/web2/
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost *:10088>
  DocumentRoot /data/server/apache/web3/
  <Directory /data/server/apache/web3/>
    Require all granted
  </Directory>
</virtualhost>
```

#### 配置基于IP区分的虚拟主机

```shell
root@ubuntu-42:apache2 # ip a a 10.0.0.186/24 dev ens33
ip a a 10.0.0.187/24 dev ens33
ip a a 10.0.0.188/24 dev ens33
root@ubuntu-42:apache2 # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:56:35:e7:76 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    altname ens33
    inet 10.0.0.42/24 brd 10.0.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
    inet 10.0.0.186/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet 10.0.0.187/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet 10.0.0.188/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe35:e776/64 scope link 
       valid_lft forever preferred_lft forever

root@ubuntu-42:apache2 # cat /etc/apache2/sites-enabled/vhost.conf
Listen 10.0.0.186:10086
Listen 10.0.0.187:10087
Listen 10.0.0.188:10088

<virtualhost 10.0.0.186:10086>
  DocumentRoot /data/server/apache/web1/
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost 10.0.0.187:10087>
  DocumentRoot /data/server/apache/web2/
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost 10.0.0.188:10088>
  DocumentRoot /data/server/apache/web3/
  <Directory /data/server/apache/web3/>
    Require all granted
  </Directory>
</virtualhost>
root@ubuntu-42:apache2 # apachectl -t
Syntax OK
root@ubuntu-42:apache2 # systemctl restart apache2
```

### **配置基于域名区分的虚拟主机**

```shell
root@ubuntu-42:apache2 # echo "10.0.0.13 www.web1.com www.web2.com www.web3.com" >> /etc/hosts
root@ubuntu-42:apache2 # cat /etc/apache2/sites-enabled/vhost.conf
Listen *:81

<virtualhost *:81>
  DocumentRoot /data/server/apache/web1/
# 定制的路径必须使用Directory配置段
  ServerName www.web1.com
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost *:81>
  DocumentRoot /data/server/apache/web2/
  ServerName www.web2.com
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</virtualhost>
<virtualhost *:81>
  DocumentRoot /data/server/apache/web3/
  ServerName www.web3.com
  <Directory /data/server/apache/web3/>
    Require all granted
  </Directory>
</virtualhost>
root@ubuntu-42:apache2 # apachectl -t
Syntax OK
root@ubuntu-42:apache2 # systemctl restart apache2
```

## 直接定制错误配置信息

**定制专属的错误信息显示页面**

```
定制错误页面
mkdir /data/server/apache/error -p
echo "<h1>The page cannot be found by hzk </h1>" > /data/server/apache/error/404.html
```

**定制应用错误提示信息的配置**

```
定制错误配置属性
root@ubuntu-42:apache2 # tail -5 /etc/apache2/apache2.conf
ErrorDocument 403 "sorry! permission denied"
ErrorDocument 404 "sorry! your page not found\n"
<Directory /var/www/html/dira>  
    Require all denied
</Directory>
```

## 定制全局错误重定向配置

**准备专属的友好提示页面**

```
准备错误访问文件
mkdir /var/www/html/error
echo '<h1>403 Page!!!</h1>' > /var/www/html/error/403.html
echo '<h1>404 Page!!!</h1>' > /var/www/html/404.html
```

**定制应用错误提示信息的配置**

```
root@ubuntu-42:apache2 # tail -6 /etc/apache2/apache2.conf
Alias "/error/" "/var/www/html/error/"
ErrorDocument 403 "/error/403.html"
ErrorDocument 404 http://10.0.0.42/404.html
<Directory /var/www/html/dira>
    Require all denied
</Directory>
```

## 定制主机错误重定向配置

准备错误信息页面

```
创建目录
mkdir -p /data/server/apache/web1/1/
```

定制应用错误提示信息的配置

```
root@ubuntu-42:apache2 # cat /etc/apache2/sites-enabled/vhost.conf 
Listen *:81

<virtualhost *:81>
  DocumentRoot /data/server/apache/web1/
  ErrorDocument 403 "<h1>sorry!</h1> permission denied"
  ErrorDocument 404 "sorry! your page not found\n"
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
  <Directory /data/server/apache/web1/1/>
    Require all denied
  </Directory>
</virtualhost>
```

检查语法后，重启服务

```
root@ubuntu42:~# apachectl -t
Syntax OK
root@ubuntu42:~# systemctl restart apache2
```

```
测试效果
root@ubuntu42:~# curl 10.0.0.13:81/
base 10086 port web
root@ubuntu42:~# curl 10.0.0.13:81/nihao
sorry! your page not found
root@ubuntu42:~# curl 10.0.0.13:81/1/
<h1>sorry!</h1> permission denied
```

## 访问控制

```
1 基于主机的访问控制：
 	允许或拒绝特定的主机或IP地址对服务器资源的访问。
 	使用<Directory>指令和Require配置项来实现。

2 基于身份验证的访问控制：
    要求用户提供有效的用户名和密码进行身份验证，才能访问服务器资源。
    使用.htaccess文件和相关的认证指令来实现。
```

```shell
1 在主配置文件中启用.htaccess支持
 <Directory "xxx">
 	AllowOverride All
 </Directory>
 
2 在.htaccess文件中配置认证信息，如认证类型、认证名称和用户文件等
	AuthType Basic 						# 采用基础认证方式
 	AuthName "Restricted Area" 			# 登录提示语，大部份浏览器"不支持"
 	AuthBasicProvider file 				# 从文件中提取验证信息
 	AuthUserFile /etc/httpd/.htpasswd 	# 指定认证文件
 	Require user xxx 					# 设定可以登录的用户 
 
注意：
 如果直接在 Directory 中定制身份验证的话，就不需要1的改动了
```

### 认证类型

```
AuthType None|Basic|Digest|Form

None：表示不对访问进行任何形式的认证。
 		这通常用于禁用之前已启用的认证机制，或者是在不需要认证的目录中设置。
Basic：一种最简单的(非安全的)HTTP认证形式。
 		它使用用户名和密码进行验证，这些信息通过Base64编码后发送给服务器。
 		浏览器会弹出一个对话框，要求用户输入用户名和密码。
Digest：一种比Basic认证更安全的认证方法。
 		它使用MD5散列算法，对用户名、密码、请求的资源URI、HTTP方法和一个服务器指定的随机数进行加密处理。
 		与Basic认证相比，Digest认证提供了更好的安全性。
Form：基于表单的认证机制，通常用于Web应用程序中。
		用户通过提交一个包含用户名和密码的HTML表单来进行认证。
 		依赖于Web页面的设计和布局，允许开发者更灵活地设计认证界面。
 		Form认证通常更易于集成到现代的Web应用程序中，并且可以提供更好的用户体验。
```

### 密码认证实验

```shell
#创建密码文件
root@ubuntu-42:~ # htpasswd -c /usr/share/apache2/apache2-pwd tom
New password: 
Re-type new password: 
Adding password for user tom
root@ubuntu-42:~ # htpasswd /usr/share/apache2/apache2-pwd hzk
New password: 
Re-type new password: 
Adding password for user hzk
root@ubuntu-42:~ # cat /usr/share/apache2/apache2-pwd 
tom:$apr1$ZlyMKiLy$BTv/n7jRUQNObL71PBobi0
hzk:$apr1$GIOhD5mK$B4N3stoFLXZQfSVJ5kVTW/
#开启模块，默认已开启
root@ubuntu-42:~ # apachectl -M | grep auth_
 auth_basic_module (shared)
 
 root@ubuntu-42:apache2 # vim /etc/apache2/apache2.conf
<Directory /var/www/>
        Options Indexes FollowSymLinks		#允许在没有索引文件
        AllowOverride None				#表示 .htaccess 文件不能覆盖该目录的配置
        # Require all granted						
        AuthType Basic					#指定使用基本身份验证（即用户名和密码）。
        AuthName "Pls Input your name and pwd"		#在弹出的身份验证对话框中显示的提示信息，提示用户输入用户名和密码。
        AuthBasicProvider file			#指定身份验证信息来源于文件（而不是数据库或其他方式）。
        AuthUserFile "/user/share/apache2/apache2-pwd"	#指定存放用户名和密码的文件路径。
        Require user tom hzk			#指定可以访问该目录的用户名，这里是 tom 和 hzk 两个用户
</Directory>
```

## 禁用模块

```shell
root@ubuntu-42:apache2 # apachectl -M | grep "status"
 status_module (shared)
root@ubuntu-42:apache2 # vim /etc/apache2/mods-enabled/status.load 
root@ubuntu-42:apache2 # cat /etc/apache2/mods-enabled/status.load 
# LoadModule status_module /usr/lib/apache2/modules/mod_status.so

root@ubuntu-42:apache2 # systemctl reload apache2.service 
root@ubuntu-42:apache2 # apachectl -M | grep "status"
```

## event模型

```shell
root@ubuntu42:~# cat /etc/apache2/mods-available/mpm_event.conf | grep -Ev "#|^$"
<IfModule mpm_event_module>
 	StartServers 2 				#服务启动时默认开启的工作进程数量
 	MinSpareThreads 25 			#最小空闲线程数
 	MaxSpareThreads 75 			#最大空闲线程数
 	ThreadLimit 64 				#每个工作进程最大能开启的线程数量
 	ThreadsPerChild 25 			#每个工作进程开启的工作线程数量
 	MaxRequestWorkers 150 		#允许启动的最大工作线程数 
 	MaxConnectionsPerChild   0 	#工作进程最多能处理多少个请求，一个工作进程处理一定数量的请求后，该进程会被父进程终止
</IfModule>
```

```
模式解读：
 	每个进程 有 25个线程，外加一个监听线程，所以每个子进程有 26个线程
 	默认开启 2个子进程，所以，一共有 52个线程记录
```

 

## prefork 模型默认配置

```shell
root@ubuntu24:~# cat /etc/apache2/mods-available/mpm_prefork.conf | grep -Ev "#|^$"
<IfModule mpm_prefork_module>
 	StartServers  5 				#服务启动时默认开启的工作进程数量
 	MinSpareServers  5 				#最小空闲进程数
 	MaxSpareServers  10 			#最大空闲进程数
 	MaxRequestWorkers  150 			#允许启动的最大工作进程数
 	MaxConnectionsPerChild   0 		#工作进程最多能处理多少个请求，
 									#一个工作进程处理一定数量的请求后，该进程会被父进程终止
</IfModule>
```

## worker 模型默认配置

```shell
root@ubuntu24:~# cat /etc/apache2/mods-available/mpm_worker.conf | grep -Ev "#|^$"
<IfModule mpm_worker_module>
 	StartServers 2 				#服务启动时默认开启的工作进程数量
	MinSpareThreads 25 			#最小空闲线程数
 	MaxSpareThreads 75 			#最大空闲线程数
 	ThreadLimit 64 				#每个工作进程最大能开启的线程数量 
 	ThreadsPerChild 25 			#每个工作进程开启的工作线程数量
 	MaxRequestWorkers  150 		#允许启动的最大工作线程数
 	MaxConnectionsPerChild   0 	#工作进程最多能处理多少个请求，
 								#一个工作进程处理一定数量的请求后，该进程会被父进程终止
</IfModule>
```

## https

### ssl环境模块

```
root@ubuntu-42:apache2 # cat /etc/apache2/mods-available/ssl.conf | grep -Ev "^#|^$"
SSLRandomSeed startup builtin
SSLRandomSeed startup file:/dev/urandom 512
SSLRandomSeed connect builtin
SSLRandomSeed connect file:/dev/urandom 512
AddType application/x-x509-ca-cert .crt
AddType application/x-pkcs7-crl .crl
SSLPassPhraseDialog  exec:/usr/share/apache2/ask-for-passphrase
SSLSessionCache     shmcb:${APACHE_RUN_DIR}/ssl_scache(512000)
SSLSessionCacheTimeout  300
SSLCipherSuite HIGH:!aNULL
SSLProtocol all -SSLv3
SSLSessionTickets off
```

```
SSLRandomSeed
	用于为 SSL/TLS 连接生成随机数。
	startup 和 connect：分别在启动和每次新连接时生成随机数。
	builtin 和 file:/dev/urandom：指定随机数来源。
		builtin 使用内置的随机数生成器。
		/dev/urandom 是 Linux 提供的伪随机数生成设备，适合快速生成随机数。
	512：指从随机数源中读取的字节数。

AddType
	application/x-x509-ca-cert .crt：将 .crt 文件的 MIME 类型设置为 application/x-x509-ca-cert，方便浏览器识别证书文件。
	application/x-pkcs7-crl .crl：将.crl文件（证书吊销列表）的MIME类型设置为application/x-
pkcs7-crl。

SSLPassPhraseDialog
	exec:/usr/share/apache2/ask-for-passphrase：指定一个外部脚本，用于提示输入 SSL 证书的密钥密码（如果密钥是加密的）。

SSLSessionCache
	shmcb:${APACHE_RUN_DIR}/ssl_scache(512000)：配置 SSL 会话缓存，使用共享内存缓存（shmcb），大小为 512,000 字节。
		这可以加速 SSL 握手过程，提升性能。
	SSLSessionCacheTimeout 300：会话缓存的超时时间为 300 秒（5 分钟）。

SSLCipherSuite
	HIGH:!aNULL：指定允许的加密套件。
		HIGH 表示优先使用强加密算法。
		!aNULL 排除不提供认证（anonymous）的加密套件，避免 MITM 攻击。

SSLProtocol
	all -SSLv3：启用所有协议（如 TLSv1.0、TLSv1.2），但禁用过时和不安全的 SSLv3 协议。

SSLSessionTickets
	off：禁用会话票据（Session Tickets）。
		这是出于安全考虑，防止会话票据密钥泄露
```

### 启用模块

```shell
root@ubuntu-42 # cd /etc/apache2/mods-enabled/
root@ubuntu-42:mods-enabled # ln -s ../mods-available/ssl.conf ./
root@ubuntu-42:mods-enabled # ln -s ../mods-available/ssl.load ./
root@ubuntu-42:mods-enabled # ln -s ../mods-available/socache_shmcb.load ./
```

### 配置ssl文件

```shell
配置端口
root@ubuntu-42:mods-enabled # vim /etc/apache2/ports.conf 
root@ubuntu-42:mods-enabled # cat /etc/apache2/ports.conf | grep -Ev "^#|^$"
Listen 80
<IfModule ssl_module>
        Listen 443
</IfModule>
<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

```
apache默认提供了ssl的配置文件
root@ubuntu-42:apache2 # cat  /etc/apache2/sites-available/default-ssl.conf | grep -Ev "^.*#|^$"
<VirtualHost *:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        SSLEngine on
        SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
        SSLCertificateKeyFile   /etc/ssl/private/ssl-cert-snakeoil.key
        <FilesMatch "\.(?:cgi|shtml|phtml|php)$">
                SSLOptions +StdEnvVars
        </FilesMatch>
        <Directory /usr/lib/cgi-bin>
                SSLOptions +StdEnvVars
        </Directory>
</VirtualHost>
```

```
<VirtualHost *:443>
	定义一个虚拟主机，监听 443 端口（HTTPS 默认端口）。
	* 表示监听所有网络接口上的请求。

ServerAdmin webmaster@localhost
	指定管理员的联系邮箱地址，通常用于在出错页面中显示。如果不需要，可以忽略。

DocumentRoot /var/www/html
	定义网站的根目录，位于 /var/www/html。存放站点的静态文件和脚本文件。

ErrorLog ${APACHE_LOG_DIR}/error.log
	定义错误日志文件的位置。${APACHE_LOG_DIR} 通常是 /var/log/apache2。

CustomLog ${APACHE_LOG_DIR}/access.log combined
	定义访问日志文件的位置和格式。
	combined 是一个常见的日志格式，包括客户端 IP、用户代理、请求方法等信息。

SSLEngine on
	启用 SSL 功能，必须在 HTTPS 虚拟主机中开启。

SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
	指定 SSL 证书文件的位置。
	这里的 ssl-cert-snakeoil.pem 是一个自签名证书，仅供测试用途。如果你是生产环境，需要替换为从 CA 获取的正式证书。

SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
	指定 SSL 私钥文件的位置，必须与证书匹配。

<FilesMatch "\.(?:cgi|shtml|phtml|php)$">
	针对特定类型的文件（如 .cgi, .shtml, .phtml, .php）启用额外的 SSL 选项。
	SSLOptions +StdEnvVars：启用标准环境变量传递给 CGI 脚本或动态内容。

<Directory /usr/lib/cgi-bin>
	针对 /usr/lib/cgi-bin 目录的配置，通常用于存放 CGI 脚本。
	同样启用了 SSLOptions +StdEnvVars。
```

### 自签https

```shell
root@ubuntu-42:sites-enabled # apt -y install easy-rsa

root@ubuntu-42:easy-rsa # cd /usr/share/easy-rsa
root@ubuntu-42:easy-rsa # ls
easyrsa  openssl-easyrsa.cnf  vars.example  x509-types
root@ubuntu-42:easy-rsa # ./easyrsa init-pki

Notice
------
'init-pki' complete; you may now create a CA or requests.

Your newly created PKI dir is:
* /usr/share/easy-rsa/pki

Using Easy-RSA configuration:
* undefined

root@ubuntu-42:easy-rsa # ls
easyrsa  openssl-easyrsa.cnf  pki  vars.example  x509-types
root@ubuntu-42:easy-rsa # tree pki/
pki/
├── inline
├── openssl-easyrsa.cnf
├── private
└── reqs

4 directories, 1 file
root@ubuntu-42:easy-rsa # ./easyrsa build-ca nopass
#查看效果
root@ubuntu-42:easy-rsa # tree pki/
pki/
├── ca.crt
├── certs_by_serial
├── index.txt
├── index.txt.attr
├── inline
├── issued
├── openssl-easyrsa.cnf
├── private
│   └── ca.key
├── reqs
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
└── serial

10 directories, 6 files
#为 www.a.com生成私钥和证书文件请求	输入www.a.com
root@ubuntu-42:easy-rsa # ./easyrsa gen-req a.com.server nopass	
#为 www.b.com生成私钥和证书文件请求 输入www.a.com
root@ubuntu-42:easy-rsa # ./easyrsa gen-req b.com.server nopass
#为 www.c.com生成私钥和证书文件请求 输入www.a.com
root@ubuntu-42:easy-rsa # ./easyrsa gen-req c.com.server nopass
#查看效果
root@ubuntu-42:easy-rsa # tree pki/
pki/
├── ca.crt
├── certs_by_serial
├── index.txt
├── index.txt.attr
├── inline
├── issued
├── openssl-easyrsa.cnf
├── private
│   ├── a.com.server.key
│   ├── b.com.server.key
│   ├── ca.key
│   └── c.com.server.key
├── reqs
│   ├── a.com.server.req
│   ├── b.com.server.req
│   └── c.com.server.req
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
└── serial
#签发证书输入 yes
root@ubuntu-42:easy-rsa # ./easyrsa sign-req server a.com.server
root@ubuntu-42:easy-rsa # ./easyrsa sign-req server b.com.server
root@ubuntu-42:easy-rsa # ./easyrsa sign-req server c.com.server

#编辑网站的https能力
root@ubuntu-42:easy-rsa # cat /etc/apache2/sites-enabled/vhost.conf 
<virtualhost *:80>
  DocumentRoot /data/server/apache/web1/
  ServerName www.a.com
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web1
  ServerName www.a.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/a.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/a.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</VirtualHost>

<virtualhost *:80>
  DocumentRoot /data/server/apache/web2/
  ServerName www.b.com
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web2/
  ServerName www.b.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/b.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/b.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</VirtualHost>

<virtualhost *:80>
  DocumentRoot /data/server/apache/web3/
  ServerName www.c.com
  <Directory /data/server/apache/web3/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web3/
  ServerName www.c.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/c.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/c.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web3/>
    Require all granted
  </Directory>
</VirtualHost>

#检查配置
root@ubuntu-42:~ # apachectl -t
Syntax OK
#重启服务
root@ubuntu-42:~ # systemctl restart apache2.service 
#编写hosts文件
root@ubuntu-42:~ # echo '10.0.0.13 www.a.com www.b.com www.c.com' >> /etc/hosts
```

## URL重定向

```
属性格式
 	Redirect [status] URL-path URL
status 			# 重定向状态码，默认temp
                # permanent，状态码301，永久重定向
                # temp，状态码302，临时重定向，默认值
                # seeother，状态码303，表示资源己经被替代
                # gone，状态码410，表示资源被删除
```

### a域名跳转到b域名

```shell
#定制配置文件
root@ubuntu-42:apache # vim /etc/apache2/sites-enabled/vhost.conf
<virtualhost *:80>
  DocumentRoot /data/server/apache/web1/
  ServerName www.a.com
  Redirect / "http://www.b.com"
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>

<virtualhost *:80>
  DocumentRoot /data/server/apache/web2/
  ServerName www.b.com
  <Directory /data/server/apache/web2/>
    Require all granted
  </Directory>
</virtualhost>
```

### http跳转到https

```shell
#定制配置文件
root@ubuntu-42:apache # cat /etc/apache2/sites-enabled/vhost.conf 
<virtualhost *:80>
  DocumentRoot /data/server/apache/web1/
  ServerName www.a.com
  Redirect permanent "/" "https://www.a.com"
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web1
  ServerName www.a.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/a.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/a.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</VirtualHost>
```

### Rewrite指令实现重定向

```shell
#加载模块
root@ubuntu24:easy-rsa# cd /etc/apache2/mods-enabled/
root@ubuntu24:mods-enabled# ln -sv ../mods-available/rewrite.load ./rewrite.load
'./rewrite.load' -> '../mods-available/rewrite.load'
#定制配置文件
root@ubuntu-42:mods-enabled # cat /etc/apache2/sites-enabled/vhost.conf 
<virtualhost *:80>
  DocumentRoot /data/server/apache/web1/
  ServerName www.a.com
  RewriteRule ^(/.*)$ https://%{HTTP_HOST}$1 [redirect=302]
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web1
  ServerName www.a.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/a.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/a.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</VirtualHost>
```

## HSTS

### 工作原理

```shell
1、服务器响应 HSTS 头：
	当浏览器访问一个支持 HSTS 的网站时，服务器会在 HTTPS 响应头中包含 Strict-Transport-Security 头信息。
	该头部告诉浏览器，今后一定要通过 HTTPS 来访问该网站，且在指定的有效期内不允许访问 HTTP 版本。
	#Strict-Transport-Security: max-age=31536000; includeSubDomains
		max-age：指定浏览器缓存这个指令的有效期（单位：秒）。例如，max-age=31536000 表示 1 年。
		includeSubDomains（可选）：指示 HSTS 规则适用于所有子域名。如果没有此参数，只有当前域会被强制使用 HTTPS。
		
2、浏览器缓存 HSTS 信息：
	一旦浏览器收到该响应头，它会将网站的域名及对应的 max-age 信息缓存一段时间。
	这意味着即使用户通过 HTTP 请求访问该网站，浏览器也会自动重定向到 HTTPS 版本。
	例如，如果 max-age=31536000，浏览器将在 1 年内强制使用 HTTPS 访问该网站。

3、自动重定向：
	当用户在未来访问同一网站时，浏览器会检查是否缓存了 HSTS 信息。如果缓存了，则会自动将所有 HTTP 请求重定向到 HTTPS。
	即使用户输入了 http://example.com，浏览器也会将请求自动转换为 https://example.com。
	
4、首次访问需要 HTTPS：
	需要注意的是，HSTS 仅在通过 HTTPS 访问网站时才会生效。
	如果第一次访问该网站时没有通过 HTTPS，而是通过 HTTP 访问，则浏览器不会启用 HSTS。
	因此，第一次访问必须通过 HTTPS 才能触发 HSTS 机制。
```

### HSTS 与服务端HTTPS重定向的区别

```
	服务端HTTPS重定向是指客户端每次访问HTTP协议的URL资源，然后服务端返回301或302，客户端再去请求HTTP资源。
 	服务端HTTPS重定向是在客户端发起HTTP请求时，由服务器"根据配置"触发的。服务器会返回一个重定向响应码和新的HTTPS URL，"引导"客户端使用HTTPS协议进行通信。
 	它依赖于服务器的正确配置。如果服务器配置不当或受到攻击，可能导致重定向失败或重定向到恶意网站。
 	
 	HSTS 是指客户端首次访问 HTTP 协议的 URL 资源，然后服务端返回带有 HSTS 的重定向头，在HSTS 规定的有效期内，下次浏览器再次访问该网站的 HTTP 协议的资源时，浏览器将直接换成 HTTPS 协议再去访问。
 	HSTS是在浏览器"首次"访问支持该策略的网站时，由服务器通过"HTTP响应头"触发的。一旦触发，浏览器会在指定的时间内"强制"使用HTTPS协议访问该网站。
	HSTS通过强制浏览器使用HTTPS协议，可以有效防止中间人攻击和协议降级攻击。
```

### HSTS 预加载列表的工作原理

1. **预加载机制：**
   当浏览器访问预加载列表中的域名时，它会在首次访问之前就知道该网站要求使用 HTTPS，无需通过 HTTP 请求。即使用户在浏览器地址栏输入 `http://example.com`，浏览器也会直接将请求转换为 `https://example.com`。
2. **无需首次访问：**
   通常，HSTS 只有在首次通过 HTTPS 访问网站后才会生效。但是，如果网站域名在 HSTS 预加载列表中，浏览器在访问该网站时会直接强制使用 HTTPS，而无需进行第一次的 HTTP 请求。这极大提升了安全性，因为不再依赖于首次通过 HTTPS 来激活 HSTS。
3. **预加载列表的作用：**
   通过将域名添加到 HSTS 预加载列表中，网站所有的访问都会立即使用 HTTPS，避免了初次访问时可能遭遇的安全风险（如 SSL 剥离攻击）。同时，浏览器厂商将预加载列表内的所有域名提前在浏览器中“硬编码”，这意味着用户即使没有访问过该网站，浏览器也知道要使用 HTTPS。

```shell
#加载模块
root@ubuntu24:# cd /etc/apache2/mods-enabled/
root@ubuntu-42:mods-enabled # ln -s ../mods-available/headers.load ./
#定制配置文件
root@ubuntu-42:mods-enabled # cat /etc/apache2/sites-enabled/vhost.conf 
<virtualhost *:80>
  DocumentRoot /data/server/apache/web1/
  ServerName www.a.com
# 添加此行为客户端添加一个头，有效期1年
  Header always set Strict-Transport-Security "Max-age=31536000"
  Redirect permanent "/" "https://www.a.com"
# 定制的路径必须使用Directory配置段
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</virtualhost>

<VirtualHost *:443>
  DocumentRoot /data/server/apache/web1
  ServerName www.a.com
  SSLEngine on
  SSLCertificateFile /usr/share/easy-rsa/pki/issued/a.com.server.crt
  SSLCertificateKeyFile /usr/share/easy-rsa/pki/private/a.com.server.key
  SSLCACertificateFile /usr/share/easy-rsa/pki/ca.crt
  <Directory /data/server/apache/web1/>
    Require all granted
  </Directory>
</VirtualHost>
```

## LAMP

```
LAMP架构是一种常用的Web应用程序开发和部署架构，它由四个核心组件组成，分别是：
    - L 是指 Linux 操作系统
    - A 是指 Apache ，用来提供Web服务
    - M 指 MySQL，用来提供数据库服务
    - P 指 PHP，是动态网站的的一种开发语言
    
 LAMP 是中小型动态网站的常见组合，虽然这些开放源代码程序本身并不是专门设计成同另几个程序一起工作的，但由于它们各自的特点和普遍性，使得这个组合在搭建动态网站这一领域开始流行起来。
```

```
1 客户端发送请求连接到Web服务器的80端口（默认用于HTTP通信的端口）。
2 Apache作为Web服务器软件，负责接收和处理HTTP请求。
 	2-1 如果客户端请求的是静态资源（如HTML、CSS、JavaScript文件等），
 		Apache会查找该资源并直接返回给客户端。
 	2-2 如果客户端请求的是动态资源（如PHP文件），
 		2-2-1 Apache会加载并调用与PHP解析相关的模块（如libphpX.so）来解析PHP代码，并生成动态内容。
 		2-2-2 在处理动态资源时，如果有需要与后台数据库进行交互的操作（如查询、插入、更新等），
 		2-2-3 PHP程序会通过适当的方式（如MySQL扩展库）与后台数据库建立连接并执行相应的操作。
 	2-3 最后，Apache将处理结果发送回客户端作为HTTP响应。
```

![image-20241203151017073](D:\桌面\mage.md\笔记\5day-png\17LAMP.png)

```
在 Apache2 中，主要有两种方式实现 PHP 网页文件的动态解析 - 模块加载、FastCGI

模块加载
模块加载运行方式（mod_php）
 	PHP 处理器（解释器）以模块的形式加载到 Apache2 中，成为 Apache2 的内置功能，当客户端请求PHP 动态资源时，由 Apache2 中的内置模块完成 PHP 动态代码的解析，再将运行结果返回给客户端
 	使用这种方式的优点是性能相对较好，因为 PHP 代码的执行与 WEB 服务器在同一进程中，避免了额外的进程通信开销，缺点是可伸缩性较差，因为 Apache2 和 PHP 解析器是一个整体，无法拆分

FastCGI
FastCGI 运行方式
 	FastCGI 是一种通信协议，允许 Web 服务器与外部的 FastCGI 进程（例如 PHP 解释器）进行通信，当 Apache2 收到 PHP 动态资源请求的时候，就将此解析请求转发给 FastCGI 进程，由 FastCGI 进程负责解析PHP，再将解析结果回传给 Apache2，然后 Apache2 再将结果返回给客户端
 	当使用 FastCGI 时，每个请求都可以由一个独立的外部 FastCGI 进程处理，这提供了更好的可伸缩性，因为 FastCGI 进程可以独立于 Web 服务器运行，但需要消耗一些额外的通信开销
```

### CGI和FastCGI

### **CGI（通用网关接口）**

**CGI** 是一种较为传统的技术，允许 Web 服务器通过执行外部程序来生成动态内容。

#### **工作原理：**

1. **请求触发：** 当用户通过浏览器访问一个动态资源（如 `.cgi` 程序）时，Web 服务器会将请求转发给 CGI 程序。
2. **启动进程：** Web 服务器为每个请求创建一个新的操作系统进程，并运行对应的 CGI 程序。
3. **数据传递：** Web 服务器通过环境变量和标准输入（stdin）将请求数据传递给 CGI 程序，CGI 程序将结果返回给服务器，通过标准输出（stdout）输出内容。
4. **结束进程：** CGI 程序处理完成后会终止，服务器将结果发送给用户。

#### **优点：**

1. **简单易用：** 实现简单，适合小型站点或偶尔的动态功能需求。
2. **独立性强：** 每个请求运行在独立的进程中，互不干扰，增加了安全性。

#### **缺点：**

1. **性能低下：** 每个请求都需要创建和销毁一个新的进程，这对系统资源（如 CPU、内存）消耗巨大。
2. **扩展性差：** 随着请求数量增加，系统很快会被大量进程占用，导致性能急剧下降。



### **FastCGI（快速通用网关接口）**

**FastCGI** 是 CGI 的一种改进版本，旨在解决 CGI 性能低下的问题。

#### **工作原理：**

1. **长时间运行：** FastCGI 程序不是为每个请求启动一个新进程，而是运行在一个长期驻留的进程或进程池中。
2. **请求复用：** Web 服务器将请求发送给已经运行的 FastCGI 进程，这些进程可以反复处理多个请求，而不需要为每个请求重新创建进程。
3. **通信方式：** 使用套接字（Socket）或管道与 Web 服务器通信，从而大幅降低进程创建和销毁的开销。
4. **进程管理：** FastCGI 可以通过进程池机制控制工作进程的数量，从而更高效地使用系统资源。

#### **优点：**

1. **高性能：** 通过复用进程和减少进程创建开销，大幅提升性能。
2. **更好的扩展性：** FastCGI 可以通过增加进程池中的进程数量来应对高并发需求。
3. **与服务器分离：** FastCGI 可以运行在远程服务器上，与 Web 服务器分离，这为分布式架构提供了可能性。

#### **缺点：**

1. **配置复杂：** 相比 CGI，FastCGI 的配置更复杂，需要额外管理进程池和通信机制。
2. **调试难度：** 由于长时间运行，可能会导致内存泄漏或其他资源管理问题，需要更严格的程序管理。



### **CGI 和 FastCGI 的对比**

| 特性           | CGI                      | FastCGI                        |
| -------------- | ------------------------ | ------------------------------ |
| **进程模型**   | 每个请求启动一个新进程   | 使用进程池复用进程             |
| **性能**       | 低，适合少量请求         | 高，适合高并发场景             |
| **资源利用率** | 资源消耗大，进程创建频繁 | 资源利用率高，避免频繁创建进程 |
| **复杂性**     | 简单易用                 | 配置和管理较复杂               |
| **扩展性**     | 差，不适合高流量         | 好，适合扩展性需求             |
| **运行环境**   | 独立进程                 | 进程池，支持远程运行           |

## PHP

### 模块形式- apache应用php

```shell
#默认是event模型
root@ubuntu-42:mods-enabled # apachectl -M | grep mpm
 mpm_event_module (shared)
#默认没有安装PHP
root@ubuntu-42:mods-enabled # apachectl -M | grep php
#安装PHP
root@ubuntu-42:mods-enabled # apt -y install php
#PHP工作目录
root@ubuntu-42:mods-enabled # ls /etc/php/8.3/
apache2  cli  mods-available
#安装完成后event变成prefork
root@ubuntu-42:mods-enabled # apachectl -M | grep mpm
 mpm_prefork_module (shared)
root@ubuntu-42:mods-enabled # ls /etc/apache2/mods-enabled/mpm*
/etc/apache2/mods-enabled/mpm_prefork.conf  /etc/apache2/mods-enabled/mpm_prefork.load
#存在了对应的php模块
root@ubuntu-42:mods-enabled # apachectl -M | grep php
 php_module (shared)
root@ubuntu-42:mods-enabled # ls /etc/apache2/mods-enabled/php*
/etc/apache2/mods-enabled/php8.3.conf  /etc/apache2/mods-enabled/php8.3.load
root@ubuntu-42:apache2 # cd /etc/apache2/sites-enabled/
root@ubuntu-42:sites-enabled # ln -s ../sites-available/000-default.conf ./
root@ubuntu-42:html # vim /var/www/html/index.php
<?php
phpinfo();
?>
```

### fpm方式- 部署php软件

```shell
root@ubuntu-42:html # apt install php-fpm -y
root@ubuntu-42:html # ls /etc/php/8.3/fpm
conf.d  php-fpm.conf  php.ini  pool.d
#设置PHP-FPM代理(需要额外的fcgi的加载)
root@ubuntu-42:mods-enabled # ln -s /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-enabled/
root@ubuntu-42:mods-enabled # ln -sv ../mods-available/proxy_fcgi.load ./
'./proxy_fcgi.load' -> '../mods-available/proxy_fcgi.load'
root@ubuntu-42:mods-enabled # ln -sv ../mods-available/proxy.conf  ./
'./proxy.conf' -> '../mods-available/proxy.conf'
root@ubuntu-42:mods-enabled # ln -sv ../mods-available/proxy.load  ./
'./proxy.load' -> '../mods-available/proxy.load'
#定制php功能访问
#配置代理
root@ubuntu-42:mods-enabled # cat /etc/apache2/mods-enabled/proxy.conf | tail -3
<FilesMatch ".+/.ph(ar|p|tml)$">
    SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost"
</FilesMatch>
```

```
<FilesMatch> 指令：
	匹配符合特定正则表达式的文件。
	正则表达式 .+/.ph(ar|p|tml)$ 匹配以下文件：
		.php：标准 PHP 文件。
		.phar：PHP 的可执行归档文件。
		.phtml：历史上用于 PHP 的文件扩展名，现代应用中较少见。
SetHandler 指令：
	指定如何处理这些匹配的文件。
	"proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost" 的含义：
		proxy:unix:/run/php/php8.3-fpm.sock：
			使用 UNIX 套接字 /run/php/php8.3-fpm.sock 作为通信通道，将请求发送到 PHP-FPM 服务。
			套接字是本地通信的一种高效方式，常用于 Web 服务器和 PHP-FPM 的连接。
fcgi://localhost：
	表示通过 FastCGI 协议与后端 PHP-FPM 进程通信。
	localhost 是一个占位符，实际不会通过网络连接，因为这里用的是本地套接字。
```

## 博客案例

```
部署流程
1. 安装 Apache2，PHP-FPM，MySQL 等相关软件
2. 配置 Apache2，PHP-FPM，保证 Apache2 能解析 PHP
3. 配置MySQL，创建数据库，连接账号，完成授权等
4. 下载 Wordpress 源码，并解析到相关目录
5. 在 Apache2 中配置虚拟主机
6. 在浏览器中测试，完成部署流程
```

```shell
#安装软件
root@ubuntu-43:~ # apt install mysql-server -y
root@ubuntu-43:~ # apt install apache2 -y
root@ubuntu-43:~ # apt install php-fpm php-mysqlnd php-json php-gd php-xml php-mbstring php-zip -y
root@ubuntu-43:~ # apt install php-fpm -y

#php配置
root@ubuntu-43:~ # mkdir /var/www/html/blog.com/
root@ubuntu-43:~ # echo '<?php echo "hello world"; ?>' > /var/www/html/blog.com/test.php
root@ubuntu-43:~ # rm -rf /etc/apache2/sites-enabled/000-default.conf
root@ubuntu-43:~ # vim /etc/apache2/sites-enabled/blog.com.conf 
<virtualhost *:80>
  DocumentRoot /var/www/html/blog.com/
  ServerName blog.hzk.com
  <FilesMatch \.php$>
    SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost"
  </FilesMatch>
  <Directory /var/www/html/blog.com/>
    Require all granted
  </Directory>
</virtualhost>
root@ubuntu-43:~ # cd /etc/apache2/mods-enabled/
root@ubuntu-43:mods-enabled # rm -rf php8.3.*
root@ubuntu-43:mods-enabled # ln -sv ../mods-available/proxy_fcgi.load ./proxy_fcgi.load
'./proxy_fcgi.load' -> '../mods-available/proxy_fcgi.load'
root@ubuntu-43:mods-enabled # ln -sv ../mods-available/proxy.conf ./proxy.conf
ln: failed to create symbolic link './proxy.conf': File exists
root@ubuntu-43:mods-enabled # ln -sv ../mods-available/proxy.load ./proxy.load
'./proxy.load' -> '../mods-available/proxy.load'
root@ubuntu-43:mods-enabled # cat proxy.conf
<FilesMatch ".+/.ph(ar|p|tml)$">
    SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost"
</FilesMatch>
root@ubuntu-43:~ # systemctl restart apache2.service 
root@ubuntu-43:~ # systemctl restart php8.3
root@ubuntu-43:~ # curl 10.0.0.43/test.php
hello word

#mysql配置
root@ubuntu-43:~ # sed -i '/^bind-address/s#127.0.0.1#10.0.0.43#' /etc/mysql/mysql.conf.d/mysqld.cnf
root@ubuntu-43:~ # grep bind-address /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address            = 10.0.0.43
root@ubuntu-43:~ # systemctl restart mysql.service
root@ubuntu-43:~ # mysql
mysql> create database wordpress;
mysql> create user 'wordpresser'@'10.0.0.%' identified with mysql_native_password by '123456';
mysql> grant all on wordpress.* to 'wordpresser'@'10.0.0.%';
mysql> exit
root@ubuntu-43:~ # mysql -u'wordpresser' -h 10.0.0.43 -p'123456' -e "select version();"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| version()               |
+-------------------------+
| 8.0.40-0ubuntu0.24.04.1 |
+-------------------------+


#获取wordpress软件
root@ubuntu-43:~ # mkdir /data/softs -p
root@ubuntu-43:~ # cd /data/softs/
root@ubuntu-43:softs # wget https://cn.wordpress.org/latest-zh_CN.zip
root@ubuntu-43:softs # ls
latest-zh_CN.zip
root@ubuntu-43:softs # apt install unzip
root@ubuntu-43:softs # unzip latest-zh_CN.zip
root@ubuntu-43:softs # mv wordpress/* /var/www/html/blog.com/
root@ubuntu-43:softs # chown -R www-data:www-data /var/www/html/blog.com
```

