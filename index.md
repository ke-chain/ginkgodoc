---
layout: default
title: Ginkgo 中文手册
---
[Ginkgo](https://github.com/onsi/ginkgo)  是一个 Go 测试框架，旨在帮助你有效地编写富有表现力的全方位测试。

与它最匹配的库是 [Gomega](https://github.com/onsi/gomega) ，但它的设计是与匹配器无关的。

这个文档假设你使用 Gomega 的 Ginkgo 。同时也假设你知道 Go 的使用方式并且对于 **$GOPATH** 目录下 Go 如何组织包有一个好的思维模型。

---

## 版本支持策略

Ginkgo 提供支持 Go 的版本记录在  [Go 发布策略](https://golang.org/doc/devel/release.html#policy) 比如 N 和 N-1 主版本。

---

## 获取 Ginkgo

使用  `go get` 获取:

    $ go get github.com/onsi/ginkgo/ginkgo
    $ go get github.com/onsi/gomega/...

该命令获取 ginkgo 并且安装  `ginkgo` 可执行文件到  `$GOPATH/bin` – 你需要在你的`$PATH`上配置它。

**Ginkgo 要求 Go 版本在 v1.6或以上** 。
要安装 Go ，请遵从 [安装手册](https://golang.org/doc/install) 。

以上命令同时安装了全部 gomega 库。如果你只想安装测试需要的包，导入你需要的包然后使用 `go get -t`。

例如，导入 gomega 包到你的测试代码：

    import "github.com/onsi/gomega"

使用 `go get -t  ` 取回你测试代码中引用的包：

    $ cd /path/to/my/app
    $ go get -t ./...

---

##  入门: 第一个测试
Ginkgo与Go现有的测试基础设施挂钩. 这允许您使用 `go test` 运行Ginkgo套件.

> 这同时意味着 Ginkgo 测试可以和传统 Go `testing`  测试一起使用。`go test` 和 `ginkgo` 都会运行你套件内的所有测试。

### 引导套件
要为一个包写 Ginkgo 测试的话你必须首先初始化一个 Ginkgo 测试套件。比如说你有一个名为 `books` 的包：

    $ cd path/to/books
    $ ginkgo bootstrap

我们将生成一个名为 `books_suite_test.go` 的文件并包含：

```go
package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
    "testing"
)

func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail)
    RunSpecs(t, "Books Suite")
}
```

我们来分析上述代码：

- Go 允许我们在  `books` 包中声明 `books_test` 包。使用 `books_test` 替代 `books` 允许我们遵守 `books` 包的封装性：你的测试将要导入 `books` 并且从外部使用它，就像其他包一样。当让，如果你想要进入包内部来测试它内部组件并进行跟多行为测试的话，你可以选择将 `package books_test`  换成 `package books` 。
- 我们使用 dot-import 将 `ginkgo` 和 `gomega` 包导入到了顶级命名空间。如果你不想这样做的话，查看下面的 [避免 Dot Imports](#avoiding-dot-imports) 。
- `TestBooks`  是一个 `testing` 测试.你运行 `go test`  或 `ginkgo` 的时候 Go 测试执行器会执行这个函数。
- `RegisterFailHandler(Fail)` ： 一个 Ginkgo 测试调用 Ginkgo 的  `Fail(description string)` 函数发出失败信号。我们使用`RegisterFailHandler` 将这个函数传给 Gomega 。这是 Ginkgo 和 Gomega 唯一的连接点。
- `RunSpecs(t *testing.T, suiteDescription string)` 告诉 Ginkgo 开始这个测试套件。如果任意 specs（说明）失败了，Ginkgo 会自动使 `testing.T` 失败。 

现在你可以运行你的套件：

    $ ginkgo #or go test

    === RUN TestBootstrap

    Running Suite: Books Suite
    ==========================
    Random Seed: 1378936983

    Will run 0 of 0 specs


    Ran 0 of 0 Specs in 0.000 seconds
    SUCCESS! -- 0 Passed | 0 Failed | 0 Pending | 0 Skipped

    --- PASS: TestBootstrap (0.00 seconds)
    PASS
    ok      books   0.019s

### 添加 Specs 到你的套件

一个空的测试套件不是非常有趣。在你可以开始添加测试到 `books_suite_test.go` 的时候，你很可能偏向把测试放在多个文件中 （特别是有多个文件的包）。我们添加一个测试文件到我们的 `book.go `模型：

    $ ginkgo generate book

这将生成一个名为 `book_test.go` 的文件并包含：

```go
package books_test

import (
    "/path/to/books"
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

var _ = Describe("Book", func() {

})
```
我们来分析上述代码：

- 我们 将 `ginkgo` 和 `gomega` 包导入到顶级命名空间。这样非常方便的同时，也是不必要的。如果你不想这样做的话，查看下面的 [避免 Dot Imports](https://ke-chain.github.io/ginkgodoc/#avoiding-dot-imports) 。
- 类似的，我们导入 `books` 包因为我们使用特别的 `books_test` 包名用于将测试和代码隔离。方便起见我们导入了 `books` 到命名空间。你可以通过编辑生成的测试文件来选择其它方式。
- 我们使用 Ginkgo 的 `Describe(text string, body func()) bool` 函数添加了*顶层*描述容器。`var _ = ...` 函数允许我们在顶层给 Describe 赋值 并且不用将它包含在 `func init() {}` 当中。

这个在 `Describe` 函数将包含我们的 specs 。 我们添加一点来从 JSON 加载 books ：

```go
var _ = Describe("Book", func() {
    var (
        longBook  Book
        shortBook Book
    )

    BeforeEach(func() {
        longBook = Book{
            Title:  "Les Miserables",
            Author: "Victor Hugo",
            Pages:  1488,
        }

        shortBook = Book{
            Title:  "Fox In Socks",
            Author: "Dr. Seuss",
            Pages:  24,
        }
    })

    Describe("Categorizing book length", func() {
        Context("With more than 300 pages", func() {
            It("should be a novel", func() {
                Expect(longBook.CategoryByLength()).To(Equal("NOVEL"))
            })
        })

        Context("With fewer than 300 pages", func() {
            It("should be a short story", func() {
                Expect(shortBook.CategoryByLength()).To(Equal("SHORT STORY"))
            })
        })
    })
})
```

我们来分析上述代码：

- Ginkgo 使用了大量的闭包使得你可以构建可描述的测试套件。
- 你应该使用 `Describe` 和 `Context` 容器来富有表现力地组织你代码的行为。
- 你可使用 `BeforeEach` 为你的 specs 初始化状态。使用 `It` 指定一个 spec。
- 要在 `BeforeEach` 和 `It` 分享状态的话你可以使用闭包变量，一般声明在 `Describe` 和 `Context` 容器最近的顶层。
- 我们使用 Gomega 的 `Expect` 语法来对 `CategoryByLength()` 方法执行期望（expectations）。

假设一个 `Book` 模型有这些行为，运行这些测试将会成功：

    $ ginkgo # or go test
    === RUN TestBootstrap

    Running Suite: Books Suite
    ==========================
    Random Seed: 1378938274

    Will run 2 of 2 specs

    ••
    Ran 2 of 2 Specs in 0.000 seconds
    SUCCESS! -- 2 Passed | 0 Failed | 0 Pending | 0 Skipped

    --- PASS: TestBootstrap (0.00 seconds)
    PASS
    ok      books   0.025s

成功！

### 将Specs标记为失败

虽然您通常希望使用匹配库（如Gomega）在您的`Spec`中进行断言，但Ginkgo提供了一个简单的全局`Fail`函数，允许您将`Spec`标记为`Fail`。只需调用：

```go
Fail("Failure reason")
```

Ginkgo 将会处理其余的部分。

`Fail`(因此 Gomega ，因为它使用 `Fail `)将为当前的 `space`  记录为失败并且 `panic`。这允许Ginkgo停止其轨道中的当前Spec - 没有后续的断言（或者任何那个事件的代码）将被调用。通常情况下，Ginkgo将会补救这个`Panic`本身然后进行下一步测试。

然而，如果你的测试启用了`goroutine`调用`Fail`（或者，等效地，调用失败的 Gomega 断言），Ginkgo 将没有办法补救由 `Fail` 引发的 `Panic.` 这将导致测试套件出现 `Panic `，并且不会运行后续测试。要解决这个问题，你必须使用 `GinkgoRecover `拯救`Panic`。这是一个例子：

```go
It("panics in a goroutine", func(done Done) {
    go func() {
        defer GinkgoRecover()

        Ω(doSomething()).Should(BeTrue())

        close(done)
    }()
})
```

现在，如果`doSomething`返回`false`，Gomega将会调用`Fail`,这将会引起`Panic`，但是`defer`的`GinkgoRecover()`将恢复所述`Panic`并防止测试套件爆炸。

有关Fail以及使用除Gomega之外的匹配器库的更多详细信息，请参阅[使用其他匹配库](https://ke-chain.github.io/ginkgodoc/#使用其他匹配库)部分

## 日志输出

Ginkgo提供了一个全局可用的`io.Writer`，名为`GinkgoWriter`，供您写入。`GinkgoWriter`在测试运行时聚合输入，并且只有在测试失败时才将其转储到`stdout`。当以详细模式运行时（`ginkgo -v`或`go test -ginkgo.v`），`GinkgoWriter`会立即将其输入重定向到`stdout`。

当Ginkgo测试套件中断（通过^ C）时，Ginkgo将发出写入`GinkgoWriter`的任何内容。这样可以更轻松地调试卡住的测试。
当与`--progress`配对使用时将会特别有用，它指示Ginkgo在运行您的`BeforeEaches`，`Its`，`AfterEaches`等时向GinkgoWriter发出通知。
## IDE 支持

Ginkgo用命令行运行最佳，`ginkgo watch`可以在检测到变化时轻松地在命令行上重新运行测试。

[Sublime Text](http://www.sublimetext.com/) 有一组 [Completions](https://github.com/onsi/ginkgo-sublime-completions)（仅使用[Package Control](https://packagecontrol.io/)来安装 `Ginkgo Completions` ）和 [VSCode](https://code.visualstudio.com/)（使用扩展安装程序并安装vscode-ginkgo）。
IDE 作者可以将 `GINKGO_EDITOR_INTEGRATION `环境变量设置为任何非空值，使专注的`Spec`能够显示覆盖范围。默认情况下，如果确定关注的spec不通过CI, Ginkgo 将会Fail，使用非零退出码。

---

## 构建 Specs

Ginkgo可以轻松编写富有表现力的 specs，以有条理的方式描述代码的行为。您可以使用`Describe` 和 `Context`容器来组织你的`It` spec，使用 `BeforeEach` 和 `AfterEach` 来搭建和拆除测试中的常见设置。

### 单个 Specs: `It`
您可以通过在`Describe`或`Context`容器块中设置 It 块来添加单个 spec：

```go
var _ = Describe("Book", func() {
    It("can be loaded from JSON", func() {
        book := NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)

        Expect(book.Title).To(Equal("Les Miserables"))
        Expect(book.Author).To(Equal("Victor Hugo"))
        Expect(book.Pages).To(Equal(1488))
    })
})
```

> `It` 也可以放在顶层，虽然这种情况并不常见。

####  `Specify` 别名

为了确保您的 specs 阅读自然，`Specify`，`PSpecify`，`XSpecify`和`FSpecify`块可用作别名，以便在相应的`It`替代品看起来不像自然语言的情况下使用。

`Specify`块的行为与`It`块相同，可以在`It`块（以及`PIt`，`XIt`和`FIt`块）的地方使用。

`Specify`替换`It`的示范如下：

```go
Describe("The foobar service", func() {
  Context("when calling Foo()", func() {
    Context("when no ID is provided", func() {
      Specify("an ErrNoID error is returned", func() {
      })
    })
  })
})
```

### 提取通用步骤： `BeforeEach`

您可以使用`BeforeEach`块在多个测试用例中去除重复的步骤以及共享通用的设置：

```go
var _ = Describe("Book", func() {
    var book Book

    BeforeEach(func() {
        book = NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)
    })

    It("can be loaded from JSON", func() {
        Expect(book.Title).To(Equal("Les Miserables"))
        Expect(book.Author).To(Equal("Victor Hugo"))
        Expect(book.Pages).To(Equal(1488))
    })

    It("can extract the author's last name", func() {
        Expect(book.AuthorLastName()).To(Equal("Hugo"))
    })
})
```

`BeforeEach`在每个 spec 之前运行，从而确保每个 spec 都具有状态的原始副本。使用闭包变量共享公共状态（在本例中为`var book Book`）。您还可以在`AfterEach`块中执行清理操作。

在`BeforeEach`和`AfterEach`块中设置断言也很常见。例如，这些断言，可以断言在为 spec 准备状态时没有发生错误。
### 使用容器来组织 Specs : `Describe` and `Context`

Ginkgo允许您使用`Describe`和`Context`容器在套件中富有表现力的组织 specs ：
```go
var _ = Describe("Book", func() {
    var (
        book Book
        err error
    )

    BeforeEach(func() {
        book, err = NewBookFromJSON(`{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`)
    })

    Describe("loading from JSON", func() {
        Context("when the JSON parses succesfully", func() {
            It("should populate the fields correctly", func() {
                Expect(book.Title).To(Equal("Les Miserables"))
                Expect(book.Author).To(Equal("Victor Hugo"))
                Expect(book.Pages).To(Equal(1488))
            })

            It("should not error", func() {
                Expect(err).NotTo(HaveOccurred())
            })
        })

        Context("when the JSON fails to parse", func() {
            BeforeEach(func() {
                book, err = NewBookFromJSON(`{
                    "title":"Les Miserables",
                    "author":"Victor Hugo",
                    "pages":1488oops
                }`)
            })

            It("should return the zero-value for the book", func() {
                Expect(book).To(BeZero())
            })

            It("should error", func() {
                Expect(err).To(HaveOccurred())
            })
        })
    })

    Describe("Extracting the author's last name", func() {
        It("should correctly identify and return the last name", func() {
            Expect(book.AuthorLastName()).To(Equal("Hugo"))
        })
    })
})
```

您可以使用`Describe`块来描述代码的各个行为，`Context`块在不同情况下执行这些行为。在此示例中，我们`Describe`从JSON加载书籍并指定两个`Contexts`：当JSON成功解析时以及JSON无法解析时。除了语义差异，两种容器类型具有相同的行为。

当嵌套`Describe`和`Context`块时，`It`执行时，围绕`It`的所有容器节点的`BeforeEach`块，从最外层到最内层运行。

注意：每个`It`块都运行`BeforeEach`和`AfterEach`块。这确保了每个 spec的原始状态。


> In general, the only code within a container block should be an `It` block or a `BeforeEach`/`JustBeforeEach`/`JustAfterEach`/`AfterEach` block, or closure variable declarations.  It is generally a mistake to make an assertion in a container block.

> It is also a mistake to *initialize* a closure variable in a container block.  If one of your `It`s mutates that variable, subsequent `It`s will receive the mutated value.  This is a case of test pollution and can be hard to track down.  **Always initialize your variables in `BeforeEach` blocks.**

> 通常，容器块中的唯一代码应该是 `It` 块或 `BeforeEach` / `JustBeforeEach` / `JustAfterEach` / `AfterEach` 块或闭包变量声明。在容器块中进行断言通常是错误的。
> 在容器块中初始化闭包变量也是错误的。如果你的一个 `It` 改变了这个变量，后期 `It` 将会收到改变后的值。这是一个测试污染的案例，很难追查。**始终在**`BeforeEach`**块中初始化变量。**

如果您想在运行时获取有关当前测试的信息，您可以在任何`It`或`BeforeEach` / `JustBeforeEach`/`JustAfterEach` / `AfterEach`块中使用`CurrentGinkgoTestDescription()`。

此次调用 `CurrentGinkgoTestDescription` 返回包含有关当前运行的测试的各种信息，包括文件名，行号，`It`块中的文本以及周围容器块中的文本。

### 分离创建和配置: `JustBeforeEach`

上面的例子说明了BDD风格测试中常见的反模式。我们的顶级 `BeforeEach` 使用有效的 `JSON` 创建了一个新的 `book` ,但是较低级别的 `Context` 使用无效的JSON创建的 `book` 执行。这使我们重新创建并覆盖原始的 `book` 。幸运的是，使用Ginkgo的 `JustBeforeEach` 块，这些代码重复是不必要的。

`JustBeforeEach` 块保证在所有 `BeforeEach` 块运行之后，并且在 `It` 块运行之前运行。我们可以使用这个特性来清除 `Book` spec：

```go
var _ = Describe("Book", func() {
    var (
        book Book
        err error
        json string
    )

    BeforeEach(func() {
        json = `{
            "title":"Les Miserables",
            "author":"Victor Hugo",
            "pages":1488
        }`
    })

    JustBeforeEach(func() {
        book, err = NewBookFromJSON(json)
    })

    Describe("loading from JSON", func() {
        Context("when the JSON parses succesfully", func() {
            It("should populate the fields correctly", func() {
                Expect(book.Title).To(Equal("Les Miserables"))
                Expect(book.Author).To(Equal("Victor Hugo"))
                Expect(book.Pages).To(Equal(1488))
            })

            It("should not error", func() {
                Expect(err).NotTo(HaveOccurred())
            })
        })

        Context("when the JSON fails to parse", func() {
            BeforeEach(func() {
                json = `{
                    "title":"Les Miserables",
                    "author":"Victor Hugo",
                    "pages":1488oops
                }`
            })

            It("should return the zero-value for the book", func() {
                Expect(book).To(BeZero())
            })

            It("should error", func() {
                Expect(err).To(HaveOccurred())
            })
        })
    })

    Describe("Extracting the author's last name", func() {
        It("should correctly identify and return the last name", func() {
            Expect(book.AuthorLastName()).To(Equal("Hugo"))
        })
    })
})
```

现在，对每一个`It`，`book`实际上只创建一次。这个失败的`JSON`上下文可以简单地将无效的`json`值分配给`BeforeEach`中的`json`变量。

抽象地，`JustBeforeEach`允许您将**创建**与**配置**分离。使用由`BeforeEach`链指定和修改的配置在`JustBeforeEach`中进行创建。

> 您可以在不同的嵌套级别使用多个`JustBeforeEach`。Ginkgo将首先从外部运行所有的`BeforeEach`，然后它将从外部运行`JustBeforeEach`。虽然功能强大，但这可能会导致测试套件混乱 - 因此请谨慎使用嵌套的`JustBeforeEach`。
>
> **一些建议：**`JustBeforeEach`**是一个很容易被滥用的强大工具。好好利用它。**


### 分离诊断收集和销毁: `JustAfterEach`

在销毁（可能会破坏有用的状态）之前，在每一个It块之后，有时运行一些代码是很有用的。比如，测试失败后，执行一些诊断的操作。我们可以在上面的示例中使用它来检查测试是否失败，如果失败，则输出实际的`book`：
```go
    JustAfterEach(func() {
        if CurrentGinkgoTestDescription().Failed {
            fmt.Printf("Collecting diags just after failed test in %s\n", CurrentGinkgoTestDescription().TestText)
            fmt.Printf("Actual book was %v\n", book)
        }
    })
```

> 您可以在不同的嵌套级别使用多个`JustAfterEach`。Ginkgo将首先从内到外运行所有`JustAfterEach`，然后它将从内到外运行`AfterEach`。虽然功能强大，但这会导致测试套件混乱 - 因此合理地使用嵌套的`JustAfterEach`。
>
> **就像**`JustBeforeEach`**一样，**`JustAfterEach `**是一个很容易被滥用的强大工具。好好利用它。**

### 全局设置和销毁: `BeforeSuite` and `AfterSuite`

有时您希望在整个测试之前运行一些设置代码和在整个测试之后运行一些清理代码。例如，您可能需要启动并销毁外部数据库。

Ginkgo提供了`BeforeSuite`和`AfterSuite`来实现这一点。通常，您可以在引导程序文件的顶层定义它们。例如，假设您需要设置外部数据库：



```go
package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "your/db"

    "testing"
)

var dbRunner *db.Runner
var dbClient *db.Client

func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail)

    RunSpecs(t, "Books Suite")
}

var _ = BeforeSuite(func() {
    dbRunner = db.NewRunner()
    err := dbRunner.Start()
    Expect(err).NotTo(HaveOccurred())

    dbClient = db.NewClient()
    err = dbClient.Connect(dbRunner.Address())
    Expect(err).NotTo(HaveOccurred())
})

var _ = AfterSuite(func() {
    dbClient.Cleanup()
    dbRunner.Stop()
})
```

`BeforeSuite` 函数在任何 spec运行之前运行。如果`BeforeSuite`运行失败则没有 spec将会运行，测试套件运行结束。

`AfterSuite`函数在所有的 spec运行之后运行，无论是否有任何测试的失败。由于`AfterSuite`通常有一些代码来清理持久的状态，所以当你使用`control+c` 打断运行的测试时，Ginkgo也将会运行`AfterSuite`。要退出`AfterSuite`的运行，再次输入`control+c`。

通过传递带有Done参数的函数，可以异步运行`BeforeSuite`和`AfterSuite`。

您只能在测试套件中定义一次`BeforeSuite`和`AfterSuite`（**不需要设置多次**！）

最后，当并行运行时，每个并行进程都将运行`BeforeSuite`和`AfterSuite`函数。在[这里](#并行 specs)查看有关并行运行测试的更多信息。

### 记录复杂的`It`: `By`

按照规则，您应该记录您的`It`，`BeforEach`， 等精炼到位。有时这是不可能的，特别是在集成式测试中测试复杂的工作流时。在这些情况下，您的测试块开始隐藏通过单独查看代码难以收集的叙述。在这些情况下，Ginkgo 通过`By`来提供帮助。这里有一个很好的例子：

```go
var _ = Describe("Browsing the library", func() {
    BeforeEach(func() {
        By("Fetching a token and logging in")

        authToken, err := authClient.GetToken("gopher", "literati")
        Exepect(err).NotTo(HaveOccurred())

        err := libraryClient.Login(authToken)
        Exepect(err).NotTo(HaveOccurred())
    })

    It("should be a pleasant experience", func() {
        By("Entering an aisle")

        aisle, err := libraryClient.EnterAisle()
        Expect(err).NotTo(HaveOccurred())

        By("Browsing for books")

        books, err := aisle.GetBooks()
        Expect(err).NotTo(HaveOccurred())
        Expect(books).To(HaveLen(7))

        By("Finding a particular book")

        book, err := books.FindByTitle("Les Miserables")
        Expect(err).NotTo(HaveOccurred())
        Expect(book.Title).To(Equal("Les Miserables"))

        By("Check the book out")

        err := libraryClient.CheckOut(book)
        Expect(err).NotTo(HaveOccurred())
        books, err := aisle.GetBooks()
        Expect(books).To(HaveLen(6))
        Expect(books).NotTo(ContainElement(book))
    })
})
```

传递给By的字符串是通过[`GinkgoWriter`](#日志输出)发出的。如果测试成功，您将看不到Ginkgo绿点之外的任何输出。但是，如果测试失败，您将看到失败之前的每个步骤的打印输出。使用`ginkgo -v`总是输出所有步骤打印。

`By` 采用一个可选的`fun()`类型函数。当传入这样的一个函数时，`By`将会立刻调用该函数。这将允许您组织您的多个`It`到一组步骤，但这纯粹是可选的。在实际应用中，每个`By`函数是一个单独的回调，这一特性限制了这种方法的可用性。

---

## Spec 执行器

### 待定 Specs

您可以将单个`Spec`或容器标记为待定。这将阻止`Spec`（或者容器中的`Specs`）运行。您可以在您的`Describe`, `Context`, `It` 和 `Measure`前面添加一个`P`或者一个`X`来实现这一点：


```go
PDescribe("some behavior", func() { ... })
PContext("some scenario", func() { ... })
PIt("some assertion")
PMeasure("some measurement")

XDescribe("some behavior", func() { ... })
XContext("some scenario", func() { ... })
XIt("some assertion")
XMeasure("some measurement")
```

> 当您标记一个`It`或者`Meature`为`Pending`态时，您不必删掉`fun() {...}`。 Ginkgo 会自动忽略字符串后面的任何参数。

> 默认，Ginkgo将会打出每一个处于`Pending`态的`Spec`的说明。您可以通过设置`--noisyPendings=false`标签来关闭它。

> 默认，Ginkgo不会因为有处于`Pending`态的 spec而导致失败。您可以通过设置`--failOnPending`标签来改变它。

在编译时，使用`P`和`X`将 spec标记为`Pending`态。如果您需要在运行时（可能是由于只能在运行时才知道约束）跳过一个 spec。您可以在您的测试中调用`Skip`：

```go
It("should do something, if it can", func() {
    if !someCondition {
        Skip("special condition wasn't met")
    }

    // assertions go here
})
```
> 默认地，Ginkgo将会为每一个跳过的 spec打印输出一份说明。您可以通过设置`--noisySkippings=false`标签来关闭它。

注意：`Skip(...)`导致闭包退出，所以没有必要返回它。

### 聚焦 Specs

当开发的时候，运行 spec的子集将会非常方便。Ginkgo有两种机制可以让您专注于特定 spec：

1. 您可以在`Descirbe`, `Context` 和 `It`前面添加F以编程方式专注于单个 spec或者整个容器的 spec：
    ```go
    FDescribe("some behavior", func() { ... })
    FContext("some scenario", func() { ... })
    FIt("some assertion", func() { ... })
    ```

   这样做是为了指示Ginkgo只运行这些 spec。要运行所有 spec，您需要退回去并删除所有的 `F`。

2. 您可以使用`--focus = REGEXP`和/或`--skip = REGEXP`标签来传递正则表达式。Ginkgo只运行 `focus` 正则表达式匹配的 spec，不运行`skip`正则表达式匹配的 spec。

3. 为了防止 spec不能在测试组之间提供足够的等级区分，可以通过`--regexScansFilePath`选项，将目录加载到`focus`和`skip`的匹配中。也就是说，如果测试的初始代码位置是`test/a/b/c/my_test.go`，可以将`--focus=/b/`和`--regexScansFilePath=true`结合起来，专注于包含路径`/b/`的测试。此功能对于在创建这些测试的原始目录的行中过滤二进制工件中的测试是十分有用的。但理想情况下，您应该遵循最大限度地减少使用此功能的需求来组织您的 spec。

当Ginkgo检测到以编程式为测试中心的测试组件时，它将以非零状态码退出。这有助于检测CI系统上错误提交的重点测试。当传入命令行`focus`/`skip`标志时，Ginkgo以`0`状态码退出。如果要将测试集中在CI系统上，则应该显示地传入`-focus`或`-skip`标志。

嵌套的以编程方式为重点的 spec遵循一个简单的规则：如果叶子节点被标记为重点，那么它的被标记为重点的任何根结点将变为非重点。根据这个规则，标记为重点的兄弟叶子节点（无论相对深度如何），将会运行无论共享的根结点是否是重点；非重点的兄弟节点将不会运行无论共享的根结点或者相对深度的兄弟姐妹是否是重点。更简单地：

```go
FDescribe("outer describe", func() {
    It("A", func() { ... })
    It("B", func() { ... })
})
```

will run both `It`s but

```go
FDescribe("outer describe", func() {
    It("A", func() { ... })
    FIt("B", func() { ... })
})
```

只会运行`B`，这种行为倾向于更紧密地反应开发人员在测试套件上进行迭代时的实际意图。

> 程序化方法和`--focus=REGEXP`/`--skip=REGEXP`方法是互斥的。使用命令行标志将覆盖程序化的重点。

> 专注于没有`It`或者`Measure`的叶子节点的容器是没有意义的。由于容器中没有任何东西可以运行，因此实际上，Ginkgo忽略了它。

> 使用命令行标志时，您可以指定`--focus`和`--skip`中的一个或两个。如果都指定了，则他们的限制将都会生效。

> 您可以通过运行`ginkgo unfocus`来取消以编程为中心的测试的关注。这将从您当前目录中可能具有任何`FDescribe`，`FContext`和`FIt`的测试中删除`F`。

> 如果你想跳过整个包（当使用`-r`标志递归运行`ginkgo`时），你可以将逗号分隔的列表传递给`--skipPackage = PACKAGES, TO, SKIP`。包含列表中目录的任何包都将会被忽略。

### Spec 序列

默认情况下，Ginkgo 会将你 specs 的运行顺序打乱。这样有助于你在开发测试套件时及早发现测试污染。

Ginkgo 的默认只会变换顶级容器的顺序，那些容器中的 specs 则继续依照测试文件中指定的顺序运行。这样有助于减轻开发时 specs 不断变换运行顺序时的理解难度。

想要让套件中所有 specs 随机的话，你可以传入 `--randomizeAllSpecs` 参数。这在 CI （持续集成）和防止测试污染时很有用。

Ginkgo 使用当前时间作为随机种子。种子的值会被打印在测试输出的起始位置附件。如果你发现测试出现间歇性的错误，并且你认为可能是测试污染引起的，你可以使用失败套件的种子来准确的重现套件的运行顺序。传递参数`--seed=SEED`即可。
当你运行多个 spec 套件时，Ginkgo 默认按照文件系统排列的顺序来执行套件。你可以通过命令 `ginkgo --randomizeSuites` 来改变套件的顺序。

### 并行 Specs

Ginkgo 支持并行运行 specs 。它的实现方式是大量产生 `go test` 进程并且通过一个共享队列为每个进程提供 specs 。这对于一个 BDD 测试框架非常重要，因为闭包的共享内容(context)在并发中容易出错。

要并发运行 Ginkgo 套件的话你必须使用 `ginkgo` CLI。只需要传递`-p` 参数：

    ginkgo -p

它将自动检测需要生成测试节点的最佳个数（请看下面的注释）。

如要指定生成的节点数，使用 `-nodes` ：

    ginkgo -nodes=N

> 你不需要同时指定 `-p` 和 `-nodes `。设置任何大于1的 `-nodes`  都意味着执行并发测试。

> 使用 `-p`  时节点的数量时如果 `runtime.NumCPU() <= 4`则 `runtime.NumCPU()` ，否则就是 `runtime.NumCPU() - 1` ，这是基于严格的启发式科学就跟“几个月经验养成的直觉”一样。

测试执行器整理运行中进程的输出统一放入一个连续的输出中。这背后的机制是客户端-服务端模型：当每个客户端套件完成一个测试，测试输出和状态会被发送给服务端，服务端收到后打印到屏幕。这将同时运行的测试输出整理到一个连续的(既无间隔的)，聚合的输出中。

有时候需要或更想要实时查看单个并行套件的输出。你可以设置`-stream`：

    ginkgo -nodes=N -stream

使用 `-stream` 参数运行测试的时候，执行器直接输出各自运行节点的日志（它会在每行开头预置输出的节点 id）。这种结果缺少连续输出（来自不同节点的日志交叉在一起），但是在调试古怪的或者待定的测试套件的时候会有用。

> 在 windows 系统中，并行测试默认带有 `-stream` 因为Ginkgo 不能捕获日志到 stdout/stderr  (对于聚合必要)。

#### 并发测试套件管理外部进程

如果你启动或者连接到外部进程，你要确保那些连接在并行环境（context）下是安全的。一种方法是为每一个 Ginkgo 进程启动一个单独外部资源的实例。举个例子，比如说你要启动并连接一个数据库。你可以启动不同数据库服务并绑定到并行进程的对应端口：

```go
package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
    "github.com/onsi/ginkgo/config"

    "your/db"

    "testing"
)

var dbRunner *db.Runner
var dbClient *db.Client


func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail)

    RunSpecs(t, "Books Suite")
}

var _ = BeforeSuite(func() {
    port := 4000 + config.GinkgoConfig.ParallelNode

    dbRunner = db.NewRunner()
    err := dbRunner.Start(port)
    Expect(err).NotTo(HaveOccurred())

    dbClient = db.NewClient()
    err = dbClient.Connect(dbRunner.Address())
    Expect(err).NotTo(HaveOccurred())
})

var _ = AfterSuite(func() {
    dbClient.Cleanup()
    dbRunner.Stop()
})
```


 `github.com/onsi/ginkgo/config`  包为你的套件提供了获取传入 Ginkgo 命令行配置参数的能力。 `config.GinkgoConfig.ParallelNode` 参数是当前节点的索引（从1开始，到 N ）。类似的  `config.GinkgoConfig.ParallelTotal`  是运行中并行节点的总数。

#### 并发套件管理单个外部进程

可以的话，你应该尽可能为每个并行节点新建一个外部资源的实例。这样严格隔离每个并行节点有助于避免测试污染。

有时候（罕见的）这样是行不通的。也许，在你控制之外的原因，你在自己机器上只能开启一个服务实例。Ginkgo 提供了替代方案`SynchronizedBeforeSuite` 和 `SynchronizedAfterSuite`。

原理很简单。 `SynchronizedBeforeSuite` 使 Ginkgo 可以让你只在一个并行节点运行准备工作的初始化代码，其它初始化代码则全部节点都运行。Ginkgo 同步了这些方法并保证节点 1 会在其它节点运行初始化代码前运行准备工作的初始化代码。此外，Ginkgo 能使运行在节点1 准备工作的初始化代码传递信息给其它节点运行初始化代码。

这是我们早些的使用`SynchronizedBeforeSuite` 的数据库案例：

```go
var _ = SynchronizedBeforeSuite(func() []byte {
    port := 4000 + config.GinkgoConfig.ParallelNode

    dbRunner = db.NewRunner()
    err := dbRunner.Start(port)
    Expect(err).NotTo(HaveOccurred())

    return []byte(dbRunner.Address())
}, func(data []byte) {
    dbAddress := string(data)

    dbClient = db.NewClient()
    err = dbClient.Connect(dbAddress)
    Expect(err).NotTo(HaveOccurred())
})
```

`SynchronizedBeforeSuite` 必须传入两个函数。第一个必须返回`[]byte`并且第二个必须接受`[]byte`。当运行多节点的时候，第一个函数只在节点 1 运行。一个函数结束后，所有节点（包括节点1）继续运行第二个函数并且会接受第一个函数返回的数据。这个例子中，我们使用数据传输机制使用数据库的地址（在节点1初始化）传给所有节点。

为了正确清理，你应该使用 `SynchronizedAfterSuite`。继续我们的案例：

```go
var _ = SynchronizedAfterSuite(func() {
    dbClient.Cleanup()
}, func() {
    dbRunner.Stop()
})
```

使用 `SynchronizedAfterSuite` 的话，第一个函数会在所有节点（包括节点1）运行。第二个函数只会在节点1 运行。此外，第二个函数只会在其它节点都运行结束的情况下运行。这很重要，因为节点 1 负责初始化和销毁单例资源，它必须等待其它节点结束才能销毁其它节点依赖的资源。

最后，所有这些函数都能传入一个惯用的`Done`参数用于异步运行。当异步运行的时候，一个可选的超时能作为第三个传入`SynchronizedBeforeSuite` 和 `SynchronizedAfterSuite`的参数。

> 请注意一个微妙之处： `dbRunner`变量值存在与节点1。其它节点都不应该试图使用该变量中的数据（其他节点上为空）。`dbClient`变量只会存在于`SynchronizedBeforeSuite`函数，当然，所有节点都能用。

---

## 理解 Ginkgo 的生命周期

Ginkgo 的使用者有时候在 Ginkgo 的生命周期上犯错误。这个章节提供了一个思维模型来帮助你理解什么代码在什么时候运行。

Ginkgo 致力于谨慎地控制 specs 的运行顺序，并且对运行中 [多进程](https://onsi.github.io/ginkgo/#parallel-specs)的并行测试套件提供无缝的支持。为了完成这些，Ginkgo 需要**预先**知道测试套件的整个测试树（比如`Describe`s, `Context`s, `BeforeEach`es, `It`s,等的嵌套结构）。Ginkgo 使用这个树来构造序列，（[伪随机](https://onsi.github.io/ginkgo/#spec-permutation)），运行的测试列表。

这意味着所有这些测试必须在 Ginkgo 运行套件*前*都定义好。因为在套件运行的时候，尝试定义一个新的测试会出现错误（例如在 `It` 块中调用 `It`）。

当让，根据配置动态生成测试套件也是可行的（实际上经常这样）。但是 Ginkgo 生命周期中你必须*在正确的时间*生成测试。这些细微差别有时候让用户犯错。

我们来卡一下典型的 Ginkgo 测试套件。跟在测试套件后面的是一个多文件的 books 包：

```go
// books_suite_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/books"

    "testing"
)

func TestBooks(t *testing.T) {     // L1
    RegisterFailHandler(Fail)      // L2
    RunSpecs(t, "Books Suite")     // L3
}                                  // L4
```

```go
// reading_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/books"

    "testing"
)

var _ = Describe("When reading a book", func() {                        // L5
    var book *books.Book                                                // L6

    BeforeEach(func() {                                                 // L7
        book = books.New("The Chronicles of Narnia", 300)               // L8
        Expect(book.CurrentPage()).To(Equal(1))                         // L9
        Expect(book.NumPages()).To(Equal(300))                          // L10
    })                                                                  // L11

    It("should increment the page number", func() {                     // L12
        err := book.Read(3)                                             // L13
        Expect(err).NotTo(HaveOccurred())                               // L14
        Expect(book.CurrentPage()).To(Equal(4))                         // L15
    })                                                                  // L16

    Context("when the reader finishes the book", func() {               // L17
        It("should not allow them to read more pages", func() {         // L18
            err := book.Read(300)                                       // L19
            Expect(err).NotTo(HaveOccurred())                           // L20
            Expect(book.IsFinished()).To(BeTrue())                      // L21
            err = book.Read(1)                                          // L22
            Expect(err).To(HaveOccurred())                              // L23
        })                                                              // L24
    })                                                                  // L25
})                                                                      // L26
```

```go
// isbn_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/books"

    "testing"
)

var _ = Describe("Looking up ISBN numbers", func() {                                                   // L27
    Context("When the book can be found", func() {                                                     // L28
        It("returns the correct ISBN number", func() {                                                 // L29
            Expect(books.ISBNFor("The Chronicles of Narnia", "C.S. Lewis")).To(Equal("9780060598242")) // L30
        })                                                                                             // L31
    })                                                                                                 // L32

    Context("When the book can't be found", func() {                                                   // L33
        It("returns an error", func() {                                                                // L34
            isbn, err := books.ISBNFor("The Chronicles of Blarnia", "C.S. Lewis")                      // L35
            Expect(isbn).To(BeZero())                                                                  // L36
            Expect(err).To(HaveOccurred())                                                             // L37
        })                                                                                             // L38
    })                                                                                                 // L39
})                                                                                                     // L40
```

当你运行 `ginkgo` cli 的时候，按顺序发生了下面这些事：

当你运行 `ginkgo` cli 的时候，按顺序发生了下面这些事：

1. `ginkgo `运行  `go test -c `来编译测试二进制文件
2. `ginkgo` 启动测试二进制文件（这相当于直接运行 `go test`  ，但是 Ginkgo 如果还要运行多进程测试的话，就不需要重新编译了）
3. 加载测试二进制文件到内存并且定义和调用顶层函数。准确点说，则意味着：
   1. `TestBooks` 函数已经被定义（行 `L1`）
   2. 行`L5` 的 `Describe`被调用并且传入了字符串“When reading a book”,还有包含测试的匿名函数嵌入到了`Describe`中。
   3. 
      行`L27`的 `Describe`被调用并且传入了字符串“Looking up ISBN numbers”,还有包含测试的匿名函数嵌入到了`Describe`中。

   注意传入`Describe` 的匿名函数现在还没被调用。这个时间点，Ginkgo 只是知道了套件有两个顶层容器。
4.  `go test` 运行时调用`TestBooks()` 开始执行测试(`L1`)
5. Ginkgo 的 `Fail` 句柄（ handler ）通过`RegisterFailHandler` (`L2`) 被注册到 `gomega`-这是必要的，因为`ginkgo` 和`gomega`并不是固定配对的，替换的匹配库也能和 Ginkgo 一起使用。
6. `RunSpecs`被调用(`L3`)。这做了一些事情：
   **测试树构造阶段：**
   1. Ginkgo 遍历每个顶层容器（即行 `L5` 和 `L27` 的两个`Describe`）并调用他们的匿名函数。
   2. 当`L5`的函数被调用：

       - 定义一个名为 `book` 的闭包变量(`L6`)
       - 注册一个`BeforeEach`并传递个匿名函数 (`L7`)。这个函数注册并保存为测试树的一部分而且**还没执行**。
       - 注册一个带描述和匿名函数的`It` (`L12`)。这个函数注册并保存为测试树的一部分而且**还没执行**。
       - 添加一个嵌入的`Context`(`L17`)。传入到 `Context` 的匿名函数**马上**被调用从而继续构建测试树。行`L18` 注册`It`。
       - 这个时间点 `L5` 的匿名函数存在**并永不会被再次调用**。

   3. 行`L27`的顶层`Describe` 中的函数被调用，行为跟前面类似。
   2. 这个时候顶层容器已经被调用，测试树像这样：
        ```
        [
          ["When Reading A Book", BeforeEach, It "should increment the page number"],
          ["When Reading A Book", BeforeEach, "When the reader finishes the book", It "should not allow them to read more pages"],          
          ["Looking up ISBN numbers", "When the book can be found", It "returns the correct ISBN number"],
          ["Looking up ISBN numbers", "When the book can't be found", It "returns an error"],
        ]
        ```
        在这里，测试树中的`BeforeEach` 和 `It`  都包含他们各自匿名函数的引用。注意有四个测试，每个都对应测试中定义的一个 `It`。

   已经构造了测试树，现在 Ginkgo 能基于随机种子进行随机 `It` 。这就是简单地随机上述测试列表。

   **测试树调用阶段：** 为了运行测试，Ginkgo 现在简单遍历随机测试树。调用匿名函数按顺序连接到所有 `BeforeEach` 和 `It` 。例如，当运行行 `L18` 的测试时，Ginkgo 会首先调用行 `L7`  传入 `BeforeEach` 的匿名函数，然后调用行 `L18` 传入 `It`  的匿名函数。

   > 再次强调， 行`L5` 的父闭包不会被重复调用。传入  `Describe`s 和 `Context`s 的函数只会在**构造树阶段**调用。

   > 在测试树调用阶段定义新的 `It`, `BeforeEach` 是错误的。

7. `RunSpecs` 持续跟踪运行测试和测试错误，更新测试运行时的所有附属报告。当测试完成，`RunSpecs` 和 `TestBooks` 函数退出。

这里有很多的详情，但可以全部归结成一个极简的流程。总结：

运行一个 Ginkgo 测试有两个阶段。

在**测试树构造阶段** 匿名函数传入所有被调用的容器（ 即`Describe` and `Context`）。这些函数定义闭包变量，调用子节点（`It`, `BeforeEach`, 和 `AfterEach` 等）来定义测试树。这些传入子节点的匿名函数在测试树构造阶段**不会**被调用。随后构造的树被随机化。

在**测试树调用阶段**，子节点函数被按顺序调用。注意容器函数这个阶段不会被调用。



### 防止测试污染

因为**测试树调用阶段**被传入到容器的匿名函数不会被重新调用，你不应该指望容器的变量初始化会被重新调用。你必须手动在`BeforeEach`函数中重新初始化任何会被测试改变的变量。

考虑下，例如，将行上面行 `L5 ` 的  `"When reading a book"` `Describe`  容器改变为：

```
var _ = Describe("When reading a book", func() {                                        //L1'
    var book *books.Book                                                                //L2'
    book = books.New("The Chronicles of Narnia", 300) // create book in parent closure  //L3'

    It("should increment the page number", func() {                                     //L4'
        err := book.Read(3)                                                             //L5'
        Expect(err).NotTo(HaveOccurred())                                               //L6'
        Expect(book.CurrentPage()).To(Equal(4))                                         //L7'
    })                                                                                  //L8'

    Context("when the reader finishes the book", func() {                               //L9'
        It("should not allow them to read more pages", func() {                         //L10'
            err := book.Read(300)                                                       //L11'
            Expect(err).NotTo(HaveOccurred())                                           //L12'
            Expect(book.IsFinished()).To(BeTrue())                                      //L13'
            err = book.Read(1)                                                          //L14'
            Expect(err).To(HaveOccurred())                                              //L15'
        })                                                                              //L16'
    })                                                                                  //L17'
})                                                                                      //L18'
```

这个变化不只是 book 变量在所有 `It` 中共享。而是同一个 book 实例在两个 `It` 中共享。这会导致扰乱测试污染，book 改为依赖 `It` 执行顺序。例如，如果 测试 `"should increment the page number"` (`L4'`)  先被调用，然后测试 `"should increment the page number"` (`L4'`)调用结果为异常测试失败， 由于book 已经是` finished`(`L13'`)。

这种测试污染的正确解决方案是将初始化变量放入 `BeforeEach` 块中。这样保证测试状态在每个测试中都是干净的。

### 不要在容器节点函数进行断言

一个相关的，常见的错误是在容器节点的匿名函数进行断言。断言必须只能在子节点的函数中，因为只有那些函数会在**测试树调用阶段**运行。

所以，避免如下代码：

```
var _ = Describe("When reading a book", func() {
    var book *books.Book
    book = books.New("The Chronicles of Narnia", 300)
    Expect(book.CurrentPage()).To(Equal(1))
    Expect(book.NumPages()).To(Equal(300))     

    It("...")
})
```

如果那些断言失败，他们会做在**测试树构造阶段**做这些，而不是在 Ginkgo 跟踪并报告错误的**测试树调用阶段**。正确的做法是，将初始化变量和执行正确性断言放入 `BeforeEach` 块中。

### 动态生成测试模式

一个常见模式(相关例子 [动态运行测试](#shared-example-patterns))是基于外部输入(例如一个文件或环境变量)动态生成测试套件。

想象以下，例如，一个名为 `isbn.json` 的文件，包含一套已知的 ISBN 索引：

```json
// isbn.json
[
  {"title": "The Chronicles of Narnia", "author": "C.S. Lewis", "isbn": "9780060598242"},
  {"title": "Ender's Game", "author": "Orson Scott Card", "isbn": "9780765378484"},  
  {"title": "Ender's Game", "author": "Victor Hugo", "isbn": "9780140444308"},  
]
```

你可能想为生成一堆测试，一本书一个测试。推荐模式是：

```go
// isbn_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/books"

    "testing"
)

var _ = Describe("Looking up ISBN numbers", func() {
    testConfigData := loadTestISBNs("isbn.json")           

    Context("When the book can be found", func() {
        for _, d := range testConfigData {
            d := d //necessary to ensure the correct value is passed to the closure
            It("returns the correct ISBN number for " + d.Title, func() {                                                
                Expect(books.ISBNFor(d.Title, d.Author)).To(Equal(d.ISBN))
            })                                                                                            
        }
    })                                                                                                
})                                                                                                    
```

这里 `data` 在**测试构建阶段**使用 `isbn.json`  文件初始化，然后用于定义一套`It`。

如果你已有类似测试配置数据，你想要在顶层`Describes`共享，或者你想在每个`Describe` （这里展示的）中加载，或者直接在全部共享变量中加载一次。推荐的模式是在`RunSpecs` 之前加载这些变量：


```go
// books_suite_test.go

package books_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/books"

    "testing"
)

var testConfigData TestConfigData

func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail) 
    testConfigData = loadTestISBNs("isbn.json")
    RunSpecs(t, "Books Suite")
}                             
```

这里，`testConfigData` 可以被任意 `Describe` 或 `Context` 闭包引用，并且保证在**测试构建阶段**之前初始化，直到`RunSpecs` 被调用。

如果你必须在 `testConfigData` 执行一个断言，你可以像下面代码一样，在 `BeforeSuite` 做：

```go
func TestBooks(t *testing.T) {
    RegisterFailHandler(Fail) 
    testConfigData = loadTestISBNs("isbn.json")
    RunSpecs(t, "Books Suite")
}

var _ = BeforeSuite(func() {
    Expect(testConfigData).NotTo(BeEmpty())
})
```                           

这能行得通，因为 `BeforeSuite` 函数只在**测试树调用阶段**执行一次。

最后，人们最常见的错误：在**测试树调用阶段**，动态生成测试用于初始化节点中的测试配置数据。例如：

```go

var _ = Describe("Looking up ISBN numbers", func() {
    var testConfigData TestConfigData

    BeforeEach(func() {
        testConfigData = loadTestISBNs("isbn.json") // WRONG!
    })

    Context("When the book can be found", func() {
        for _, d := range testConfigData {
            d := d //necessary to ensure the correct value is passed to the closure
            It("returns the correct ISBN number for " + d.Title, func() {                                                
                Expect(books.ISBNFor(d.Title, d.Author)).To(Equal(d.ISBN))
            })                                                                                            
        }
    })                                                                                                
})  
```

这会生成零个测试，由于  `testConfigData` 在**测试构造阶段**是空的。

---

## 异步测试

Go 的并发做得很好。Ginkgo 为高效异步测试提供支持。

考虑这个案例：

```go
It("should post to the channel, eventually", func() {
    c := make(chan string, 0)

    go DoSomething(c)
    Expect(<-c).To(ContainSubstring("Done!"))
})
```
这个测试会阻塞直到接受到通道`c`的响应。对于这种测试，一个死锁或超时是常见的错误模式。对于这种情况，一个常见模式是在底部添加一个 select 语句 ，并包括一个`<-time.After(X)`通道来指定超时。

Ginkgo 有这种内置模式。在所有无容器块（`It`, `BeforeEach`, `AfterEach`, `JustBeforeEach`, `JustAfterEach`, 和 `Benchmark`）中`body`函数能接受一个可选的`done Done` 参数：

```go
It("should post to the channel, eventually", func(done Done) {
    c := make(chan string, 0)

    go DoSomething(c)
    Expect(<-c).To(ContainSubstring("Done!"))
    close(done)
}, 0.2)
```

`Done` 是一个 `chan interface{}`。当 Ginkgo 检测到 `done Done` 参数已经被请求了，它会运行 用 goroutine 运行 `body` 函数，并将它包裹到一个应用超时断言的必要逻辑中。你必须要么关闭 `done` 通道，要么发送一些东西（任何东西都行）给它来告诉 Ginkgo你的测试已经结束。如果你的测试超时不结束，Ginkgo会让测试失败并进行下一个。 

默认的超时是 1 秒。你可以在 `body` 函数后面传递一个 `float64` （秒为单位）修改超时时间。

> Gomega 对于丰富的异步代码断言有额外支持。确保查看了 `Eventually` 在 Gomega 是如何工作的。

---

## Ginkgo CLI

可以通过如下命令来安装Ginkgo命令：

    $ go install github.com/onsi/ginkgo/ginkgo

Ginkgo 比 `go test` 提供了更多方便的指令。推荐使用 Ginkgo 命令虽然这不是必需的。

### 运行测试

在当前目录下运行该套件，只需：

    $ ginkgo #or go test

在其它目录下运行该套件，只需：

    $ ginkgo /path/to/package /path/to/other/package ...

传递参数和特定的标签到该测试套件：

    $ ginkgo -- <PASS-THROUGHS>

注意：这个"--"是重要的。只有该双横线后面的参数才会被传递到测试套件。要在你的测试套件中解析参数和特定标签，需要声明一个变量并在包级别初始化它：

```go
var myFlag string
func init() {
    flag.StringVar(&myFlag, "myFlag", "defaultvalue", "myFlag is used to control my behavior")
}
```

当然，Ginkgo使用一些标签。在运行指定的包之前必须指定这些标签。以下是调用语法的摘要：

    $ ginkgo <FLAGS> <PACKAGES> -- <PASS-THROUGHS>

下面是Ginkgo可以接受的一些参数：

**指定运行哪些测试套件：**

- `-r`

    使用`-r`递归运行目标文件夹下的所有测试套件。适用于在所有包中运行所有测试。

- `-skipPackage=PACKAGES,TO,SKIP`

    当运行带有 `-r`  的测试，你可以传递一个逗号分隔的条目列表给 `-skipPackage`  。任何包的路径如果含有逗号分隔的条目列表之一就会被跳过。

**并行测试：**

- `-p`

    设置  `-p` 可以并行运行测试套件并自动经检测节点数。

- `--nodes=NODE_TOTAL`

    使用这个可以并行运行测试套件并使用 NODE_TOTAL 个数的进程。你不需要指定`-p` （尽管你可以！）。

- `-stream`

    默认地，当你并行运行测试套件，测试执行器从每个并行节点聚合数据，在运行测试的时候产生连贯的输出。设置 `stream`为 `true`，则会实时以流形式输出所有并行节点日志，每行头都会带有相应节点 id 。

**修改输出：**

- `--noColor`

    如果提供该参数，Ginkgo 默认不使用多种颜色打印报告。

- `--succinct`

    Succinct （简洁）会静默 Ginkgo 的详情输出。成功执行的测试套件基本上只会打印一行！当在一个包中运行测试的时候，Succinct 默认关闭。它在 Ginkgo 运行多个测试包的时候默认打开。

- `--v`

    如果设置该参数， Ginkgo 默认报告会在每个 spec 运行前打印文本和位置。同时，GinkgoWriter 会实时刷新输出到标准输出。

- `--noisyPendings=false`

    默认情况下，Ginkgo 默认报告会提供暂停 spec 的详情输出。你可以设置` --noisyPendings=false` 来禁止该行为。

- `--noisySkippings=false`

    默认情况下，Ginkgo 默认报告会提供跳过 spec 的详情输出。你可以设置` --noisySkippings=false` 来禁止该行为。

- `--reportPassed`

    如果设置该参数，Ginkgo 默认报告会提供通过 spec 的详情输出。

- `--reportFile=<file path>`

    在指定路径(相对路径或绝对路径)创建报告输出文件。它会同时覆盖预设的`ginkgo.Reporter` 路径，并且父目录不存在的话会被创建。

- `--trace`

    如果设置该参数，Ginkgo 默认报告会为每个失败打印全栈跟踪日志，不仅仅打印失败发现的行号。

- `--progress`

    如果设置该参数，当 Ginkgo 进入并运行每个 `BeforeEach`, `AfterEach`, `It` 节点的时候，Ginkgo 会输出过程到 `GinkgoWriter`。这在调试被卡主的测试时（例如测试卡在哪里？），或使用测试输出更多易读的日志到`GinkgoWriter` （例如什么日志在`BeforeEach`中输出？什么日志在`It`中输出？）。结合 `--v`  输出 `--progress` 日志到标准输出。

**控制随机性：**

- `--seed=SEED`

    变换 spec 顺序时使用的随机种子。

- `--randomizeAllSpecs`

    如果设置该参数，所有 spec 都会被重新排序。默认 Ginkgo 只会改变顶层容器的顺序。

- `--randomizeSuites`

    如果设置该参数并运行多个 spec 套件，specs 运行的顺序会被随机化。

**聚焦 spec 和跳过 spec：**

- `--skipMeasurements`

    如果设置该参数，Ginkgo 会跳过任何你定义的 `Measure` spec 。

- `--focus=REGEXP`

    如果设置该参数，Ginkgo 只会运行带有符合正则表达式 REGEXP 的描述的 spec。

- `--skip=REGEXP`

    如果设置该参数，Ginkgo 只会运行不有符合正则表达式 REGEXP 的描述的 spec。

**运行竞态检测和测试覆盖率工具：**

- `-race`

    设置`-race` 来让 `ginkgo` CLI 使用竞态检测来运行测试。

- `-cover`

    设置`-race` 来让 `ginkgo` CLI 使用代码覆盖率分析工具来运行测试（Go 1.2+ 的功能）。Ginkgo 会在在个测试包的目录下生成名为`PACKAGE.coverprofile` 的代码覆盖文件。

- `-coverpkg=<PKG1>,<PKG2>`

    `-cover`, `-coverpkg` 运行你的测试并开启代码覆盖率分析。然而， `-coverpkg` 允许你知道需要分析的包。它允许你获得当前包之外的包的代码覆盖率，这对集成测试很有用。注意，它默认不在当前包运行覆盖率分析，你需要制定所有你想分析的包。包名应该是全写，例如`github.com/onsi/ginkgo/reporters/stenographer`。

- `-coverprofile=<FILENAME>`

    使用 `FILENAME` 重命名代码覆盖率文件的名字。
    
- `-outputdir=<DIRECTORY>`
   
    将覆盖率输出文件移到到指定目录。<br />
    结合`-coverprofile` 参数也能使用。
    
**构建参数：**

- `-tags`

    设置`-tags`来传递 标识到编译步骤。

- `-compilers`

    当编译多个测试套件（如 `ginkgo -r`），Ginkgo 会使用 `runtime.NumCPU()` 绝对启动的编译进程数。在一些环境中这不是个好主意。你可以通过这个参数手动指定编译器进程数。

**失败行为：**

- `--failOnPending`

    如果设置该参数，Ginkgo 会在有暂停 spec 的情况下使套件失败。

- `--failFast`

    如果设置该参数，Ginkgo 会在第一个 sepc 时候后立即停止套件。

**监视参数：**

- `--depth=DEPTH`

    当监视包的时候，Ginkgo 同时监视包依赖的变化。默认的 `--depth` 为 1 ，意味着只有直接依赖的包被监控。你能调整它到 依赖的依赖（dependencies-of-dependencies），或者设置为零就只监控它自己，不监控依赖。

- `--watchRegExp=WATCH_REG_EXP`

    当监视包的时候，Ginkgo只监控符合该正则表达式的文件。默认值是`\.go$` ，意味着只有 go 文件的变化会被监视。

**减少随机失败的测试(flaky test):**

- `--flakeAttempts=ATTEMPTS`

    如果一个测试失败了，Ginkgo 能马上返回。设置这个参数大于 1 的话会重试。只要一个重试成功，Ginkgo 就不会认为测试套件失败。单独失败的运行仍会被报告在输出中；举个例子，JUnit 输出中，会声称 0 失败（因为套件通过了），但是仍会包含一个同时失败和成功的测试的所有失败的运行。

    这个参数很危险！不要试图使用它来掩盖失败的测试！

**杂项：**

- `-dryRun`

    如果设置该参数，Ginkgo 会遍历你的测试套件并报告输出，但是不会真正运行你的测试。这最好搭配`-v`来预览你将运行的测试。测试的顺序遵循了 `--seed` 和 `--randomizeAllSpecs` 指定的随机策略。

- `-keepGoing`

    默认地，当多个测试运行的时候（使用 `-r`或一列表的包），Ginkgo 在一个测试失败的时候会中断。要让 Ginkgo 时候后继续接下来的测试套件，你可以设置 `-keepGoing`。

- `-untilItFails`

    如果设置为 `true`，Ginkgo 会持续运行测试直到发送失败。这会有助于弄明白竞态条件或者古怪测试。最好搭配 `--randomizeAllSpecs` 和 `--randomizeSuites` 来变换迭代的测试顺序。

- `-notify`

    设置 `-notify`  来接受桌面测试套件完成的通知。结合子命令 `watch`  特别有用。当前 `-notify`  只有 OS X 和 Linux 支持。在 OS X 上，你需要运行 `brew install terminal-notifier` 来接受通知，在 Linux 你需要下载安装 `notify-send`。

- `--slowSpecThreshold=TIME_IN_SECONDS`

    默认地，Ginkgo报告器会表示运行超过 5 秒的测试，这不会使测试失败，它只是通知你该 sepc 运行慢。你可以使用这个参数修改该门槛。

- `-timeout=DURATION`

    如果时间超过 `DURATION` ，Ginkgo 会使测试套件失败。默认值是 24 小时。

- `--afterSuiteHook=HOOK_COMMAND`

    Ginko 有能力在套件测试结束后运行一个命令（a command hook）。你只需给它需要运行的命令，它就会替换字符串来传给命令数据。举例：` –afterSuiteHook=”echo (ginkgo-suite-name) suite tests have [(ginkgo-suite-passed)]” `，这个测试沟子会替换 (ginkgo-suite-name) 和 (ginkgo-suite-passed) 为套件名和各自的通过/失败状态，然后输出到终端。

- `-requireSuite`

    如果你使用 Ginkgo 测试文件创建包，但是你忘了运行  `ginkgo bootstrap`  初始化，你的测试不会运行而且该套件会一致通过。Ginkgo 会通知你  `Found no test suites, did you forget to run "ginkgo bootstrap"?`  ，但是不会失败。如果有测试文件但没有引用`RunSpecs.`，这个参数使得 Ginkgo 标识套件为失败。

### 监视修改

Ginkgo CLI 提供子命令  `watch`  ，监视（几乎）所有的  `ginkgo`  命令参数。使用`ginkgo watch` ，Ginkgo 会监控当前目录的包，当有修改的时候就触发测试。

你也可以使用 `ginkgo watch -r`  递归监控所有包。

对每个被监控的包，Ginkgo 也会监控包的依赖并在依赖产生修改的时候触发测试套件。默认地，`ginkgo watch`  监控包的直接依赖。你可以使用  `-depth`  来调整。设置 `-depth` 为0则不监控依赖，设置  `-depth`  大于 1 则监控更深依赖路径。

在 Linux 或 OS X 传递  `-notify`  参数，会在 `ginkgo watch` 触发和完成测试的时候产生桌面通知。

### 预编译测试

Ginkgo 对写集成风格的验收测试（integration-style acceptance tests）有强力的支持。比如，这些测试有助于验证一个复杂分布式系统的函数是否正确。它常便于分布这些作为单独二进制文件的验收测试。
Ginkgo 允许你这样构建这些二进制文件：

    ginkgo build path/to/package

这会产生一个名为 `package.test `的预编译二进制文件。然后，你能直接调用 `package.test` 来运行测试套件。原理很简单， `ginkgo` 只是调用 `go test -c -o` 来编译 `package.test` 二进制文件。
直接调用  `package.test` 会*连续*运行测试。要并行测试的话，你需要 `ginkgo` cli 编排并行节点。你可以运行：

    ginkgo -p path/to/package.test

来这样做。因为 Ginkgo CLI 是一个单独二进制文件，你能直接分布两个二进制文件，来提供一个并行(所以快速)的集成风格验收测试集合。

> `build`子命令接受一系列 `ginkgo` 和 `ginkgo watch` 接收的参数。这些参数仅限关注于编译时，就像 `--cover` 和 `--race`。通过 `ginkgo help build`，你能获得更多信息。

> 使用标准 `GOOS` 和 `GOARCH` 环境变量，你能交叉编译并面向不同平台。因此，在 OS X 上运行 `GOOS=linux GOARCH=amd64 ginkgo build path/to/package` ，会产生一个能在 Linux 上运行的二进制文件。
### 生成器

- 在当前目录，为一个包引导 Ginkgo 测试套件，可以运行：

        $ ginkgo bootstrap

    这会生成一个名为 `PACKAGE_suite_test.go` 的文件，PACKAGE 是当前目录的名称。

- 如要添加一个测试文件，运行：

        $ ginkgo generate <SUBJECT>

    这会生成一个名为 `SUBJECT_test.go` 的文件。如果你不指定 SUBJECT ，它会生成一个名为 `PACKAGE_test.go` 的文件，PACKAGE 是当前目录的名称。

默认地，这些生成器会点引用（dot-import）Ginkgo 和 Gomega。想避免点导入，你可以传入  `--nodot`  到两个子命令。详情请看 [下一章](#避免点导入)

> 注意，你不是必须使用这两个生成器。他们是方便你快速初始化。

### 避免点导入

Ginkgo 和 Gomega 提供了一个 DSL ，而且，默认地 `ginkgo bootstrap` 和 `ginkgo generate` 命令使用点导入导入两个包到顶层命名空间。
有少许确定的情况，你需要避免点导入。例如，你的代码可能定义了与 Ginkgo 或 Gomega 方法冲突的方法名。这中情况下，你可以将你的代码导入到自己的命名空间（换言之，移除导入你的包签名的 `.`）。或者，你可以移除 Ginkgo 或 Gomega 签名的 `.`。后者会导致你一直要在  `Describe` 和 `It` 前面加 `ginkgo.` ，并且你的 `Expect` 和 `ContainSubstring ` 前面也都要加 `gomega.` 。
然而，这是第三个 ginkgo CLI 提供的选项。如果你需要（或想要）避免点导入你可以：

    ginkgo bootstrap --nodot

和

    ginkgo generate --nodot <filename>

这会创建一个引导文件，明确地在顶级命名空间，导入所有 Ginkgo 和 Gomega 的导出标识符。这出现在你引导文件的地步，生成的代码就像这样：
```go
import (
    github.com/onsi/ginkgo
    ...
)

...

// Declarations for Ginkgo DSL
var Describe = ginkgo.Describe
var Context = ginkgo.Context
var It = ginkgo.It
// etc...
```

这允许你使用 `Describe`, `Context`, 和 `It`写测试，而不用添加 `ginkgo.`前缀。关键地，它同时允许你冲定义任何冲突的标识符（或组织你自己的语意）。例如：

```go
var _ = ginkgo.Describe
var When = ginkgo.Context
var Then = ginkgo.It
```

这会避免导入`Describe`，并会将`Context` 和 `It` 重命名为 `When` 和 `Then`。
当新匹配库被添加到 Gomega ，你需要更新这些导入的标识符。你可以这样，进入包含引导文件的目录并运行：

    ginkgo nodot

这会更新导入，保留你提供的重命名。

### 转换已存在的测试

如果你有一个 XUnit 测试套件，而且你想把它转化为 Ginkgo 套件，你可以使用  `ginkgo convert` 命令：

    ginkgo convert github.com/your/package

这会生成一个 Ginkgo 引导文件，转化所有 XUnit 风格  `TestX...(t *testing.T)`  为简单（平坦）的 Ginkgo 测试。它同时将你代码中的 `GinkgoT()` 替换为  `*testing.T`  。  `ginkgo convert`  一般第一次就能正确转换，但事后你可能需要微调一下测试。
同时： `ginkgo convert`  会**覆盖 **你的测试文件，因此确保你尝试 `ginkgo convert` 之前，已经没有未提交的修改了。
`ginkgo convert` 是[Tim Jarratt](https://github.com/tjarratt) 的主意。
### 其它子命令

- 将当前目录(和子目录)写入代码的重点测试设为普通测试：

        $ ginkgo unfocus

- 查看帮助：

        $ ginkgo help

    查看特定子目录的帮助：

        $ ginkgo help <COMMAND>

- 获取当前 Ginkgo 的版本：

        $ ginkgo version

---

## 基准测试

Ginkgo 允许你使用Measure块来测量你的代码的性能。Measure块可以运行在任何It块可以运行的地方--每一个Meature生成一个规格。传递给Measure的闭包函数必须使用Benchmarker参数。Benchmarker用于测量运行时间并记录任意数值。你也必须在该闭包函数之后传递一个整型参数给Measure，它表示Measure将执行的你的代码的样本数。例如：

```go
Measure("it should do something hard efficiently", func(b Benchmarker) {
    runtime := b.Time("runtime", func() {
        output := SomethingHard()
        Expect(output).To(Equal(17))
    })

    Ω(runtime.Seconds()).Should(BeNumerically("<", 0.2), "SomethingHard() shouldn't take too long.")

    b.RecordValue("disk usage (in MB)", HowMuchDiskSpaceDidYouUse())
}, 10)
```

它将联合“runtime” 和 “disk usage”的数据把这个闭包函数运行10次。然后， Ginkgo的reporter将会打印出每一个包含简单统计的指标的总结：

    • [MEASUREMENT]
    Suite
        it should do something hard efficiently

        Ran 10 samples:
        runtime:
          Fastest Time: 0.01s
          Slowest Time: 0.08s
          Average Time: 0.05s ± 0.02s

        disk usage (in MB):
          Smallest: 3.0
           Largest: 5.2
           Average: 3.9 ± 0.4

通过使用Measure, 你可以编写富有表现力，探索性的规格来测量你的代码各个部分的性能（或者外部组件，如果你在使用Ginkgo来编写集成测试）。在收集数据时，你可以保留Measure规格以监控性能，如果组件开始变得缓慢和臃肿，则套件会失败。

`Measures` 和 `Its` 可以在同一个测试套件中使用。如果你只想运行 `Its` ，你可以传递 `--skipMeasurements` 标签给 Ginkgo.

> 您还可以使用 `PMeasure` 和 `XMeasure` 将 `Measures` 标记为待处理，或者将它们与 `FMeasure` 一起使用。

### 测量时间

传递到你的闭包函数的Benchmarker提供了

```go
Time(name string, body func(), info ...Interface{}) time.Duration
```

方法。Times运行传入的body函数，并且记录和返回它的运行时。对每个样本的测量值进行汇总和计算一些简单的统计数据。在传入的name下的规格输出中会显示这些统计数据。注意，在Measure节点的作用域内name必须是唯一的。
你还可以通过可选的info参数传递任意信息。它将会和Time测量的聚合运行时一起传递给reporter。默认reporter使用info的字符串表示形式，但是你也可以编写一个定制的reporter来执行更架构化的东西。例如，您可能会运行相同代码的多个测量，但会在运行时更改某些参数。您可以在info中对该参数的值进行编码，然后让自定义报告器使用该info和Ginkgo提供的统计信息来生成CSV文件 - 甚至可能是图表。

如果你想断言body在某个阈值时间内运行，你可以对Time的返回值进行断言。
### 记录任意值

Benchmarker 也提供

```go
RecordValue(name string, value float64, info ...Interface{})
```

方法。RecordValue允许你记录任意数字化的数值。聚合这些结果并计算一些简单的统计数据。这些统计信息显示在您传入name下的规格输出中。注意，在Measure节点的作用域内name必须是唯一的。

可选的info参数可用于将结构化数据传递给自定义报告器。请参阅上面的[测量时间](#测量时间)以获取更多详

---

## 共享示例模式

Ginkgo对共享示例（也称为共享行为）没有任何显式支持，但是您可以使用一些模式来复用套件中的测试。

### 本地作用域的共享行为

经常有这种情况，有一个套件，包含了相同断言行为的不同 `Context` ，但其中有相同的 `It` 。这些 `Context` 唯一的不同，就是他们各自`BeforeEach`中的初始化。与其在 `Context` 重复相同 `It` ，这里提供两个方法来提取代码，避免重复。

#### 模式 1 ：提取定义共享`It` 的函数

在这，我们会拉取一个包含`Context`的相同闭包中的函数。该函数定义这些 `Context` 中相同的 `It` 。例如：

```go
Describe("my api client", func() {
    var client APIClient
    var fakeServer FakeServer
    var response chan APIResponse

    BeforeEach(func() {
        response = make(chan APIResponse, 1)
        fakeServer = NewFakeServer()
        client = NewAPIClient(fakeServer)
        client.Get("/some/endpoint", response)
    })

    Describe("failure modes", func() {
        AssertFailedBehavior := func() {
            It("should not include JSON in the response", func() {
                Ω((<-response).JSON).Should(BeZero())
            })

            It("should not report success", func() {
                Ω((<-response).Success).Should(BeFalse())
            })
        }

        Context("when the server does not return a 200", func() {
            BeforeEach(func() {
                fakeServer.Respond(404)
            })

            AssertFailedBehavior()
        })

        Context("when the server returns unparseable JSON", func() {
            BeforeEach(func() {
                fakeServer.Succeed("{I'm not JSON!")
            })

            AssertFailedBehavior()
        })

        Context("when the request errors", func() {
            BeforeEach(func() {
                fakeServer.Error(errors.New("oops!"))
            })

            AssertFailedBehavior()
        })
    })
})
```
注意，`AssertFailedBehavior` 函数在`Context` 的 body 中被调用。`It` 在该外部容器中的函数中定义。因为函数共享闭包作用域，我们不需要传入`response` 通道。 

你可以放入任意 `It`  到上面的共享行为 `AssertFailedBehavior` 中，并且甚至可以在 `AssertFailedBehavior` 中连同 `Context` 嵌套 `It`。尽管这并不总是一个 有效地优化(DRY don't repeat yourself) 测试套件的好方法，但你认为它合适的时候，这个模式让你能这样做。这个方法的缺点之一，你不能聚焦或待定一个共享行为组，或者组中的`examples` 或`contexts`。换句话说，你不能直接使用 `FAssertFailedBehavior`和 `XAssertFailedBehavior`。

#### 模式 2：提取返回闭包的函数，并将其传入`It`

要理解这个模式，我们重做上述案例：

```go
Describe("my api client", func() {
    var client APIClient
    var fakeServer FakeServer
    var response chan APIResponse

    BeforeEach(func() {
        response = make(chan APIResponse, 1)
        fakeServer = NewFakeServer()
        client = NewAPIClient(fakeServer)
        client.Get("/some/endpoint", response)
    })

    Describe("failure modes", func() {
        AssertNoJSONInResponse := func() func() {
            return func() {
                Ω((<-response).JSON).Should(BeZero())
            }
        }

        AssertDoesNotReportSuccess := func() func() {
            return func() {
                Ω((<-response).Success).Should(BeFalse())
            }
        }
        Context("when the server does not return a 200", func() {
            BeforeEach(func() {
                fakeServer.Respond(404)
            })

            It("should not include JSON in the response", AssertNoJSONInResponse())
            It("should not report success", AssertDoesNotReportSuccess())
        })

        Context("when the server returns unparseable JSON", func() {
            BeforeEach(func() {
                fakeServer.Succeed("{I'm not JSON!")
            })

            It("should not include JSON in the response", AssertNoJSONInResponse())
            It("should not report success", AssertDoesNotReportSuccess())
        })

        Context("when the request errors", func() {
            BeforeEach(func() {
                fakeServer.Error(errors.New("oops!"))
            })

            It("should not include JSON in the response", AssertNoJSONInResponse())
            It("should not report success", AssertDoesNotReportSuccess())
        })
    })
})
```

注意，这个解决方案仍然很简洁，尤其因为每个 `Context` 只有两个共享的 `It` 。这里多了一点重复，但是它也更明确了一点。主要的好处是，你可以聚焦和待定一个在单独 `Context` 中的单独 `It` 。

### 全局共享行为

在共享行为只在一个固定作用域使用的时候，上面的模式很好用。如果你想要构建一个模式，但是涉及跨文件的话，你需要微调一下该模式来传递输入。我们扩展上面两个例子来展示怎么做：

#### 模式 1 ：

```go
package sharedbehaviors

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

type FailedResponseBehaviorInputs struct {
    response chan APIResponse
}

func SharedFailedResponseBehavior(inputs *FailedResponseBehaviorInputs) {
    It("should not include JSON in the response", func() {
        Ω((<-(inputs.response)).JSON).Should(BeZero())
    })

    It("should not report success", func() {
        Ω((<-(inputs.response)).Success).Should(BeFalse())
    })
}
```

#### 模式 2

```go
package sharedbehaviors

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

type FailedResponseBehaviorInputs struct {
    response chan APIResponse
}

func AssertNoJSONInResponese(inputs *FailedResponseBehaviorInputs) func() {
    return func() {
        Ω((<-(inputs.response)).JSON).Should(BeZero())
    }
}

func AssertDoesNotReportSuccess(inputs *FailedResponseBehaviorInputs) func() {
    return func() {
        Ω((<-(inputs.response)).Success).Should(BeFalse())
    }
}
```

共享行为的用户必须生成并放入 `FailedResponseBehaviorInputs` ，再将它传入`SharedFailedResponseBehavior` 或 `AssertNoJSONInResponese` 和 `AssertDoesNotReportSuccess` 。为什么这样做？两个原因：

1. 将输入变量封装到结构体中（就像 `FailedResponseBehaviorInputs` ），允许你清楚地规定 sepc 和共享行为间的合约。共享行为需要这些输入才能工作。
2. 更重要的是，像`response`通道的输入，一般在 `BeforeEach` 块中创建或赋值。然而，共享行为函数必须在容器内调用，并且不能访问`BeforeEach`中指定的变量，因为此时 `BeforeEach` 还没运行。要解决这个问题，我们实例化一个 `FailedResponseBehaviorInputs` ，并将一个指向它的指针传入共享行为函数。在 `BeforeEach` 中，我们操作 `FailedResponseBehaviorInputs` 的字段，确保他们的值能和共享行为生成的 `It` 交互。
下面是点导入 `sharedbehaviors` 包后，调用测试的代码（简单起见，我们两个模式放入到一个例子中）：

```go
Describe("my api client", func() {
    var client APIClient
    var fakeServer FakeServer
    var response chan APIResponse
    sharedInputs := FailedResponseBehaviorInputs{}

    BeforeEach(func() {
        sharedInputs.response = make(chan APIResponse, 1)
        fakeServer = NewFakeServer()
        client = NewAPIClient(fakeServer)
        client.Get("/some/endpoint", sharedInputs.response)
    })

    Describe("failure modes", func() {
        Context("when the server does not return a 200", func() {
            BeforeEach(func() {
                fakeServer.Respond(404)
            })

            // Pattern 1
            SharedFailedResponseBehavior(&sharedInputs)
        })

        Context("when the server returns unparseable JSON", func() {
            BeforeEach(func() {
                fakeServer.Succeed("{I'm not JSON!")
            })

            // Pattern 2
            It("should not include JSON in the response", AssertNoJSONInResponse(&sharedInputs))
            It("should not report success", AssertDoesNotReportSuccess(&sharedInputs))
        })
    })
})
```

---

## Ginkgo与持续集成

Ginkgo附带了许多标签，您可能希望在持续集成环境运行时打开这些标签。建议如下：

    ginkgo -r --randomizeAllSpecs --randomizeSuites --failOnPending --cover --trace --race --progress

- `-r` 会递归运行目标文件夹下的所有测试套件。
- `--randomizeAllSpecs` 和 `--randomizeSuites` 会使套件中的 sepc 和不同套件的运行顺序随机。这对于识别测试污染很好用。你总可以重现该顺序通过设置对应 `--seed`  参数。
- `--failOnPending` 会在有待定的 spec 的情况下使套件失败。 (通常，这些测试不该被提交，但应该提示正在运行的测试).
- `--cover` 生成 `.coverprofile`文件，并未每个测试套件做覆盖率统计。
- `--trace` 报告会为每个失败打印全栈跟踪日志。这使得持续集成的日志更易与调试。
- `--race` 运行测试并打开竞态检测器。
- `--progress` 输出测试过程到 GinkgoWriter. 更利于识别失败发生的地方。

不推荐你在持续集成的时候使用 `-p`运行并行测试。很多持续集成系统运行在很多核的机器（例如32 节点）。这么大规模的并行常常花费更多的测试时间（尤其是这种情况，你的测试可能运行在一些限制 cup 共享的容器中：你实际上无法完全使用 32 个核）。要在持续集成中使用并行，更好的方式是使用`-nodes`提供一个显式的并行节点数。

### .travis.yml 的案例

对于 Travis 持续集成，你可能使用类似下面的东西：

    language: go
    go:
        - 1.9
        - tip

    install:
        - go get -v github.com/onsi/ginkgo/ginkgo
        - go get -v github.com/onsi/gomega
        - go get -v -t ./...
        - export PATH=$PATH:$HOME/gopath/bin

    script: ginkgo -r --randomizeAllSpecs --randomizeSuites --failOnPending --cover --trace --race --compilers=2

注意，我们添加了  `--compilers=2` 。这解决了一个问题，即 Travis 会在 Ginkgo 请求过多的时候终止进程。默认地，Ginkgo 会运行 `runtime.NumCPU()`  个编译器，对应在 Travis 就是 32 个编译器。类似的，如果你在 Travis 上运行并行测试，确保指定  `--nodes=N`  而不是 `-p`。


---

## 插件


Ginkgo 装载了核心 DSL(Domain Specific Language  领域专属语言) 的插件。可以通过点导入来增加 Ginkgo 的默认 DSL。当前只有一个扩展：表格插件。

### 表格驱动测试

 [表格](https://godoc.org/github.com/onsi/ginkgo/extensions/table) 为写表格驱动测试提供了富有表现力的 DSL 。

| 注意：如果你`vendor`目录中有 Ginkgo，确保添加包`github.com/onsi/ginkgo/extensions/table` 到 `vendor`。详情请看 [issue 234](https://github.com/onsi/ginkgo/issues/234#issuecomment-196645747)| | :——————– |

使用数据结构和 for 循环写你自己的表格驱动测试是简单的，DSL 层使书写和管理表格驱动测试变得尤其简单。

例如：

```go
package table_test

import (
    . "github.com/onsi/ginkgo/extensions/table"

    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

var _ = Describe("Math", func() {
    DescribeTable("the > inequality",
        func(x int, y int, expected bool) {
            Expect(x > y).To(Equal(expected))
        },
        Entry("x > y", 1, 0, true),
        Entry("x == y", 0, 0, false),
        Entry("x < y", 0, 1, false),
    )
})
```

> 这个例子中，我们点导入的 table 插件。这不是必须的，但这样使 DSL 更容易交互。

我们分析下， `DescribeTable` 带有一个描述 ， 一个运行每个测试的函数，还有一个 entry 集合。

传入`DescribeTable`的函数可以接收任意参数。这些参数被传入每个 `Entry`的参数会被传入函数（类型不匹配的话，会导致运行时 panic ）


每个 `Entry` 构建一个  `TableEntry` 传入 `DescribeTable`。`TableEntry` 的组成元素是：一个描述（ `Entry` 的第一个调用），一个`DescribeTable` 注册被传入函数的任意参数集。

理解表格的生命周期很重要。 `table` 包简单包装了 Ginkgo 的 DSL。`DescribeTable` 生成了单个 Ginkgo `Describe`， `Describe` 中 每个 `Entry` 生成一个 Ginkgo `It`。这都在测试运行前发生（在"构建测试树的时候"）。结果就是，表格发展了一定数量的 `It`（每个 `Entry` 一个），它们都服从所有 Ginkgo 测试运行的语义：`It` 在多节点间可以被随机化和并行化。

需要明确的是，上述测试*完全*等同于：

```go
package table_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

var _ = Describe("Math", func() {
    Describe("the > inequality",
        It("x > y", func() {
            Expect(1 > 0).To(Equal(true))
        })

        It("x == y", func() {
            Expect(0 > 0).To(Equal(false))
        })

        It("x < y", func() {
            Expect(0 > 1).To(Equal(false))
        })
    )
})
```

你应该知道了 Ginkgo 的测试生命周期，尤其通过 [动态生成测试](#动态生成测试)和使用`DescribeTable`。

#### 聚焦并待定表格和Entry

这是很棒的部分。整个表格都可以被聚焦或标记为怪气通过简单的将`DescribeTable` 置换为 `FDescribeTable`
（聚焦） 或 `PDescribeTable` （待定）。

类似的，单个 entry 可以使用 `FEntry` and `PEntry` 被聚焦或待定。调试测试的时候这尤其有用。

#### 管理复杂的参数

当你传递任意参数到 `Entry` 的时候，很容易就使得测试例子变得难以理解。对于更复杂的表，更合理的方式是，定义一个新的类型来传递。例如：
```go
package table_test

import (
    . "github.com/onsi/ginkgo/extensions/table"

    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)

var _ = Describe("Substring matching", func() {
    type SubstringCase struct {
        String    string
        Substring string
        Count     int
    }

    DescribeTable("counting substring matches",
        func(c SubstringCase) {
            Ω(strings.Count(c.String, c.Substring)).Should(BeNumerically("==", c.Count))
        },
        Entry("with no matching substring", SubstringCase{
            String:    "the sixth sheikh's sixth sheep's sick",
            Substring: "emir",
            Count:     0,
        }),
        Entry("with one matching substring", SubstringCase{
            String:    "the sixth sheikh's sixth sheep's sick",
            Substring: "sheep",
            Count:     1,
        }),
        Entry("with many matching substring", SubstringCase{
            String:    "the sixth sheikh's sixth sheep's sick",
            Substring: "si",
            Count:     3,
        }),
    )
})
```

注意，这个模式使用了相同的 DSL，这是一个简单的方法来管理 `Entry` 间的参数流和`DescribeTable` 注册的回调函数。

#### 自定义 Entry 描述

有一些场景，描述中的一部分是参数，这样有助于理解测试的意图。`Entry`支持传递一个接收`Entry`参数并返回相关描述的帮助函数，而不是在每个描述中添加参数。例如：

```go
package table_test

import (
    . "github.com/onsi/ginkgo/extensions/table"

    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"
)
var _ = Describe("TableWithParametricDescription", func() {
    describe := func(desc string) func(int, int, bool) string {
        return func(x, y int, expected bool) string {
            return fmt.Sprintf("%s x=%d y=%d expected:%t", desc, x, y, expected)
        }
    }

    DescribeTable("a simple table",
        func(x int, y int, expected bool) {
            Ω(x > y).Should(Equal(expected))
        },
        Entry(describe("x > y"), 1, 0, true),
        Entry(describe("x == y"), 0, 0, false),
        Entry(describe("x < y"), 0, 1, false),
    )
}

```

这个例子中，`Entry` 中每个 `It` 的描述都是通过传入 `Entry` 的 `describe`函数生成的。

---

## 编写自定义报告器

因为 Ginkgo 的默认报告器提供了全面的功能， Ginkgo 很容易同时写和运行多个自定义报告器。这有很多使用案例。你能实现一个自定义报告器使你的持续集成方案支持一个特殊的输出格式，或者你能实现一个自定义报告器从Ginkgo `Measure` 节点[聚合数据](#measuring-time) 和制造 HTML 或 CSV 报告（或者甚至图表！）。

在 Ginkgo 中，一个报告器必须满足 `Reporter` 接口：

```go
type Reporter interface {
    SpecSuiteWillBegin(config config.GinkgoConfigType, summary *types.SuiteSummary)
    BeforeSuiteDidRun(setupSummary *types.SetupSummary)
    SpecWillRun(specSummary *types.SpecSummary)
    SpecDidComplete(specSummary *types.SpecSummary)
    AfterSuiteDidRun(setupSummary *types.SetupSummary)
    SpecSuiteDidEnd(summary *types.SuiteSummary)
}
```

方法的名字应该能是自解释的。为了使你获得合理可用的数据，确保深入理解  `SuiteSummary` 和 `SpecSummary` 。如果你写了一个自定义报告器，用于获取 `Measure` 节点产生的基准测试数据，你会想看看  `ExampleSummary.Measurements` 提供的结构体 `ExampleMeasurement` 。


一旦你创建了自定义报告器，你可能要替换你测试套件中的`RunSpecs`命令，来传入该实例到 Ginkgo，要么这样：

```go
RunSpecsWithDefaultAndCustomReporters(t *testing.T, description string, reporters []Reporter)
```

要么这样

```go
RunSpecsWithCustomReporters(t *testing.T, description string, reporters []Reporter)
```

`RunSpecsWithDefaultAndCustomReporters` 会运行你的自定义报告器和 Ginkgo 默认报告器。`RunSpecsWithCustomReporters` 只会运行你的自定义报告器。

如果你希望运行并行测试，你不应该使用 `RunSpecsWithCustomReporters`，因为默认报告器是 ginkgo CLI 测试输出流的重要角色。

---
## 第三方集成

### 使用其它匹配库

大多数匹配库接受  `*testing.T`  对像。不幸的是，这是个具体类型 ，因此难以和 Ginkgo 兼容。

通常，用满足  `*testing.T` 的接口替换此类库中的  `*testing.T` 并不困难。例如[testify](https://github.com/stretchr/testify) ，通过接口接收 `t`。这种情况下你可以传递  `GinkgoT()`。这将产生一个模仿  `*testing.T` 的对像，并且能直接和 Ginkgo 通信。

For example, to get testify working:

例如，这样使用 testify ：

```go
package foo_test

import (
    . "github.com/onsi/ginkgo"

    "github.com/stretchr/testify/assert"
)

var _ = Describe(func("foo") {
    It("should testify to its correctness", func(){
        assert.Equal(GinkgoT(), foo{}.Name(), "foo")
    })
})
```

> 注意，从 传递来自 Ginkgo 的引导函数  `Test...()`  的 `*testing.T` 的话，会导致套件遇到第一个测试失败时停止。不要这样做。你需要将失败传递给 Ginkgo 的单个（全局） `Fail`  函数。

### 集成 Gomock

Ginkgo 没有提供模拟/桩（ mocking/stubbing）框架。作者认为，mocks 和 stub 能完全被依赖注入和注入 Go 接口来替代。然后，将实际依赖注入到生产代码中，并在测试代码中注入伪造的依赖。建立和维护此类伪造品往往很简单，并且可以比模拟进行更清晰，更具表现力的测试。

话虽如此，使用诸如[Gomock](https://code.google.com/p/gomock/) 之类的模拟框架相对简单。`GinkgoT()`  实现了 Gomock 的 `TestReporter` 的接口。使用方法如下（举例）：

```go
import (
    "code.google.com/p/gomock/gomock"

    . github.com/onsi/ginkgo
    . github.com/onsi/gomega
)

var _ = Describe("Consumer", func() {
    var (
        mockCtrl *gomock.Controller
        mockThing *mockthing.MockThing
        consumer *Consumer
    )

    BeforeEach(func() {
        mockCtrl = gomock.NewController(GinkgoT())
        mockThing = mockthing.NewMockThing(mockCtrl)
        consumer = NewConsumer(mockThing)
    })

    AfterEach(func() {
        mockCtrl.Finish()
    })

    It("should consume things", func() {
        mockThing.EXPECT().OmNom()
        consumer.Consume()
    })
})
```

当使用 Gomock时，你可能想要使用 `-trace` 参数运行 `ginkgo`，以打印失败的堆栈跟踪信息，这将帮你追溯代码中无效调用的发生位置。

### 生成 JUnit XML 的输出。

Ginkgo 提供了一个 [自定义报告器](https://onsi.github.io/ginkgo/#writing-custom-reporters) 来生成 JUnit 兼容的 XML 输出。这是一个示例引导文件，该文件实例化了JUnit报告程序并将其传递给测试运行器：



```go
package foo_test

import (
    . "github.com/onsi/ginkgo"
    . "github.com/onsi/gomega"

    "github.com/onsi/ginkgo/reporters"
    "testing"
)

func TestFoo(t *testing.T) {
    RegisterFailHandler(Fail)
    junitReporter := reporters.NewJUnitReporter("junit.xml")
    RunSpecsWithDefaultAndCustomReporters(t, "Foo Suite", []Reporter{junitReporter})
}
```

这会在包含你测试的目录中生成一个名为 “junit.xml” 的文件。这个 xml 文件兼容最新版本的 Jenkins JUnit 插件。

如果你想要并行运行你的测试，你需要让你的 JUnit xml 文件带有并行节点号。你可以这样做：

    junitReporter := reporters.NewJUnitReporter(fmt.Sprintf("junit_%d.xml", config.GinkgoConfig.ParallelNode))

注意，你需要导入 `fmt` 和  `github.com/onsi/ginkgo/config` ，以使其正常工作。这会为每个并行节点生成一个 xml 文件。 Jenkins JUnit 插件（举例） 会自动聚合所有这些文件的数据。
