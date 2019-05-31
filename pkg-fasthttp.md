fasthttp是由valyala编写的声称快于Go官方标准库net/http包十倍的快速HTTP实现。  
从各方的性能测试结果来看（评测一、评测二），fasthttp作为当下最快的http Go语言包当之无愧。


### fasthttp包 
import "github.com/valyala/fasthttp"

fasthttp包提供了快速HTTP服务端及客户端API。

### Fasthttp功能

　　*速度优化。在现代硬件上，可轻松实现处理超过100K qps，超过100万的keep-alive长连接并发。

　　*低内存使用优化。

　　*通过RequestCtx.Hijack，支持易连接升级“Connection:Upgrade”协议。

　　*服务端支持请求pipelining流水线操作。可以从一个网络包中读取多个请求，亦可以从一个网络包中发送多个响应。这十分利于支撑高负载的REST服务。

　　*服务端提供以下反Dos攻击限制：

　　　　*并发连接数量限制。

　　　　*每个客户端IP并发连接数量限制。

　　　　*每个连接请求数限制。

　　　　*请求读取超时限制。

　　　　*响应写入超时限制。

　　　　*最大header请求头大小限制。

　　　　*最大body请求体大小限制。

　　　　*最大请求执行时间限制。

　　　　*最大keep-alive长连接生命周期限制。

　　　　*提供早期过滤掉非GET请求功能。

　　*诸多额外有用的信息被暴露在请求处理程序handler中：

　　　　*服务端及客户端地址。

　　　　*每一个请求的日志。

　　　　*唯一的请求id。

　　　　*请求开始时间。

　　　　*连接开始时间。

　　　　*当前连接的请求序列号。

　　*客户端支持失败请求自动重发。

　　*fasthttp API的设计具有扩展现有客户端及服务端实现，或重新编写自定义客户端及服务端的能力。

func AppendBytesStr(dst []byte, src string) []byte
AppendBytesStr函数用于附加src源字符串至dst目的字节切片中，并返回扩展后的dst目的字节切片结果。
该函数与append(dst, src...)相比并没有性能优势。此处保留仅出于向后兼容考虑。
该函数并不推荐使用，未来可能被移除。

func AppendGunzipBytes(dst, src []byte) ([]byte, error)
AppendGunzipBytes函数用于附加gunzip压缩源src字节切片至dst目的字节切片中，并返回处理后的dst目的字节切片结果及错误信息。

func AppendGzipBytes(dst, src []byte) []byte
func AppendGzipBytes函数用于附加gzip压缩源src字节切片至dst目的字节切片中，并返回处理后的dst目的字节切片。

func AppendGzipBytesLevel(dst, src []byte, level int) []
AppendGzipBytesLevel函数用于以给定压缩级别附加gzip压缩源src字节切片至目的字节切片，并返回处理后的dst目的字节切片结果。
支持的压缩级别：
* compressnocompression	未压缩
* compressbestspeed	最快速度
* compressbestcompression	最大压缩比
* compressdefaultcompression	默认压缩级别

func AppendHTMLEscape(dst []byte, s string) []byte
AppendHTMLEscape函数用于附加含有转义字符的HTML字符串s至dst目的字节切片中，并返回扩展后的dst目的字节切片结果。

func AppendHTMLEscapeBytes(dst, s []byte) []byte
AppendHTMLEscapeBytes函数用于附加含有转移字符的HTML字节切片s至dst目的字节切片中，并返回扩展后的dst目的字节切片结果。

func AppendHTTPDate(dst []byte, date time.Time) []byte
AppendHTTPDate函数用于附加符合RFC1123格式的日期time.Time类型date至dst目的字节切片中，并返回扩展后的dst目的字节切片结果。

func AppendIPv4(dst []byte, ip net.IP) []byte
AppendIPv4函数用于附加给定ipv4字符串表达式至dst目的字节切片。并返回扩展后的dst目的字节切片结果。

func AppendNormalizedHeaderKey(dst []byte, key string) []byte
AppendNormalizedHeaderKey函数用于附加正常的header key键名字符串至dst目的字节切片，并返回dst目的字节切片结果。
正常的header key以大写字母开头，且破折号后的第一个字母同样为大写字母，而其他字母则由小写字母表示。例如：
* coNTENT-TYPe -> Content-Type
* HOST -> Host
* foo-bar-baz -> Foo-Bar-Baz

