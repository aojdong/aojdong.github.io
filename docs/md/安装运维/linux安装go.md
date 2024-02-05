## linux安装go

可以直接下载的发布包解压使用

- `wget` 下载对应版本和架构的[发布包](https://golang.google.cn/dl/)
```bash
sudo wget https://golang.google.cn/dl/go1.21.1.linux-amd64.tar.gz --no-check-certificate
```

> 可以本地下载后上传至 linux 环境

- `tar` 解压文件到指定目录
```bash
tar xfz go1.21.1.linux-amd64.tar.gz -C /usr/local
```

- 配置全局环境变量`/etc/profile`，系统启动时被调用
```bash
echo 'export GOROOT=/usr/local/go
export GOPATH=$HOME/gowork
export GOBIN=$GOPATH/bin
export PATH=$GOPATH:$GOBIN:$GOROOT/bin:$PATH' | sudo tee -a /etc/profile

source /etc/profile
```

- 配置用户级别环境变量`~/.bashrc`，用户打开新的终端时被调用
```bash
echo "source /etc/profile" >> ~/.bashrc
```

- 配置 Go 代理
```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

- 安装结果
```bash
go env        // 查看 go 环境运行变量
go version    // 查看当前 go 版本
which go      // 查看当前 go 路径
```

