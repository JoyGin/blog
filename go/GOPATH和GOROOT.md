###  GOPATH 和 GOROOT
GOROOT：这是Go语言安装的路径。当你安装Go语言后，GOROOT将指向你的Go安装目录，这个目录下包含了Go的源代码、预编译的标准库和其他一些命令工具等。如果你是通过包管理器（如apt、yum等）安装的Go，那么GOROOT通常会被设置为/usr/local/go。

GOPATH：这是你的工作目录，你的Go代码、依赖包、编译后的二进制文件等都会存放在这里。你可以把GOPATH理解为Go语言的工作空间，它是Go语言的重要组成部分。在Go 1.8版本之前，GOPATH是必须要设置的，但在Go 1.8及以后的版本中，如果你没有设置GOPATH，那么它默认就是你的用户目录下的go目录（如Linux下的$HOME/go，Windows下的%USERPROFILE%\go）。