func AppendNormalizedHeaderKeyBytes(dst, key []byte) []byte
AppendNormalizedHeaderKeyBytes函数用于附加正常的header key键名字节切片至dst目的字节切片，并返回dst目的字节切片结果。
正常的header key以大写字母开头，且破折号后的第一个字母同样为大写字母，而其他字母则由小写字母表示。例如：
* coNTENT-TYPe -> Content-Type
* HOST -> Host
* foo-bar-baz -> Foo-Bar-

func AppendQuotedArg(dst, src []byte) []byte
AppendQuotedArg函数用于附加编码过的url src源字节切片至dst目的字节切片中，并返回附加过的dst目的字节切片结果。

func AppendUint(dst []byte, n int) []byte
AppendUint函数用于附加整型变量n至dst目的字节切片中，并返回扩展后的dst目的字节切片。

func Dial(addr string) (net.Conn, error)
Dial函数使用tcp4拨打给定TCP地址。
该函数与net.Dial相比具有以下附加功能：
*利用DefaultDNSCacheDuration缓存解决TCP寻址，降低了DNS解析器负载。
*其会以轮询方式拨打所有已解决的TCP地址直到连接被建立。如果某些地址暂时无法访问，其将会起到作用。
*如果在DefaultDialTimeout秒内连接无法被建立，其将会返回ErrDialTimeout错误。
此拨号器用于在传递给Client.Dial或HostClient.Dial前，包装自定义代码。
例如，每个主机计数器或及其限制可能通过此种包装器实现。
传递给该函数的地址必需包含端口号。
例如地址值为：
* foobar.baz:443
* foo.bar:80
* aaa.com:8080

func DialDualStack(addr string) (net.Conn, error)
DialDualStack函数使用tcp4或tcp6拨打给定TCP地址。
该函数与net.Dial相比具有以下附加功能：
*利用DefaultDNSCacheDuration缓存解决TCP寻址，降低了DNS解析器负载。
*其会以轮询方式拨打所有已解决的TCP地址直到连接被建立。如果某些地址暂时无法访问，其将会起到作用。
*如果在DefaultDialTimeout秒内连接无法被建立，其将会返回ErrDialTimeout错误。使用DialDualStackTimeout自定义拨号超时。
此拨号器用于在传递给Client.Dial或HostClient.Dial前，包装自定义代码。
例如，每个主机计数器或及其限制可能通过此种包装器实现。
传递给该函数的地址必需包含端口号。
例如地址值为：
* foobar.baz:443
* foo.bar:80
* aaa.com:8080

func DialDualStackTimeout(addr string, timeout time.Duration) (net.Conn, error)
DialDualStackTimeout函数会在给定超时时间内使用tcp4或tcp6拨打给定TCP地址。
该函数与net.Dial相比具有以下附加功能：
*利用DefaultDNSCacheDuration缓存解决TCP寻址，降低了DNS解析器负载。
*其会以轮询方式拨打所有已解决的TCP地址直到连接被建立。如果某些地址暂时无法访问，其将会起到作用。
此拨号器用于在传递给Client.Dial或HostClient.Dial前，包装自定义代码。
例如，每个主机计数器或及其限制可能通过此种包装器实现。
传递给该函数的地址必需包含端口号。
例如地址值为：
* foobar.baz:443
* foo.bar:80
* aaa.com:8080

func DialTimeout(addr string, timeout time.Duration) (net.Conn, error)
DialTimeout函数会在给定超时时间内使用tcp4拨打给定TCP地址。
该函数与net.Dial相比具有以下附加功能：
*利用DefaultDNSCacheDuration缓存解决TCP寻址，降低了DNS解析器负载。
*其会以轮询方式拨打所有已解决的TCP地址直到连接被建立。如果某些地址暂时无法访问，其将会起到作用。
此拨号器用于在传递给Client.Dial或HostClient.Dial前，包装自定义代码。
例如，每个主机计数器或及其限制可能通过此种包装器实现。
传递给该函数的地址必需包含端口号。
例如地址值为：
* foobar.baz:443
* foo.bar:80

func Do(req *Request, resp *Response) error
Do函数用于执行给定HTTP请求，并填充给定HTTP响应。
请求必需是具有完整URL格式（包括scheme及host）且非零的RequestURI，或是非零Host header + RequestURI的形式。
客户端将根据以下规则决定被请求的服务器：
-若RequestURI包含具有scheme及host的完整URL，将由其决定。
-否则由Host header决定。
该函数不遵循重定向。如需重定向，请使用Get*函数。
若resp为nil，则响应将被忽略。
若DefaultMaxConnsPerHost所连接的请求主机当前出于繁忙状态，ErrNoFreeConns将被返回。
建议在对性能要求严格的代码中，通过AcquireRequest和AcquireResponse获取req及resp。

