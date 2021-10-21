# 写一个Web框架(Go)

## 为什么要从零开发框架？

不管什么语言，在 Web 领域，总有很多款开发框架，真的可以说是日新月异，[GitHub 上的 Go Web 框架](https://github.com/mingrammer/go-web-framework-stars)现在已经有 41 款了。

选用框架需要放到特定场景上讨论，这个特定场景，就是开发团队的业务环境。

比如，如果你一个人负责一个小的外包项目，那可以说所有框架都差不多；如果是 2～3 个服务端人员的初创团队，你用的框架要承担的工作大概率是提升开发效率，那就要选择功能更强大、易用性更好的框架；等团队再大一些，框架所承担的职责就更多了，往往得更多考虑性能、扩展性，或者定制自己的框架。

**聊聊 Web 框架**

所有框架都有倾向性，我觉得基本可以分为两个大类：一类是追求运行性能，一类是追求开发效率。

追求性能的框架往往很简洁，包含的东西也很少，一个路由一个 MVC 就完事了，比如 Gin 框架；另外一类框架追求开发效率，封装得非常好，很多功能会让你惊叹，能帮你省很多事，最典型的就是 Beego 框架。

在 Web 领域，特别是中小型项目，开发效率往往才是业务的第一需求。一个产品拥有的市场机会总是转瞬即逝的，抢占市场都是靠更快的开发速度和迭代速度。所以，在任何领域做到第一名的产品基本上都有一个共性：开发、迭代速度快。这就和古龙小说中评价天下侠士的武功一样，唯快不破。

这就要求好的框架要能区分清楚业务团队和架构团队的边界，写架构的同学，做好框架的底层封装。而写业务的同学可以从底层实现中释放出来，专注于业务逻辑，遇到任何底层问题，在框架中都有简单易用的封装可用，框架中的每一个类、每一个服务接口都在告诉你，要完成这个功能，只需要这样使用，无需更多的操作。

**Go Web 框架**

在 Go 出现之前，我们都知道，PHP 和 Java 基本上是 Web 领域的主流，特别是 PHP，是当时各种公司建站的首选。其实 PHP 和 HTML 的契合性是好于 Go 的，尤其在快速编写动态网页的时候，因为它可嵌入 HTML 的特性，于是建立 Web 站就变成了一件非常容易的事。

但是这几年，Web 领域越来越流行前后端分离，Web 后端开发慢慢从繁杂的 HTML 渲染中解放出来了，开始把研发注意力放在如何建立数据模型（接口设计）、如何搭建更稳定可扩展的服务（分布式）上。在这个大趋势下，Go 在 Web 领域可以说是大放异彩。

<u>Go 真的很好地平衡了开发和性能。</u>

比方说，Go 语言中的 Goroutine 设计，提供了“一个请求一个协程”的请求模型，对比 PHP 的“一个请求一个进程”的模型，能有效提升后端的资源占用和调度负载；另外，Go 的 Runtime 机制让运行程序不再依赖各种的环境和库，将 Web 服务的部署和搭建变得简单高效；而 Go 提供的交叉编译、数据结构、channel 等语言级别特性，都让“处理 Web 请求”这个事情变得非常简单。

所以这几年，Go 在各个领域，比如云原生、游戏、微服务、区块链等等，优势也越发明显。这也是我为什么选择 Golang 来搭建 Web 框架的原因，一方面看好 Golang 的市场前景，另一方面 Golang 确实让 Web 开发变得越来越有效率。

**学习路径**

- 实战第一关：我会带你分析 Web 框架的本质，从最底层的 Go 的 HTTP 库开始讲起，如何基于 HTTP 库建立 server、如何搭建路由、如何增加中间件等等，从而搭建出一个 Web 框架最核心的设计部分。
- 实战第二关：框架核心搭建好了，我们会基于具体业务场景重新思考：设计框架的目标到底是什么? 框架的设计感和要解决的问题在哪里? 框架的倾向性是什么? 如果要搭建出一个“一切皆服务”的框架应该如何设计。
- 实战第三关：我将带你为这个框架增加不同的周边功能，在添加功能时，我们会先讨论目前社区中的标准做法是什么样的，有没有更好的设计，最终把这些标准做法融合到我们的框架中。
- 实战第四关：现在框架已经基本搭建完成了，我们会用这个框架应用开发一个统计管理后台，我们使用 vue-element-admin 来做前端封面，再结合我们的框架开发具体的统计展示和计算业务。

每节课我都会给到代码 demo，你能跟着课程一步步实现。课程的所有代码，包括这个框架我都已经放在 [GitHub](https://github.com/gohade/hade) 上了，还有完整的[说明文档](http://hade.funaio.cn/)。

## 一、从零开始

### 1.net/http：使用标准库搭建Server并不是那么简单

我们从最底层的 HTTP 协议开始，搞清楚 Web Server 本质，通过 net/http 代码库梳理 HTTP 服务的主流程脉络，先知其所以然，再搭建框架的 Server 结构。

#### Web Server 的本质

Web Server 是一个通过 HTTP 协议处理 Web 请求的计算机系统。这句话乍听有点绕口，我给你解释下。

HTTP 协议，在 OSI 网络体系结构中，是基于 TCP/IP 之上第七层应用层的协议，全称叫做超文本传输协议。啥意思？就是说 HTTP 协议传输的都是文本字符，只是这些字符是有规则排列的。这些字符的排列规则，就是一种约定，也就是协议。这个协议还有一个专门的描述文档，就是[RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616)。

对于 HTTP 协议，无论是请求还是响应，传输的消息体都可以分为两个部分：HTTP 头部和 HTTP Body 体。头部描述的一般是和业务无关但与传输相关的信息，比如请求地址、编码格式、缓存时长等；Body 里面主要描述的是与业务相关的信息。

<u>Web Server 的本质，实际上就是接收、解析 HTTP 请求传输的文本字符，理解这些文本字符的指令，然后进行计算，再将返回值组织成 HTTP 响应的文本字符，通过 TCP 网络传输回去。</u>

#### 一定要用标准库吗

对 Web Server 来说，Golang 提供了 net 库和 net/http 库，分别对应 OSI 的 TCP 层和 HTTP 层，它们两个负责的就是 HTTP 的接收和解析。

一般我们会使用 net/http 库解析 HTTP 消息体。但是可能会有人问，如果我想实现 Web 服务，可不可以不用 net/http 库呢？比如我直接用 net 库，逐字读取消息体，然后自己解析获取的传输字符。

答案是可以的，如果你有兼容其它协议、追求极致性能的需求，而且你有把握能按照 HTTP 的 RFC 标准进行解析，那完全可以自己封装一个 HTTP 库。

其实在一些大厂中确实是这么做的，每当有一些通用的协议需求，比如一个服务既要支持 HTTP，又要支持 Protocol Buffers，又或者想要支持自定义的协议，那么他们就可能抛弃 HTTP 库，甚至抛弃 net 库，直接自己进行网络事件驱动，解析 HTTP 协议。

有个开源库，叫 [FastHTTP](https://github.com/valyala/fasthttp)，它就是抛弃标准库 net/http 来实现的。作者为了追求极高的 HTTP 性能，自己封装了网络事件驱动，解析了 HTTP 协议。你感兴趣的话，可以去看看。

但是现在绝大部分的 Web 框架，都是基于 net/http 标准库的。我认为原因主要有两点：
- **第一是相信官方开源的力量**。自己实现 HTTP 协议的解析，不一定会比标准库实现得更好，即使当前标准库有一些不足之处，我们也都相信，随着开源贡献者越来越多，标准库也会最终达到完美。
- **第二是 Web 服务架构的变化**。随着容器化、Kubernetes 等技术的兴起，业界逐渐达成共识，单机并发性能并不是评判 Web 服务优劣的唯一标准了，易用性、扩展性也是底层库需要考量的。

所以总体来说，net/http 标准库，作为官方开源库，其易用性和扩展性都经过开源社区和 Golang 官方的认证，是我们目前构建 Web Server 首选的 HTTP 协议库。

用 net/http 来创建一个 HTTP 服务，其实很简单，下面是[官方文档](https://pkg.go.dev/net/http@go1.15.5)里的例子。我做了些注释，帮你理解。
```go

// 创建一个Foo路由和处理函数
http.Handle("/foo", fooHandler)

// 创建一个bar路由和处理函数
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

// 监听8080端口
log.Fatal(http.ListenAndServe(":8080", nil))
```
这五行代码做了什么，为什么就能启动一个 HTTP 服务，具体的逻辑是什么样的？

要回答这些问题，你就要深入理解 net/http 标准库。要不然，只会简单调用，却不知道原理，后面哪里出了问题，或者你想调优，就无从下手了。

#### net/http 标准库怎么学

**这里我教给你一个快速掌握代码库的技巧：库函数 > 结构定义 > 结构函数。**


简单来说，就是当你在阅读一个代码库的时候，不应该从上到下阅读整个代码文档，而应该先阅读整个代码库提供的对外库函数（function）=> 再读这个库提供的结构（struct/class）=> 最后再阅读每个结构函数（method）。

为什么要这么学呢？因为这种阅读思路和代码库作者的思路是一致的。首先搞清楚这个库要提供什么功能（提供什么样的对外函数），然后为了提供这些功能，我要把整个库分为几个核心模块（结构），最后每个核心模块，我应该提供什么样的能力（具体的结构函数）来满足我的需求。

##### 库函数（功能）

按照这个思路，我们来阅读 net/http 库，先看提供的对外库函数是为了实现哪些功能。这里顺带补充说明一下，我们课程对应的 Golang 源码的版本是 1.15.5，你可以在01 分支的 [coredemo/go.mod](https://github.com/gohade/coredemo/blob/geekbang/01/go.mod)里看到。

你直接通过 go doc net/http | grep "^func" 命令行能查询出 net/http 库所有的对外库函数：
```go

func CanonicalHeaderKey(s string) string
func DetectContentType(data []byte) string
func Error(w ResponseWriter, error string, code int)
func Get(url string) (resp *Response, err error)
func Handle(pattern string, handler Handler)
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func Head(url string) (resp *Response, err error)
func ListenAndServe(addr string, handler Handler) error
func ListenAndServeTLS(addr, certFile, keyFile string, handler Handler) error
func MaxBytesReader(w ResponseWriter, r io.ReadCloser, n int64) io.ReadCloser
func NewRequest(method, url string, body io.Reader) (*Request, error)
func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error)
func NotFound(w ResponseWriter, r *Request)
func ParseHTTPVersion(vers string) (major, minor int, ok bool)
func ParseTime(text string) (t time.Time, err error)
func Post(url, contentType string, body io.Reader) (resp *Response, err error)
func PostForm(url string, data url.Values) (resp *Response, err error)
func ProxyFromEnvironment(req *Request) (*url.URL, error)
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)
func ReadRequest(b *bufio.Reader) (*Request, error)
func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)
func Redirect(w ResponseWriter, r *Request, url string, code int)
func Serve(l net.Listener, handler Handler) error
func ServeContent(w ResponseWriter, req *Request, name string, modtime time.Time, ...)
func ServeFile(w ResponseWriter, r *Request, name string)
func ServeTLS(l net.Listener, handler Handler, certFile, keyFile string) error
func SetCookie(w ResponseWriter, cookie *Cookie)
func StatusText(code int) string
```
在这个库提供的方法中，我们去掉一些 New 和 Set 开头的函数，因为你从命名上可以看出，这些函数是对某个对象或者属性的设置。

剩下的函数大致可以分成三类：
- 为服务端提供创建 HTTP 服务的函数，名字中一般包含 Serve 字样，比如 Serve、ServeFile、ListenAndServe 等。
- 为客户端提供调用 HTTP 服务的类库，以 HTTP 的 method 同名，比如 Get、Post、Head 等。
- 提供中转代理的一些函数，比如 ProxyURL、ProxyFromEnvironment 等。

我们现在研究的是，如何创建一个 HTTP 服务，所以关注包含 Serve 字样的函数就可以了。
```go

// 通过监听的URL地址和控制器函数来创建HTTP服务
func ListenAndServe(addr string, handler Handler) error{}
// 通过监听的URL地址和控制器函数来创建HTTPS服务
func ListenAndServeTLS(addr, certFile, keyFile string, handler Handler) error{}
// 通过net.Listener结构和控制器函数来创建HTTP服务
func Serve(l net.Listener, handler Handler) error{}
// 通过net.Listener结构和控制器函数来创建HTTPS服务
func ServeTLS(l net.Listener, handler Handler, certFile, keyFile string) error{}
```

##### 结构定义（模块）

然后，我们过一遍这个库提供的所有 struct，看看核心模块有哪些，同样使用 go doc:
```go
 go doc net/http | grep "^type"|grep struct
```
```go
type Client struct{ ... }
type Cookie struct{ ... }
type ProtocolError struct{ ... }
type PushOptions struct{ ... }
type Request struct{ ... } 
type Response struct{ ... }
type ServeMux struct{ ... }
type Server struct{ ... }
type Transport struct{ ... }
```
看结构的名字或者 go doc 查看结构说明文档，能逐渐了解它们的功能：
- Client 负责构建 HTTP 客户端；
- Server 负责构建 HTTP 服务端；
- ServerMux 负责 HTTP 服务端路由；
- Transport、Request、Response、Cookie 负责客户端和服务端传输对应的不同模块。

现在通过库方法（function）和结构体（struct），我们对整个库的结构和功能有大致印象了。整个库承担了两部分功能，一部分是构建 HTTP 客户端，一部分是构建 HTTP 服务端。

构建的 HTTP 服务端除了提供真实服务之外，也能提供代理中转服务，它们分别由 Client 和 Server 两个数据结构负责。除了这两个最重要的数据结构之外，HTTP 协议的每个部分，比如请求、返回、传输设置等都有具体的数据结构负责。

##### 结构函数（能力）

下面从具体的需求出发，我们来阅读具体的结构函数（method）。

我们当前的需求是创建 HTTP 服务，开头我举了一个最简单的例子：
```go
// 创建一个Foo路由和处理函数
http.Handle("/foo", fooHandler)

// 创建一个bar路由和处理函数
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

// 监听8080端口
log.Fatal(http.ListenAndServe(":8080", nil))
```
我们跟着 http.ListenAndServe 这个函数来理一下 net/http 创建服务的主流程逻辑。

阅读具体的代码逻辑用 go doc 命令明显就不够了，你需要两个东西：

一个是可以灵活进行代码跳转的 IDE，VS Code 和 GoLand 都是非常好的工具。以我们现在要查看的 http.ListenAndServe 这个函数为例，我们可以从上面的例子代码中，直接通过 IDE 跳转到这个函数的源码中阅读.

另一个是可以方便记录代码流程的笔记，这里我的个人方法是使用思维导图。具体方法是将要分析的代码从入口处一层层记录下来，每个函数，我们只记录其核心代码，然后对每个核心代码一层层解析。记得把思维导图的结构设置为右侧分布，这样更直观。

比如下面这张图，就是我解析部分 HTTP 库服务端画的代码分析图。
![](_images/4-1.jpg)

我们先顺着 http.ListenAndServe 的脉络读。

![](_images/4-2.jpg)

**第一层**，http.ListenAndServe 本质是通过创建一个 Server 数据结构，调用 server.ListenAndServe 对外提供服务，这一层完全是比较简单的封装，目的是，将 Server 结构创建服务的方法 ListenAndServe ，直接作为库函数对外提供，增加库的易用性。

**第二层**，创建服务的方法 ListenAndServe 先定义了监听信息 net.Listen，然后调用 Serve 函数。

**第三层 Serve 函数中**，用了一个 for 循环，通过 l.Accept不断接收从客户端传进来的请求连接。当接收到了一个新的请求连接的时候，通过 srv.NewConn创建了一个连接结构（http.conn），并创建一个 Goroutine 为这个请求连接对应服务（c.serve）。

从第四层开始，后面就是单个连接的服务逻辑了。
![](_images/4-3.jpg)

**第四层**，c.serve函数先判断本次 HTTP 请求是否需要升级为 HTTPs，接着创建读文本的 reader 和写文本的 buffer，再进一步读取本次请求数据，然后**第五层**调用最关键的方法 serverHandler{c.server}.ServeHTTP(w, w.req) ，来处理这次请求。

这个关键方法是为了实现自定义的路由和业务逻辑，调用写法是比较有意思的：
```go
serverHandler{c.server}.ServeHTTP(w, w.req)
```
serverHandler 结构体，是标准库封装的，代表“请求对应的处理逻辑”，它只包含了一个指向总入口服务 server 的指针。

这个结构将总入口的服务结构 Server 和每个连接的处理逻辑巧妙联系在一起了，你可以看接着的**第六层**逻辑：
```go
// serverHandler 结构代表请求对应的处理逻辑
type serverHandler struct {
  srv *Server
}

// 具体处理逻辑的处理函数
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
  handler := sh.srv.Handler
  if handler == nil {
    handler = DefaultServeMux
  }
  ...
  handler.ServeHTTP(rw, req)
}
```
如果入口服务 server 结构已经设置了 Handler，就调用这个 Handler 来处理此次请求，反之则使用库自带的 DefaultServerMux。

这里的 serverHandler 设计，能同时保证这个库的扩展性和易用性：你可以很方便使用默认方法处理请求，但是一旦有需求，也能自己扩展出方法处理请求。

那么 DefaultServeMux 是怎么寻找 Handler 的呢，这就是思维导图的最后一部分**第七层**。
![](_images/4-4.jpg)
DefaultServeMux.Handle 是一个非常简单的 map 实现，key 是路径（pattern），value 是这个 pattern 对应的处理函数（handler）。它是通过 mux.match(path) 寻找对应 Handler，也就是从 DefaultServeMux 内部的 map 中直接根据 key 寻找到 value 的。

这种根据 map 直接查找路由的方式是不是可以满足我们的路由需求呢？我们会在第三讲路由中详细解说。

好，HTTP 库 Server 的代码流程我们就梳理完成了，整个逻辑线大致是：创建服务 -> 监听请求 -> 创建连接 -> 处理请求.

这里我也给你整理了一下逻辑线各层的关键结论：
- 第一层，标准库创建 HTTP 服务是通过创建一个 Server 数据结构完成的；
- 第二层，Server 数据结构在 for 循环中不断监听每一个连接；
- 第三层，每个连接默认开启一个 Goroutine 为其服务；
- 第四、五层，serverHandler 结构代表请求对应的处理逻辑，并且通过这个结构进行具体业务逻辑处理；
- 第六层，Server 数据结构如果没有设置处理函数 Handler，默认使用 DefaultServerMux 处理请求；
- 第七层，DefaultServerMux 是使用 map 结构来存储和查找路由规则。

这条逻辑线是 HTTP 服务启动最核心的主流程逻辑，后面我们会基于这个流程继续开发，你要掌握到能背下来的程度。

#### 创建框架的 Server 结构

咱也分析了主流程代码，其中第一层的关键结论就是：net/http 标准库创建服务，实质上就是通过创建 Server 数据结构来完成的。所以接下来，我们就来创建一个 Server 数据结构。

通过 go doc net/http.Server 我们可以看到 Server 的结构：
```go
type Server struct {
    // 请求监听地址
  Addr string
    // 请求核心处理函数
  Handler Handler 
  ...
}
```
其中最核心的是 Handler 这个字段，从主流程中我们知道（第六层关键结论），当 Handler 这个字段设置为空的时候，它会默认使用 DefaultServerMux 这个路由器来填充这个值，但是我们一般都会使用自己定义的路由来替换这个默认路由。

所以在框架代码中，我们要创建一个自己的核心路由结构，实现 Handler。

先来理一下目录结构，我们在GitHub上创建一个项目 [coredemo](https://github.com/gohade/coredemo/tree/geekbang/01)，这个项目是这门课程所有的代码集合，包含要实现的框架和使用框架的示例业务代码。

<u>所有的框架代码都存放在 framework 文件夹中，而所有的示例业务代码都存放在 framework 文件夹之外。这里为了后面称呼方便，我们就把 framework 文件夹叫框架文件夹，而把外层称为业务文件夹。</u>

当然 GitHub 上的这个 coredemo 是我在写课程的时候为了演示创建的，推荐你跟着一步一步写。成品在[hade 项目](https://github.com/gohade/hade)里，你可以先看看，在最后发布的时候，我们会将整个项目进行发布。在一个新的业务中，如果要使用到我们自己写好的框架，可以直接通过引用 “import 项目地址 /framework” 来引入，在最后一部分做实战项目的时候我们会具体演示。

好，下面我们来一步步实现这个项目~。

创建一个 framework 文件夹，新建 core.go，在里面写入。
```go

package framework

import "net/http"

// 框架核心结构
type Core struct {
}

// 初始化框架核心结构
func NewCore() *Core {
  return &Core{}
}

// 框架核心结构实现Handler接口
func (c *Core) ServeHTTP(response http.ResponseWriter, request *http.Request) {
  // TODO
}

```
而在业务文件夹中创建 main.go，其中的 main 函数就变成这样：
```go

func main() {
  server := &http.Server{
        // 自定义的请求核心处理函数
    Handler: framework.NewCore(),
        // 请求监听地址
    Addr:    ":8080",
  }
  server.ListenAndServe()
}

```
整理下这段代码，我们通过自己创建了 Server 数据结构，并且在数据结构中创建了自定义的 Handler（Core 数据结构）和监听地址，实现了一个 HTTP 服务。这个服务的具体业务逻辑都集中在我们自定义的 Core 结构中，后续我们要做的事情就是不断丰富这个 Core 数据结构的功能逻辑

后续每节课学完之后，我都会把代码放在对应的 GitHub 的分支中。你跟着课程敲完代码过程中有不了解的地方，可以对比参考分支。

本节课我们完成的代码分支是：geekbang/01 ，代码结构我也截了图：

![](_images/4-5.jpg)

#### 小结

今天我以 net/http 标准库为例，分享了快速熟悉代码库的技巧，库函数 > 结构定义 > 结构函数。在阅读代码库时，从功能出发，先读对外库函数，再细读这个库提供的结构，搞清楚功能和对应结构之后，最后基于实际需求看每个结构函数。

主流程的链条比较长，但是你先理顺逻辑，记住几个关键的节点，再结合思维导图，就能记住整个主流程逻辑了，之后所有关于 HTTP 的细节和问题，我们都会基于这个主流程逻辑来思考和回答。

### 2. Context：请求控制器，让每个请求都在掌控之中

今天我将带你进一步丰富我们的框架，添加上下文 Context 为请求设置超时时间。

从主流程中我们知道（第三层关键结论），HTTP 服务会为每个请求创建一个 Goroutine 进行服务处理。在服务处理的过程中，有可能就在本地执行业务逻辑，也有可能再去下游服务获取数据。如下图，本地处理逻辑 A，下游服务 a/b/c/d， 会形成一个标准的树形逻辑链条。
![](_images/4-6.jpg)

在这个逻辑链条中，每个本地处理逻辑，或者下游服务请求节点，都有可能存在超时问题。**而对于 HTTP 服务而言，超时往往是造成服务不可用、甚至系统瘫痪的罪魁祸首。**

系统瘫痪也就是我们俗称的雪崩，某个服务的不可用引发了其他服务的不可用。比如上图中，如果服务 d 超时，导致请求处理缓慢甚至不可用，加剧了 Goroutine 堆积，同时也造成了服务 a/b/c 的请求堆积，Goroutine 堆积，瞬时请求数加大，导致 a/b/c 的服务都不可用，整个系统瘫痪，怎么办？

最有效的方法就是从源头上控制一个请求的“最大处理时长”，所以，对于一个 Web 框架而言，“超时控制”能力是必备的。今天我们就用 Context 为框架增加这个能力。

#### context 标准库设计思路

如何控制超时，官方是有提供 context 标准库作为解决方案的，但是由于标准库的功能并不够完善，一会我们会基于标准库，来根据需求自定义框架的 Context。所以理解其背后的设计思路就可以了。

<u>为了防止雪崩，context 标准库的解决思路是：在整个树形逻辑链条中，用上下文控制器 Context，实现每个节点的信息传递和共享。</u>

具体操作是：用 Context 定时器为整个链条设置超时时间，时间一到，结束事件被触发，链条中正在处理的服务逻辑会监听到，从而结束整个逻辑链条，让后续操作不再进行。

明白操作思路之后，我们深入 context 标准库看看要对应具备哪些功能。

按照上一讲介绍的了解标准库的方法，我们先通过 go doc context | grep "^func" 看提供了哪些库函数（function）：

```go
// 创建退出 Context
func WithCancel(parent Context) (ctx Context, cancel CancelFunc){}
// 创建有超时时间的 Context
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc){}
// 创建有截止时间的 Context
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc){}
```
其中，WithCancel 直接创建可以操作退出的子节点，WithTimeout 为子节点设置了超时时间（还有多少时间结束），WithDeadline 为子节点设置了结束时间线（在什么时间结束）。

但是这只是表层功能的不同，其实这三个库函数的本质是一致的。怎么理解呢？

我们先通过 go doc context | grep "^type" ，搞清楚 Context 的结构定义和函数句柄，再来解答这个问题。
```go

type Context interface {
    // 当 Context 被取消或者到了 deadline，返回一个被关闭的 channel
    Done() <-chan struct{}
    ...
}

//函数句柄
type CancelFunc func() 
```
这个库虽然不大，但是设计感强，比较抽象，并不是很好理解。所以这里，我把 Context 的其他字段省略了。现在，我们只理解核心的 Done() 方法和 CancelFunc 这两个函数就可以了。

在树形逻辑链条上，一个节点其实有两个角色：一是下游树的管理者；二是上游树的被管理者，那么就对应需要有两个能力

- 一个是能让整个下游树结束的能力，也就是函数句柄 CancelFunc；
- 另外一个是在上游树结束的时候被通知的能力，也就是 Done() 方法。同时因为通知是需要不断监听的，所以 Done() 方法需要通过 channel 作为返回值让使用方进行监听。

[官方示例](https://pkg.go.dev/context@go1.15.5)
```go

package main

import (
  "context"
  "fmt"
  "time"
)

const shortDuration = 1 * time.Millisecond

func main() {
    // 创建截止时间
  d := time.Now().Add(shortDuration)
    // 创建有截止时间的 Context
  ctx, cancel := context.WithDeadline(context.Background(), d)
  defer cancel()

    // 使用 select 监听 1s 和有截止时间的 Context 哪个先结束
  select {
  case <-time.After(1 * time.Second):
    fmt.Println("overslept")
  case <-ctx.Done():
    fmt.Println(ctx.Err())
  }

}
```
主线程创建了一个 1 毫秒结束的定时器 Context，在定时器结束的时候，主线程会通过 Done() 函数收到事件结束通知，然后主动调用函数句柄 cancelFunc 来通知所有子 Context 结束（这个例子比较简单没有子 Context）。我打个更形象的比喻，CancelFunc 和 Done 方法就像是电话的话筒和听筒，话筒 CancelFunc，用来告诉管辖范围内的所有 Context 要进行自我终结，而通过监听听筒 Done 方法，我们就能听到上游父级管理者的终结命令。

<u>总之，CancelFunc 是主动让下游结束，而 Done 是被上游通知结束。</u>

搞懂了具体实现方法，我们回过头来看这三个库函数 WithCancel / WithDeadline / WithTimeout 就很好理解了。

它们的本质就是“通过定时器来自动触发终结通知”，WithTimeout 设置若干秒后通知触发终结，WithDeadline 设置未来某个时间点触发终结。

对应到 Context 代码中，它们的功能就是：为一个父节点生成一个带有 Done 方法的子节点，并且返回子节点的 CancelFunc 函数句柄。
![](_images/4-7.jpg)

我们用一张图来辅助解释一下，Context 的使用会形成一个树形结构，下游指的是树形结构中的子节点及所有子节点的子树，而上游指的是当前节点的父节点。比如图中圈起来的部分，当 WithTimeout 调用 CancelFunc 的时候，所有下游的 With 系列产生的 Context 都会从 Done 中收到消息。

#### Context 是怎么产生的

现在我们已经了解标准库 context 的设计思路了，在开始写代码之前，我们还要把 Context 放到 net/http 的主流程逻辑中，其中有两个问题要搞清楚：<u>Context 在哪里产生？它的上下游逻辑是什么？</u>

要回答这两个问题，可以用我们在上一讲介绍的思维导图方法，因为主流程已经拎清楚了，现在你只需要把其中 Context 有关的代码再详细过一遍，然后在思维导图上标记出来就可以了。

这里，我已经把 Context 的关键代码都用蓝色背景做了标记，你可以检查一下自己有没有标漏。
![](_images/4-8.jpg)
照旧看图梳理代码流程，来看蓝色部分，从前到后的层级梳理就不再重复讲了，我们看关键位置。

照旧看图梳理代码流程，来看蓝色部分，从前到后的层级梳理就不再重复讲了，我们看关键位置。

从图中最后一层的代码 req.ctx = ctx 中看到，每个连接的 Context 最终是放在 request 结构体中的。

而且这个时候， Context 已经有多层父节点。因为，在代码中，每执行一次 WithCancel、WithValue，就封装了一层 Context，我们通过这一张流程图能清晰看到最终 Context 的生成层次。
![](_images/4-9.jpg)
你发现了吗，其实每个连接的 Context 都是基于 baseContext 复制来的。对应到代码中就是，在为某个连接开启 Goroutine 的时候，为当前连接创建了一个 connContext，这个 connContext 是基于 server 中的 Context 而来，而 server 中 Context 的基础就是 baseContext。