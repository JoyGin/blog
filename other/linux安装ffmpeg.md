# Linux 安装 ffmpeg

### Ubuntu
Ubuntu 直接使用 apt 安装即可
```sh
$ sudo apt install -y ffmpeg
```

### CentOS
CentOS 需要使用源码安装
- 下载并解压
```shell
$ wget https://ffmpeg.org/releases/ffmpeg-4.1.tar.bz2
$ tar -xjvf ffmpeg-4.1.tar.bz2
$ cd ffmpeg-4.1
```

- 配置
```shell
$ ./configure
nasm/yasm not found or too old. Use --disable-x86asm for a crippled build.
```

如果提示上面的错误，可以使用 --disable-x86asm 参数略过该配置或者直接安装，然后再重新配置
```shell
$ sudo yum install -y yasm
$ ./configure
```

- 编译
```shell
$ sudo make && sudo make install
```

##### 参考链接
- https://wxnacy.com/2018/11/27/linux-install-ffmpeg/