func DoDeadline(req *Request, resp *Response, deadline time.Time) error
DoDeadline函数用于执行给定请求并在规定截止时间deadline前等待响应。
请求必需是具有完整URL格式（包括scheme及host）且非零的RequestURI，或是非零Host header + RequestURI的形式。
客户端将根据以下规则决定被请求的服务器：
-若RequestURI包含具有scheme及host的完整URL，将由其决定。
-否则由Host header决定。
该函数不遵循重定向。如需重定向，请使用Get*函数。
若resp为nil，则响应将被忽略。
若直到给定截止时间deadline响应仍未被返回，则ErrTimeout将被返回。
建议在对性能要求严格的代码中，通过AcquireRequest和AcquireResponse获取req及resp。

func DoTimeout(req *Request, resp *Response, timeout time.Duration) error
DoTimeout函数用于执行给定请求并在给定超时持续时间timeout内等待响应。
请求必需是具有完整URL格式（包括scheme及host）且非零的RequestURI，或是非零Host header + RequestURI的形式。
客户端将根据以下规则决定被请求的服务器：
-若RequestURI包含具有scheme及host的完整URL，将由其决定。
-否则由Host header决定。
该函数不遵循重定向。如需重定向，请使用Get*函数。
若resp为nil，则响应将被忽略。
若直到给定超时时间timeout响应仍未被返回，则ErrTimeout将被返回。
建议在对性能要求严格的代码中，通过AcquireRequest和AcquireResponse获取req及resp。

func EqualBytesStr(b []byte, s string) bool
若string(b)==成立，则EqualBytesStr函数将会返回true。
该函数与string(b)==s相比不具有性能优势。此处保留仅出于向后兼容考虑。
该函数并不推荐使用，未来可能被移除。

func FileLastModified(path string) (time.Time, error)
FileLastModified用于返回最近一次修改文件的时间。

func Get(dst []byte, url string) (statusCode int, body []byte, err error)
Get函数用于附加url字符串内容至dst目的字节切片，并将其返回作为body主体。
该函数跟随重定向。如需手动处理重定向，请使用Do*函数。
若dst目的字节切片为nil，则新body主体缓冲将被分配。

func GetDeadline(dst []byte, url string, deadline time.Time) (statusCode int, body []byte, err error)
GetDeadline函数用于附加url字符串内容至dst目的字节切片中，并将其返回作为body主体。
该函数跟随重定向。如需手动处理重定向，请使用Do*函数。
若dst目的字节切片为nil，则新body主体缓冲将被分配。
若url内容在给定超时时间timeout内无法被取出，则ErrTimeout将会被返回。

func ListenAndServe(addr string, handler RequestHandler) error
ListenAndServe函数用于服务给定TCP地址addr及给定handler的HTTP请求。

func ListenAndServeTLS(addr, certFile, keyFile string, handler RequestHandler) error
ListenAndServeTLS函数用于服务给定TCP地址addr及给定handler的HTTPS请求。
certFile、keyFile分别为到TLS证书及密钥文件的路径。

func ListenAndServeTLSEmbed(addr string, certData, keyData []byte, handler RequestHandler) error
ListenAndServeTLSEmbed函数用于服务给定TCP地址addr及给定handler的HTTPS请求。
certData与keyData必需包含有效的TLS证书及密钥数据。

func ListenAndServeUNIX(addr string, mode os.FileMode, handler RequestHandler) error
ListenAndServeUNIX函数用于服务给定UNIX地址addr及给定handler的HTTP请求。
该函数在开始服务前会删除addr地址现有的文件。
UNIX地址addr的服务器将设置为给定文件模式。

func NewStreamReader(sw StreamWriter) io.ReadCloser
NewStreamReader函数将会返回一个由sw生成的重放其所有数据的读取器。
被返回的读取器可被传递到Response.SetBodyStream中。
Close必须在返回的读取器中所有所需数据被读取后调用，否则Go协程可能会发生泄漏。
详情请参见Response.SetBodyStreamWriter。

func ParseByteRange(byteRange []byte, contentLength int) (startPos, endPos int, err error)
ParseByteRange函数用于解析“Range:bytes=...”格式的header请求头的值。
其遵从 https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.35 。

