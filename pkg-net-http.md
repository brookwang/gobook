### 介绍
http包提供了HTTP客户端和服务端的实现

### 引用示例
import (
	"net/http"	
)

|方法|介绍|示例|
|:---|:---|:---|
|Get|利用get方法请求指定的url，Get请求指定的页面信息，并返回实体主体|func (c *Client) Get(url string) (resp *Response, err error)|
|Head|利用head方法请求指定的url，Head只返回页面的首部|func (c *Client) Head(url string) (resp *Response, err error)|
|Post|利用post方法请求指定的URl,如果body也是一个io.Closer,则在请求之后关闭它|func (c *Client) Head(url string) (resp *Response, err error)|
|PostForm|利用post方法请求指定的url,利用data的key和value作为请求体.|func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)|
|Do|Do发送http请求并且返回一个http响应,遵守client的策略,如重定向,cookies以及auth等.当调用者读完resp.body之后应该关闭它|func (c *Client) Do(req *Request) (resp *Response, err error)|
|CanonicalHeaderKey|返回header key的规范化形式,规范化形式是以"-"为分隔符,每一部分都是首字母大写,其他字母小写.例如"accept-encoding" 的标准化形式是 "Accept-Encoding".|func CanonicalHeaderKey(s string) string|
|DetectContentType|检查给定数据的内容类型Content-Type,最多检测512byte数据,如果有效的话,该函数返回一个MIME类型,否则的话,返回一个"application/octet-stream"|func DetectContentType(data []byte) string|
|Error|利用指定的错误信息和Http code来响应请求,其中错误信息必须是纯文本.|func Error(w ResponseWriter, error string, code int)|
|NotFound|返回HTTP404 not found错误|func NotFound(w ResponseWriter, r *Request)|
|Handle|将handler按照指定的格式注册到DefaultServeMux,ServeMux解释了模式匹配规则|func Handle(pattern string, handler Handler)|
|HandleFunc|同上，主要用来实现动态文件内容的展示，这点与ServerFile()不同的地方。|func HandleFunc(pattern string, handler func(ResponseWriter, *Request))|
|ListenAndServe|监听TCP网络地址addr然后调用具有handler的Serve去处理连接请求.通常情况下Handler是nil,使用默认的DefaultServeMux|func ListenAndServe(addr string, handler Handler) error|
|ListenAndServeTLS|该函数与ListenAndServe功能基本相同,二者不同之处是该函数需要HTTPS连接.也就是说,必须给该服务Serve提供一个包含整数的秘钥的文件,如果证书是由证书机构签署的,那么证书文件必须是服务证书之后跟着CA证书.|func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error|
|ServeFile|利用指定的文件或者目录的内容来响应相应的请求.|func ServeFile(w ResponseWriter, r *Request, name string)|
|SetCookie|给w设定cookie|func SetCookie(w ResponseWriter, cookie *Cookie)|
|StatusText|对于http状态码返回文本表示,如果这个code未知,则返回空的字符串.|func StatusText(code int) string|
|Serve|接受Listener l的连接，创建一个新的服务协程。该服务协程读取请求然后调用srv.Handler来应答。实际上就是实现了对某个端口进行监听，然后创建相应的连接。|func (srv *Server) Serve(l net.Listener) error|
|SetKeepAlivesEnabled|该函数控制是否http的keep-alives能够使用，默认情况下，keep-alives总是可用的。只有资源非常紧张的环境或者服务端在关闭进程中时，才应该关闭该功能。|func (s *r) SetKeepAlivesEnabled(v bool)|
|CancelRequest|通过关闭连接来取消传送中的请求。|func (t *Transport) CancelRequest(req *Request)|
|CloseIdleConnections|关闭所有之前请求但目前处于空闲状态的连接。该方法并不中断任何正在使用的连接。|func (t *Transport) CloseIdleConnections()|
||||


   

   
    
    func (t *Transport) RegisterProtocol(scheme string, rt RoundTripper)//RegisterProtocol注册一个新的名为scheme的协议。t会将使用scheme协议的请求转交给rt。rt有责任模拟HTTP请求的语义。RegisterProtocol可以被其他包用于提供"ftp"或"file"等协议的实现。
    func (t *Transport) RoundTrip(req *Request) (resp *Response, err error)//该函数实现了RoundTripper接口，对于高层http客户端支持，例如处理cookies以及重定向，查看Get，Post以及Client类型。
