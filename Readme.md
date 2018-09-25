[TOC]

### Go 学习

#### 工作空间

* 包含三个子目录
  * `src` 目录包含Go的源文件，它们被组织成包（每个目录都对应一个包）
  * `pkg` 目录包含包对象
  * `bin` 目录包含可执行命令

#### GOPATH环境变量

* 在`~/.profile`中加入

  ```
  export GOPATH=$HOME/gowork
  export PATH=$PATH:$GOPATH/bin
  ```

  * 但是后面直接运行`go install `时候报错，找不到bin

    * 解决：在`~/.profile`中直接加入

      `export GOBIN=$PATH:$GOPATH/bin`

* 执行配置

  `source $HOME/.profile`

#### 创建第一个程序并运行

* 创建相应的包目录`$ mkdir $GOPATH/src/github.com/user/hello`

* 在该目录下创建名为`hello.go`的文件，其代码为

  ```go
  package main

  import "fmt"

  func main() {
  	fmt.Printf("Hello, world.\n")
  }
  ```

* 用go工具构建并安装此程序

  * 系统任何地方都可以运行，go工具会根据GOPATH指定的工作空间，在 `github.com/user/hello` 包内查找源码

    ```
    go install github.com/user/hello
    ```

  * 若在包目录下运行`go install`，可以省略包路径

    ```
    $ cd $GOPATH/src/github.com/user/hello
    $ go install
    ```

    此命令会构建 `hello` 命令，产生一个可执行的二进制文件。 接着它会将该二进制文件作为 `hello`（在 Windows 下则为 `hello.exe`）安装到工作空间的 `bin` 目录中

* 运行程序

  * 输入完整路径运行

    ```
    $ $GOPATH/bin/hello
    Hello, world.
    ```

  * 如果`$GOPATH/bin` 添加到 `PATH` 中了，只需输入该二进制文件名即可

    ```
    $ hello
    Hello, world.
    ```

* 新建本地仓库并将代码推到远程仓库

  * 初始化仓库

    ```
    git init
    ```

  * 添加到缓存区

    ```
    git add *
    ```

  * 添加备注，可以用于版本回退

    ```
    git commit -m "something change"
    ```

  * 关联远程仓库

    ```
    git remote add origin github.com/user/project
    ```

  * 推送代码到远程仓库

    ```
    git push -u origin master
    ```



#### 创建一个库

* 选择包路径（使用 `github.com/user/stringutil`） 并创建包目录：

  ```
  $ mkdir $GOPATH/src/github.com/user/stringutil
  ```

* 在目录中创建名为`reverse.go`文件，内容如下

  ```go
  // stringutil contain a function which is used to handle string 
  package stringutil

  // reverse the string 
  func Reverse(s string) string {
  	r := []rune(s)
  	for i, j := 0, len(r) - 1; i < len(r)/2; i, j = i + 1, j - 1{
  		r[i], r[j] = r[j], r[i]
  	}
  	
  	return string(r)
  }
  ```

* 使用`go build`命令来测试该包的编译

  ```
  $ go build github.com/user/stringutil
  ```

  假如在该包的源码目录，只需执行：

  ```
  $ go build
  ```

  这不会产生输出文件。想要输出的话，必须使用 `go install` 命令，它会将包的对象放到工作空间的 `pkg` 目录中。

* 确认 `stringutil` 包构建完毕后，修改之前创建的`Hello.go`文件使用它

  ```go
  package main

  import (
  	"fmt"

  	"github.com/user/stringutil"
  )

  func main() {
  	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
  }
  ```

* 运行`hello.go`

  无论是安装包还是二进制文件，`go` 工具都会安装它所依赖的任何东西。 因此当我们通过

  ```
  $ go install github.com/user/hello
  ```

  来安装 `hello` 程序时，`stringutil` 包也会被自动安装。

  运行此程序的新版本，你应该能看到一条新的，反向的信息：

  ```
  $ hello
  Hello, Go!
  ```

* 包名

  Go源文件中的第一个语句必须是

  ```
  package 名称
  ```

  这里的 **名称** 即为导入该包时使用的默认名称。 （一个包中的所有文件都必须使用相同的 **名称**。）

  Go的约定是包名为导入路径的最后一个元素：作为 “`crypto/rot13`” 导入的包应命名为 `rot13`。

  可执行命令必须使用 `package main`。

  链接成单个二进制文件的所有包，其包名无需是唯一的，只有导入路径（它们的完整文件名） 才是唯一的。

#### 测试

* Go拥有一个轻量级的测试框架，它由 `go test` 命令和 `testing` 包构成。

* 你可以通过创建一个名字以 `_test.go` 结尾的，包含名为 `TestXXX` 且签名为 `func (t *testing.T)` 函数的文件来编写测试。 测试框架会运行每一个这样的函数；若该函数调用了像 `t.Error` 或 `t.Fail` 这样表示失败的函数，此测试即表示失败。

* 我们可通过创建文件 `$GOPATH/src/github.com/user/stringutil/reverse_test.go` 来为 `stringutil` 添加测试，其内容如下：

  ```go
  package stringutil

  import "testing"

  func TestReverse(t *testing.T) {
  	cases := []struct {
  		in, want string
  	}{
  		{"Hello, world", "dlrow ,olleH"},
  		{"Hello, 世界", "界世 ,olleH"},
  		{"", ""},
  	}
  	for _, c := range cases {
  		got := Reverse(c.in)
  		if got != c.want {
  			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
  		}
  	}
  }
  ```

* 接着使用 `go test` 运行该测试：

  ```
  $ go test github.com/user/stringutil
  ```

  * 同样，若你在包目录下运行 `go` 工具，也可以忽略包路径

    ```
    $ go test
    ```

    ​

#### 学习链接

* [如何使用Go编程](https://go-zh.org/doc/code.html)
* [安装go语言开发环境](https://pmlpml.github.io/ServiceComputingOnCloud/ex-install-go#6%E5%AE%9E%E9%AA%8C%E6%8A%A5%E5%91%8A%E4%B8%8E%E4%BD%9C%E4%B8%9A%E8%A6%81%E6%B1%82)