func ParseHTTPDate(date []byte) (time.Time, error)
ParseHTTPData函数用于解析遵从RFC1123格式的日期。

func ParseIPv4(dst net.IP, ipStr []byte) (net.IP, error)
ParseIPv4函数用于将ipStr字节切片解析到net.IP类型的dst中，并返回扩展后的dst目的net.IP及错误信息。

func ParseUfloat(buf []byte) (float64, error)
ParseUfloat函数用于从buf字节切片中解析无符号浮点数。

func ParseUint(buf []byte) (int, error)
ParseUint函数用于从buf字节切片中解析无符号整数。

func Post(dst []byte, url string, postArgs *Args) (statusCode int, body []byte, err error)
Post函数用于发送携带给定POST参数的POST请求到给定url。
响应的body主体将会被附加到dst字节切片中并被返回。
该函数跟随重定向。如需手动处理重定向，请使用Do*函数。
若dst目的字节切片为nil，则新body主体缓冲将被分配。
若postArgs为nil，空POST的body主体将会被发送。

func ReleaseArgs(a *Args)
ReleaseArgs函数经过AquireArgs返回对象的捕捉到池中。
不要访问释放的Args对象，否则将可能发生数据竞争。

func ReleaseByteBuffer(b *ByteBuffer)
ReleaseByteBUffer函数用于返回字节缓冲到池中。
不得接触返回到池中后的ByteBuffer.B，否则数据竞争将会发生。

func ReleaseCookie(c *Cookie)
ReleaseCookie函数经过AcquireCookie返回Cookie对象的捕捉到池中。
不要访问释放的Cookie对象，否则可能发生数据竞争。

func ReleaseRequest(req *Request)
ReleaseRequest函数经过AcquireRequest返回req的捕捉到请求池中。
在返回请求到请求池中后，禁止访问req及其成员。

func ReleaseResponse(resp *Response)
ReleaseResponse函数经过AcquireResponse返回resp的捕捉到响应池中。
在返回响应到响应池中后，禁止访问resp及其成员。

func ReleaseURI(u *URI)
ReleaseURI函数经过AcquireURI释放URI。
不得在释放URI后使用释放的URI，否则可能发送数据竞争。

func SaveMultipartFile(fh *multipart.FileHeader, path string) error
SaveMultipartFile函数用于在给定的文件路径下保存多段文件fh。

func Serve(ln net.Listener, handler RequestHandler) error
Serve函数用于从给定的监听ln中传入给定handler。
服务将阻塞直到给定监听返回永久性错误。

func ServeConn(c net.Conn, handler RequestHandler) 
ServeConn函数利用给定handler从给定连接中服务HTTP请求。
若来此c的所有请求都被成功服务，ServeConn函数将返回nil。否则返回非nil错误。
连接c必须立即向Write函数发送所有已通过的数据至客户端。否则请求的处理可能被挂起。
ServeConn函数在返回前需关闭连接c。

func ServeFile(ctx *RequestCtx, path string)
ServeFile函数用于返回包含在给定路径的压缩文件内容的HTTP响应。
HTTP响应可以在下列情况下，包含非压缩的内容：
*缺少“Accept-Encoding: gzip”请求头。
*没有该文件目录的写权限。
若路径指向目录，则目录内容将被返回。
使用ServeFileUncompressed时，无需提供压缩文件内容。
详情请参见RequestCtx.SendFile。

func ServeFileBytes(ctx *RequestCtx, path []byte)
ServeFileBytes函数用于返回给定路径下包含压缩文件内容的HTTP响应。
HTTP响应可以在下列情况下，包含非压缩的内容：
*缺少“Accept-Encoding: gzip”请求头。
*没有该文件目录的写权限。
若路径指向目录，则目录内容将被返回。
使用ServeFileUncompressed时，无需提供压缩文件内容。
详情请参见RequestCtx.SendFileBytes。

func ServeFileBytesUncompressed(ctx *RequestCtx, path []byte)
ServeFileBytesUncompressed函数用于返回给定路径下包含文件内容的HTTP响应。
若路径指向目录，则目录内容被返回。
当服务文件具有良好的压缩比时，ServeFileBytes可节约网络流量。
详情请参见RequestCtx.SendFileBytes。

func ServeFileUncompressed(ctx *RequestCtx, path string)
ServeFileUncompressed函数用于返回给定路径下包含文件内容的HTTP响应。
若路径指向目录，则目录内容被返回。
当服务文件具有良好的压缩比时，ServeFile可节约网络流量。
详情请参见RequestCtx.SendFile。

