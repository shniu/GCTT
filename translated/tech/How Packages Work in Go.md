
# Go 中的包是如何工作的

自从我开始在 Go 中编写代码以来，对于我来说，如何最好地组织我的代码并使用 package 关键字一直是个谜。package 关键字与在 C＃ 中使用名称空间类似，但是约定是将包名称与目录结构绑定。

Go 有一个 web 页面试图去解释如何编写 Go 代码。
http://golang.org/doc/code.html

当我开始用 Go 编程时，这是我读到的文档之一。这让我头脑发热，主要是因为我一直在 Visual Studio 中工作，并且在 Solution 和 Project 文件中打包代码。 在文件系统上运行一个目录是一个疯狂的想法。 现在我喜欢它的简单性，但它花了相当长的一段时间才有意义。

“如何编写 Go 代码” 以工作区的概念开始。把它想象成你的项目的根目录。如果你在 Visual Studio 中工作，则这是 solution 或 project 文件所在的位置。然后从工作区内部创建一个名为 src 的子目录。如果你希望 Go 工具正常工作，这是必需的。从 src 目录中，你可以按照自己的想法自由组织代码。但是，您需要了解 Go 团队为软件包和源代码提出的约定，否则您可能会重构代码。

在我的机器上，我创建了一个名为 Test 的工作区以及名为 src 的必需子目录。 这是创建项目的第一步。

![go package img](https://github.com/shniu/resources/raw/master/images/go-packages-step1.png)

然后在 LiteIDE 中打开 Test 目录（工作区），并创建以下子目录并清空 Go 源代码文件。

![go package img](https://github.com/shniu/resources/raw/master/images/go-packages-step2.png)

首先，我们为我们正在创建的应用程序创建一个子目录。main 函数所在目录的名称将是可执行文件的名称。在我们的例子中 main.go 包含 main 函数，位于 myprogram 目录下。这意味着我们的可执行文件将被称为 myprogram。

src 中的其他子目录将包含我们项目的包。按照惯例，该目录的名称应该是位于该目录中的那些源代码文件的包名称。在我们的例子中，新包被称为 samplepkg 和 subpkg。源代码文件的名称可以是任何你喜欢的。

接下来创建相同的包文件夹并清空 Go 源代码文件。

如果你没有将 Workspace 文件夹添加到 GOPATH 中，我们将遇到问题。

我花了一点才意识到 Custom Directories 窗口是一个文本框。所以你可以直接编辑这些文件夹。系统 GOPATH 是只读的。
![img3](https://github.com/shniu/resources/raw/master/images/go-packages-step3.png)

Go 设计人员在命名软件包和源代码文件时做了几件事。所有文件名和目录都是小写字母，并且它们没有使用下划线在包目录名称中分开单词。此外，包名称与目录名称相匹配。目录内的代码文件属于以该目录命名的包。

查看几个标准库软件包的Go源代码目录：

![img4](https://github.com/shniu/resources/raw/master/images/go-packages-step4.png)

bufio 和 builtin 的包目录是目录命名约定的很好的例子。他们可能被称为 `buf_io` 和 `built_in`。

再次查看 Go 源代码目录并查看源代码文件的名称。

![img5](https://github.com/shniu/resources/raw/master/images/go-packages-step5.png)

注意在一些文件名中使用下划线。当文件包含测试代码或特定于特定平台时，使用下划线。

通常的惯例是将其中一个源代码文件命名为与包名称相同。在 bufio 中，遵循这个惯例。但是，这是一个松散遵循的惯例。

![img6](https://github.com/shniu/resources/raw/master/images/go-packages-step6.png)

在 fmt 包中，你会注意到没有名为 fmt.go 的源代码文件。我个人喜欢用不同的方式命名我的包和源代码文件。

最后，打开 doc.go，format.go，print.go 和 scan.go 文件。他们都被宣布为在fmt包中。

让我们来看一下 sample.go 的代码：

```go
package samplepkg

import (
    "fmt"
)

type Sample struct {
    Name string
}

func New(name string) (sample * Sample) {
    return &Sample{
        Name: name,
    }
}

func (sample * Sample) Print() {
    fmt.Println("Sample Name:", sample.Name)
}
```

代码是没用的，但它会让我们专注于两个重要的惯例。首先，请注意包的名称与子目录的名称相同。其次，有一个叫 New 的函数。

New 函数是创建核心类型或不同类型以供应用程序开发人员使用的包的 Go 约定。看看 New 是如何在 log.go，bufio.go 和 cypto.go 中定义和实现的：

```go
log.go
// New creates a new Logger. The out variable sets the
// destination to which log data will be written.
// The prefix appears at the beginning of each generated log line.
// The flag argument defines the logging properties.
func New(out io.Writer, prefix string, flag int) * Logger {
    return &Logger{out: out, prefix: prefix, flag: flag}
}

bufio.go
// NewReader returns a new Reader whose buffer has the default size.
func NewReader(rd io.Reader) * Reader {
    return NewReaderSize(rd, defaultBufSize)
}

crypto.go
// New returns a new hash.Hash calculating the given hash function. New panics
// if the hash function is not linked into the binary.
func (h Hash) New() hash.Hash {
    if h > 0 && h < maxHash {
        f := hashes[h]
        if f != nil {
            return f()
        }
    }
    panic("crypto: requested hash function is unavailable")
}
```

由于每个包都充当命名空间，因此每个包都可以拥有自己的 New 版本。在 bufio.go 中可以创建多个类型，所以没有独立的 New 函数。在这里你可以找到像 NewReader 和 NewWriter 这样的函数。

回顾一下 sample.go。在我们的代码中，核心类型是 Sample，所以我们的 New 函数返回对 Sample 类型的引用。然后我们添加了一个成员函数来显示我们在 New 中提供的名称。

让我们来看一下 sub.go 的代码：

```go
package subpkg

import (
    "fmt"
)

type Sub struct {
    Name string
}

func New(name string) (sub * Sub) {
    return &Sub{
        Name: name,
    }
}

func (sub * Sub) Print() {
    fmt.Println("Sub Name:", sub.Name)
}
```




----------------

via: https://www.ardanlabs.com/blog/2013/07/how-packages-work-in-go-language.html

作者：[William Kennedy](https://github.com/ardanlabs/gotraining)
译者：[shniu](https://github.com/shniu)
校对：[polaris1119](https://github.com/polaris1119)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出

