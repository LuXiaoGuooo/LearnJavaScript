<h3 id="eXdRR">Day1 V8引擎以及浏览器运行原理</h3>
<h5 id="YfOz8">浏览器输入一个URL并enter的时候，到底发生了什么</h5>
1. 当你在浏览器中输入一个url的时候，通常是域名地址，直接通过域名是无法找到服务器的，因为服务器本质是一套拥有IP地址的主机，浏览器会先进行一个DNS域名解析。
    1. 第一步的话他会去找缓存记录
        1. 浏览器缓存：首先，浏览器会检查它的缓存中是否有这个记录。
        2. 如果没有的话，浏览器就会询问操作系统，因为操作系统也可能有自己的DNS缓存
        3. 如果操作系统也没有的话，请求会发送到本地网络的路由器，因为它可能也有自己的DNS缓存
        4. 如果都没有的话，请求最终会发送到对应的网络的ISP
        5. 如果在缓存找到的话，就可以使用对应的IP地址去连接主机，
    2. 如果所有的本地缓存查找都失败了的话，DNS会进行一个递归查询，以下我拿<font style="color:rgba(17, 17, 51, 0.5);background-color:rgba(175, 184, 193, 0.2);"> www.baidu.com. </font><font style="background-color:rgba(175, 184, 193, 0.2);">为例子</font>
        1. 首先的话，DNS查询会发送到根域名服务器，根域名服务器是最高级别的DNS域名，负责重定向顶级域名的顶级域名（这个根域名服务器就是www.baidu.com. 最后的那个.）
        2. 根服务器会告诉你的ISP的DNS服务器去查询哪个顶级域名服务器来找到.com域的信息。这个服务器掌握所有.com域名以及相应服务器的信息 (Inertnet Service Provider 就是互联网服务商，比如中国联通，其中顶级域名服务器就是.com .cn 这种）
        3. 一旦DNS查询到了正确的顶级域名服务器，它会进一步定向到负责baidu.com的权威服务器，然后权威服务器有该域名的ip地址
    3. 最后权威服务器会提供baidu.com域名对应的IP地址，这个信息会发送到用户的电脑中
    4. 一旦ip地址被找到，它会在用户的电脑中缓存起来。
2. 建立TCP连接，进行
    1. 虽然我们发送的是HTTP请求，但是HTTP协议是应用层的协议，它是建立在TCP传输层协议之上，所以我们需要先进行TCP链接
    2. TCP连接就是三次握手，客户端发送SYN包初始化一个链接，服务器收到后返回一个SYN-ACK包做响应，客户端再次发送一个ACK包作为回应，完成握手过程
    3. 这个时候TCP连接建立完成，双方开始传输数据了。
3. 发送HTTP请求
    1. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">一旦TCP建立连接成功，客户端就可以通过这个链接发送HTTP请求，包括请求方法（GET、Post）、URI、协议版本、请求头、请求体。</font>
    2. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">服务器收到HTTP请求后，会处理这个请求，并且返回一个HTTP响应。 </font>
    3. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">HTTP响应会包含状态码、响应头、响应内容，我们这里的请求通常是index.html文件</font>
4. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">下载HTML，CSS，JS</font>
    1. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">浏览器会先下载index.html文件。</font>
    2. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">这个过程中他会遇到link元素和script元素</font>
    3. <font style="color:rgb(20.000000%, 20.000000%, 20.000000%);background-color:rgb(100.000000%, 100.000000%, 100.000000%);">遇到link和sprict元素会继续向服务器发送HTTP请求，并且下载css、js文件</font>

![](https://cdn.nlark.com/yuque/0/2025/png/34962139/1765173656585-cb363027-a6db-4aca-9bf9-dc3af331f5d9.png)

5. 渲染页面
    1. 解析HTML，创建DOM Tree
    2. 同时遇到css的link元素时候，浏览器会下载对应的CSS文件
    3. 下载css文件之后，解析css文件，解析出对应的CSSOM ， CSS规则树
    4. DOM树和CSSOM共同构建Render树
        1. link元素不会阻塞DOM Treee的构建过程，但是会阻塞Render Tree的构建过程
        2. Render Tree 和 DOM Tree并不是一一对应的，比如说display为none的元素，就不会出现在render Tree中
    5. 后在渲染树上运行布局（layout）计算每个节点的具体位置
    6. 再由浏览器将每个阶段绘制（paint）到屏幕的像素点

![](https://cdn.nlark.com/yuque/0/2025/png/34962139/1765175034810-e55dd072-f142-4902-a257-79e476351784.png)

<h5 id="v1bdM">JavaScript引擎</h5>
1. 高级的编程语言都是需要转化为机器指令执行的
2. 所以我们编写的javascript无论交给浏览器还是Node，最后都需要被Cpu执行
3. 但是Cpu只认识机器指令，就是0101
4. 所以我们需要javascript引擎帮助我们将Javascript代码翻译成Cpu指令

常见的js引擎有

![](https://cdn.nlark.com/yuque/0/2025/png/34962139/1765175567406-ac4fb951-42c8-421a-839c-8177f8020ba2.png)

<h5 id="sCVYr">关于浏览器内核和JS引擎之间的关系</h5>
浏览器内核就是渲染引擎，比如说开源的浏览器引擎WebKit，其实这个就是由两部分组成的

1. WebCore：负责HTML解析，布局，渲染等相关工作
2. JavaScriptCore：就是解析，执行JavaScript代码的

<h5 id="SZgk0">关于V8引擎（待补充）</h5>
官方链接：[https://v8.dev/blog/scanner](https://v8.dev/blog/scanner) 