func ServeTLS(ln net.Listener, certFile, keyFile string, handler RequestHandler) error
ServeTLS函数利用给定handler服务给定net.Listener中HTTPS请求。
certFile、keyFile分别为到TLS证书及密钥文件的路径。

func ServeTLSEmbed(ln net.Listener, certData, keyData []byte, handler RequestHandler) error
ServeTLSEmbed函数利用给定handler服务给定net.Listner中的HTTS请求。
certData与keyData必需包含有效的TLS证书及密钥数据。

func StatusMessage(statusCode int) string
StatusMessage函数用于返回给定状态码的HTTP状态信息。

func WriteGunzip(w io.Writer, p []byte) (int, error)
WriteGunzip函数用于写入未压缩的字节切片p至写入器w中，并返回写入到w中的非压缩的字节数及错误信息。

func WriteGzip(w io.Writer, p []byte) (int, error)
WriteGzip函数用于写入gzip压缩的字节切片p至写入器w中，并返回写入到w中的压缩过的字节数及错误信息。

func WriteGzipLevel(w io.Writer, p []byte, level int) (int, error)
WriteGzipLevel函数使用给定压缩级别写入gzip压缩的字节切片p至写入器w中，并返回写入到w中的压缩过的字节数及错误信息。
支持的压缩级别：
* compressnocompression	未压缩
* compressbestspeed	最快速度
* compressbestcompression	最大压缩比
* compressdefaultcompression	默认压缩级别

func WriteInflate(w io.Writer, p []byte) (int, error)
WriteInflate函数用于inflate方式写字节切片p至写入器w中，并返回写入到w中非压缩的字节数及错误信息。

func WriteMultipartForm(w io.Writer, f *multipart.Form, boundary string) error
WriteMultipartForm函数用于将给定的多部分表格f的给定边界boundary中的内容写入至写入器w。

type Args

type Args Struct {
　　// 包含过滤掉的或未导出的内容
}
Args代表查询参数。
严禁复制Args实例。请新建新实例代替并使用CopyTo函数。
Args实例不得从并发运行的Go协程中使用。

func AcquireArgs() *Args
AcquireArgs函数用于从池中返回一个空的Args对象。
当不再需要时，返回的Args可能会通过ReleaseArgs被返回到池中。这将会降低GC负载。

func (a *Args) Add(key, value string)
Add函数用于添加“key=value”键值对参数。
同一个键可允许添加多个值。

func (a *Args) AddBytesK(key []byte, value string)
AddBytesK函数用于添加“key=value”参数。
同一个键可允许添加多个值。

func (a *Args) AddBytesKV(key, value []byte)
AddBytesKV函数用于添加“key=value”参数。
同一个键可允许添加多个值。

func (a *Args) AddBytesV(key string, value []byte)
AddBytesV函数用于添加“key=value”参数。
同一个键可允许添加多个值。

func (a *Args) AppendBytes(dst []byte) []byte
AppendBytes函数用于附加查询字符串到dst目的字节切片，并返回扩展后的dst目的字符切片结果。

func (a *Args) CopyTo(dst *Args)
CopyTo函数用于赋值所有args到dst目的字符切片中。

func (a *Args) Del(key string)
Del函数通过给定键key从查询参数args中删除参数。

func (a *Args) DelBytes(key []byte)
DelBytes函数通过给定键key从查询参数args中删除参数。

func (a *Args) GetUfloat(key string) (float64, error)
GetUfloat函数通过给定键key返回无符号浮点数。

func (a *Args) GetUfloatOrZero(key string) float64
GetUfloatOrZero函数通过给定键key返回无符号浮点数。
当发生错误时，零值被返回。

func (a *Args) GetUint(key string) (int, error)
GetUint函数通过给定键key返回无符号整数。

func (a *Args) GetUintOrZero(key string) int
GetUintOrZero函数通过给定键key返回无符号整数。
当发生错误时，零值被返回。

func (a *Args) Has(key string) bool
若给定键key存在于Args中，则Has函数返回ture。

func (a *Args) HasBytes(key []byte) bool
若给定键key存在于Args中，则HasBytes函数返回true。

func (a *Args) Len() int
Len函数用于返回查询参数args的数量。

func (a *Args) Parse(s string)
Parse函数用于解析包含查询参数的给定字符串string。

func (a *Args) ParseBytes(b []byte)
ParseBytes函数用于解析包含查询参数的给定字节切片b。

