# flag - 命令行参数解析
在写命令行程序（工具、server）时，对命令参数进行解析是常见的需求。各种语言一般都会提供解析命令行参数的方法或库，以方便程序员使用。如果命令行参数纯粹自己写代码解析，对于比较复杂的，还是挺费劲的。在 go 标准库中提供了一个包：flag，方便进行命令行解析。

注：区分几个概念

命令行参数（或参数）：是指运行程序提供的参数
已定义命令行参数：是指程序中通过flag.Xxx等这种形式定义了的参数
非flag（non-flag）命令行参数（或保留的命令行参数）：后文解释

## 1.2. flag 包概述
flag 包实现了命令行参数的解析。

### 1.2.1. 定义 flags 有两种方式
1）flag.Xxx()，其中 Xxx 可以是 Int、String 等；返回一个相应类型的指针，如：

var ip = flag.Int("flagname", 1234, "help message for flagname")
2）flag.XxxVar()，将 flag 绑定到一个变量上，如：

var flagvar int
flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
### 1.2.2. 自定义 Value
另外，还可以创建自定义 flag，只要实现 flag.Value 接口即可（要求 receiver 是指针），这时候可以通过如下方式定义该 flag：

flag.Var(&flagVal, "name", "help message for flagname")
例如，解析我喜欢的编程语言，我们希望直接解析到 slice 中，我们可以定义如下 Value：
```
type sliceValue []string

func newSliceValue(vals []string, p *[]string) *sliceValue {
    *p = vals
    return (*sliceValue)(p)
}

func (s *sliceValue) Set(val string) error {
    *s = sliceValue(strings.Split(val, ","))
    return nil
}

func (s *sliceValue) Get() interface{} { return []string(*s) }

func (s *sliceValue) String() string { return strings.Join([]string(*s), ",") }
```
之后可以这么使用：

var languages []string
flag.Var(newSliceValue([]string{}, &languages), "slice", "I like programming `languages`")
这样通过 -slice "go,php" 这样的形式传递参数，languages 得到的就是 [go, php]。

flag 中对 Duration 这种非基本类型的支持，使用的就是类似这样的方式。

### 1.2.3. 解析 flag
在所有的 flag 定义完成之后，可以通过调用 flag.Parse() 进行解析。

命令行 flag 的语法有如下三种形式：

-flag // 只支持bool类型
-flag=x
-flag x // 只支持非bool类型
其中第三种形式只能用于非 bool 类型的 flag，原因是：如果支持，那么对于这样的命令 cmd -x *，如果有一个文件名字是：0或false等，则命令的原意会改变（之所以这样，是因为 bool 类型支持 -flag 这种形式，如果 bool 类型不支持 -flag 这种形式，则 bool 类型可以和其他类型一样处理。也正因为这样，Parse()中，对 bool 类型进行了特殊处理）。默认的，提供了 -flag，则对应的值为 true，否则为 flag.Bool/BoolVar 中指定的默认值；如果希望显示设置为 false 则使用 -flag=false。

int 类型可以是十进制、十六进制、八进制甚至是负数；bool 类型可以是1, 0, t, f, true, false, TRUE, FALSE, True, False。Duration 可以接受任何 time.ParseDuration 能解析的类型。

## 1.3. 类型和函数
在看类型和函数之前，先看一下变量。

ErrHelp：该错误类型用于当命令行指定了 ·-help 参数但没有定义时。

Usage：这是一个函数，用于输出所有定义了的命令行参数和帮助信息（usage message）。一般，当命令行参数解析出错时，该函数会被调用。我们可以指定自己的 Usage 函数，即：flag.Usage = func(){}

### 1.3.1. 函数
go标准库中，经常这么做：

定义了一个类型，提供了很多方法；为了方便使用，会实例化一个该类型的实例（通用），这样便可以直接使用该实例调用方法。比如：encoding/base64 中提供了 StdEncoding 和 URLEncoding 实例，使用时：base64.StdEncoding.Encode()

在 flag 包使用了有类似的方法，比如 CommandLine 实例，只不过 flag 进行了进一步封装：将 FlagSet 的方法都重新定义了一遍，也就是提供了一序列函数，而函数中只是简单的调用已经实例化好了的 FlagSet 实例：CommandLine 的方法。这样，使用者是这么调用：flag.Parse() 而不是 flag. CommandLine.Parse()。（Go 1.2 起，将 CommandLine 导出，之前是非导出的）

这里不详细介绍各个函数，在类型方法中介绍。

### 1.3.2. 类型（数据结构）
1）ErrorHandling

type ErrorHandling int
该类型定义了在参数解析出错时错误处理方式。定义了三个该类型的常量：
```
const (
    ContinueOnError ErrorHandling = iota
    ExitOnError
    PanicOnError
)
```
三个常量在源码的 FlagSet 的方法 parseOne() 中使用了。

2）Flag

// A Flag represents the state of a flag.
type Flag struct {
    Name     string // name as it appears on command line
    Usage    string // help message
    Value    Value  // value as set
    DefValue string // default value (as text); for usage message
}
Flag 类型代表一个 flag 的状态。

比如，对于命令：./nginx -c /etc/nginx.conf，相应代码是：

flag.StringVar(&c, "c", "conf/nginx.conf", "set configuration `file`")
则该 Flag 实例（可以通过 flag.Lookup("c") 获得）相应各个字段的值为：

&Flag{
    Name: c,
    Usage: set configuration file,
    Value: /etc/nginx.conf,
    DefValue: conf/nginx.conf,
}
3）FlagSet

