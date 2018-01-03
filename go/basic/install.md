
## 安装GO

先下载二进制包
https://redirector.gvt1.com/edgedl/go/go$VERSION.$OS-$ARCH.tar.gz
eg:
wget https://redirector.gvt1.com/edgedl/go/go1.8.1.linux-amd64.tar.gz

```go
#解压
sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz

#设置环境变量
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

```
最后source一下使之生效