func (a *Args) Peek(key string) []byte
Peek函数用于返回给定键key的产讯参数值。
返回值在下一次调用Args前有效。

func (a *Args) PeekBytes(key []byte) []byte
PeekBytes函数用于返回给定键key的查询参数值。
返回值在下一次调用Args前有效。

func (a *Args) PeekMulti(key string) [][]byte
PeekMulti函数用于返回给定键key的所有参数值。

func (a *Args) PeekMultiBytes(key []byte) [][]byte
PeekMultiBytes函数用于返回给定键key的所有参数值。

func (a *Args) QueryString() []byte
QueryString函数用于返回args的查询字符串。
返回值在下一次调用Args方法前有效。

func (a *Args) Reset()
Reset函数用于情况查询参数args。

func (a *Args) Set(key, value string)
Set函数用于设置“key=value”键值对参数。

func (a *Args) SetBytesK(key []byte, value string)
SetBytesK函数用于设置“key=value”键值对参数。

func (a *Args) SetBytesKV(key, value []byte)
SetBytesKV函数用于设置“key=value”键值对参数。

func (a *Args) SetBytesV(key string, value []byte)
SetBytesV函数用于设置“key=value”键值对参数。

func (a *Args) SetUint(key string, value int)
SetUint函数通过给定键key设置无符号整数值。

func (a *Args) SetUintBytes(key []byte, value int)
SetUintBytes函数通过给定键key设置无符号整数值。

func (a *Args) String() string
String函数用于返回查询参数args的字符串表示。

func (a *Args) VisitAll(f func(key, value []byte))
VisitAll函数会对于每一个存在的arg参数调用函数f。
f函数不得在返回后保留键、值的引用。如果你需要在返回后存储他们，请创建键、值的副本。

func (a *Args) WriteTo(w io.Writer) (int64, error)
WriteTo函数用于吸入查询字符串到写入器w中。
WriteTo实现了io.WriteTo接口。

type BalancingClient

type BalancingClient interface {
DoDeadline(req *Request, resp *Response, deadline time.Time) error
PendingRequests() int
}
BalancingClient是一个客户端接口，其可被传递给LBClient.Client。

type ByteBuffer

type ByteBuffer bytebufferpool.ByteBuffer
ByteBuffer提供字节缓冲器，其可以于fasthttp API中以减少内存分配。
使用AcquireByteBuffer可获得一个空的字节缓冲器。
不推荐使用ByteBuffer。请使用 github.com/valyala/bytebufferpool 替代。
例程：

// 该请求handler用于设置“Your-IP”你的IP响应头
// “Your IP is <ip>”信息利用ByteBuffer构建响应。
// header将分配为零值。
yourIPRequestHandler := func(ctx *fasthttp.RequestCtx) {
    b := fasthttp.AcquireByteBuffer()
    b.B = append(b.B, "Your IP is <"...)
    b.B = fasthttp.AppendIPv4(b.B, ctx.RemoteIP())
    b.B = append(b.B, ">"...)
    ctx.Response.Header.SetBytesV("Your-IP", b.B)
 
    fmt.Fprintf(ctx, "Check response headers - they must contain 'Your-IP: %s'", b.B)
 
    // 现在便可以安全地释放字节缓冲器了。
    // 不再使用。
    fasthttp.ReleaseByteBuffer(b)
}
 
// 启动fasthttp服务，并在响应头中返回你的IP。
fasthttp.ListenAndServe(":8080", yourIPRequestHandler)
func AcquireByteBuffer() *ByteBuffer
AcquireByteBuffer用于从池中返回一个空的字节缓冲器。
获取字节缓冲器可能会通过调用ReleaseByteBuffer函数而被返回到池中。这会减少字节缓冲器管理所需的内存分配数量。

func (b *ByteBuffer) Reset()
Reset函数用于清空字节缓冲器ByteBuffer.B。

func (b *ByteBuffer) Set(p []byte)
Set函数用于设置字节缓冲器ByteBuffer.B至字节切片p中。

func (b *ByteBuffer) SetString(s string)
SetString函数用于设置字节缓冲器ByteBuffer.B至字符串s中。

func (b *ByteBuffer) Write(p []byte) (int, error)
Write函数用于实现io.Writer写入器，其用于附加字节切片p至字节缓冲器ByteBuffer.B中。

func (b *ByteBuffer) WriteString(s string) (int, error)
WriteString函数用于附加字符串s至字节缓冲器ByteBuffer.B中。