// A FlagSet represents a set of defined flags.
type FlagSet struct {
    // Usage is the function called when an error occurs while parsing flags.
    // The field is a function (not a method) that may be changed to point to
    // a custom error handler.
    Usage func()

    name string // FlagSet的名字。CommandLine 给的是 os.Args[0]
    parsed bool // 是否执行过Parse()
    actual map[string]*Flag // 存放实际传递了的参数（即命令行参数）
    formal map[string]*Flag // 存放所有已定义命令行参数
    args []string // arguments after flags // 开始存放所有参数，最后保留 非flag（non-flag）参数
    exitOnError bool // does the program exit if there's an error?
    errorHandling ErrorHandling // 当解析出错时，处理错误的方式
    output io.Writer // nil means stderr; use out() accessor
}
4）Value 接口

// Value is the interface to the dynamic value stored in a flag.
// (The default value is represented as a string.)
type Value interface {
    String() string
    Set(string) error
}
所有参数类型需要实现 Value 接口，flag 包中，为int、float、bool等实现了该接口。借助该接口，我们可以自定义flag。（上文已经给了具体的例子）

## 1.4. 主要类型的方法（包括类型实例化）
flag 包中主要是 FlagSet 类型。

### 1.4.1. 实例化方式
NewFlagSet() 用于实例化 FlagSet。预定义的 FlagSet 实例 CommandLine 的定义方式：

// The default set of command-line flags, parsed from os.Args.
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
可见，默认的 FlagSet 实例在解析出错时会退出程序。

由于 FlagSet 中的字段没有 export，其他方式获得 FlagSet实例后，比如：FlagSet{} 或 new(FlagSet)，应该调用Init() 方法，以初始化 name 和 errorHandling，否则 name 为空，errorHandling 为 ContinueOnError。

### 1.4.2. 定义 flag 参数的方法
这一序列的方法都有两种形式，在一开始已经说了两种方式的区别。这些方法用于定义某一类型的 flag 参数。

### 1.4.3. 解析参数（Parse）
func (f *FlagSet) Parse(arguments []string) error
从参数列表中解析定义的 flag。方法参数 arguments 不包括命令名，即应该是os.Args[1:]。事实上，flag.Parse() 函数就是这么做的：

// Parse parses the command-line flags from os.Args[1:].  Must be called
// after all flags are defined and before flags are accessed by the program.
func Parse() {
    // Ignore errors; CommandLine is set for ExitOnError.
    CommandLine.Parse(os.Args[1:])
}
该方法应该在 flag 参数定义后而具体参数值被访问前调用。

如果提供了 -help 参数（命令中给了）但没有定义（代码中没有），该方法返回 ErrHelp 错误。默认的 CommandLine，在 Parse 出错时会退出程序（ExitOnError）。

为了更深入的理解，我们看一下 Parse(arguments []string) 的源码：
```
func (f *FlagSet) Parse(arguments []string) error {
    f.parsed = true
    f.args = arguments
    for {
        seen, err := f.parseOne()
        if seen {
            continue
        }
        if err == nil {
            break
        }
        switch f.errorHandling {
        case ContinueOnError:
            return err
        case ExitOnError:
            os.Exit(2)
        case PanicOnError:
            panic(err)
        }
    }
    return nil
}
```
真正解析参数的方法是非导出方法 parseOne。

结合 parseOne 方法，我们来解释 non-flag 以及包文档中的这句话：

Flag parsing stops just before the first non-flag argument ("-" is a non-flag argument) or after the terminator "--".

我们需要了解解析什么时候停止。

根据 Parse() 中 for 循环终止的条件（不考虑解析出错），我们知道，当 parseOne 返回 false, nil 时，Parse 解析终止。正常解析完成我们不考虑。看一下 parseOne 的源码发现，有两处会返回 false, nil。

1）第一个 non-flag 参数

s := f.args[0]
if len(s) == 0 || s[0] != '-' || len(s) == 1 {
    return false, nil
}
也就是，当遇到单独的一个"-"或不是"-"开始时，会停止解析。比如：

./nginx - -c 或 ./nginx build -c

这两种情况，-c 都不会被正确解析。像该例子中的"-"或build（以及之后的参数），我们称之为 non-flag 参数。

2）两个连续的"--"

if s[1] == '-' {
    num_minuses++
    if len(s) == 2 { // "--" terminates the flags
        f.args = f.args[1:]
        return false, nil
    }
}
也就是，当遇到连续的两个"-"时，解析停止。

说明：这里说的"-"和"--"，位置和"-c"这种的一样。也就是说，下面这种情况并不是这里说的：

./nginx -c --

这里的"--"会被当成是 c 的值

parseOne 方法中接下来是处理 -flag=x 这种形式，然后是 -flag 这种形式（bool类型）（这里对bool进行了特殊处理），接着是 -flag x 这种形式，最后，将解析成功的 Flag 实例存入 FlagSet 的 actual map 中。

另外，在 parseOne 中有这么一句：

f.args = f.args[1:]
也就是说，每执行成功一次 parseOne，f.args 会少一个。所以，FlagSet 中的 args 最后留下来的就是所有 non-flag 参数。

### 1.4.4. Arg(i int) 和 Args()、NArg()、NFlag()
Arg(i int) 和 Args() 这两个方法就是获取 non-flag 参数的；NArg()获得 non-flag 的个数；NFlag() 获得 FlagSet 中 actual 长度（即被设置了的参数个数）。

### 1.4.5. Visit/VisitAll
这两个函数分别用于访问 FlatSet 的 actual 和 formal 中的 Flag，而具体的访问方式由调用者决定。

### 1.4.6. PrintDefaults()
打印所有已定义参数的默认值（调用 VisitAll 实现），默认输出到标准错误，除非指定了 FlagSet 的 output（通过SetOutput() 设置）

### 1.4.7. Set(name, value string)
设置某个 flag 的值（通过 name 查找到对应的 Flag）

