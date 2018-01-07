## GLIDE

### 安装
```go
 go install github.com/Masterminds/glide
```

### 初始化
到项目目录下面执行
```go
cd $GOPATH/src/foo
glide init
```
会生成gilde.yaml

### 安装依赖
```go
glide install
```

### 升级版本
发过程中如果需要使用新版代码，可以执行这个命令： 修改一下glide.yaml中的一个Package版本
然后执行
```go
glide up
```

### 设置mirror
在国内这个是必须的，
可以通过命令行的方式设置
```go
glide mirror set https://github.com/example/foo https://git.example.com/example/foo.git --vcs git
 glide mirror set https://github.com/example/foo file:///path/to/local/repo --vcs git
```
他们会在$HOME/.glide/mirrors.yaml，生成相应配置，你也可以直接修改这个文件，下面是常用的mirror
```go
repos:
- original: https://golang.org/x/crypto
  repo: https://github.com/golang/crypto
  vcs: git
- original: https://golang.org/x/crypto/bcrypt
  repo: https://github.com/golang/crypto
  vcs: git
  base: golang.org/x/crypto
- original: https://golang.org/x/crypto/acme/autocert
  repo: https://github.com/golang/crypto
  vcs: git
  base: golang.org/x/crypto
- original: https://golang.org/x/image
  repo: https://github.com/golang/image
  vcs: git
- original: https://golang.org/x/mobile
  repo: https://github.com/golang/mobile
  vcs: git
- original: https://golang.org/x/net
  repo: https://github.com/golang/net
  vcs: git
- original: https://golang.org/x/net/context
  repo: https://github.com/golang/net
  vcs: git
  base: golang.org/x/net
- original: https://golang.org/x/net/html
  repo: https://github.com/golang/net
  vcs: git
  base: golang.org/x/net
- original: https://golang.org/x/net/http2
  repo: https://github.com/golang/net
  vcs: git
  base: golang.org/x/net
- original: https://golang.org/x/sys
  repo: https://github.com/golang/sys
  vcs: git
- original: https://golang.org/x/sys/unix
  repo: https://github.com/golang/sys
  vcs: git
  base: golang.org/x/sys
- original: https://golang.org/x/text
  repo: https://github.com/golang/text
  vcs: git
- original: https://golang.org/x/text/unicode/norm
  repo: https://github.com/golang/text
  vcs: git
  base: golang.org/x/text
- original: https://golang.org/x/text/secure/bidirule
  repo: https://github.com/golang/text
  vcs: git
  base: golang.org/x/text
- original: https://golang.org/x/tools
  repo: https://github.com/golang/tools
  vcs: git
- original: https://golang.org/x/oauth2
  repo: https://github.com/golang/oauth2
  vcs: git
- original: https://google.golang.org/appengine/urlfetch
  repo: https://github.com/golang/appengine
  base: google.golang.org/appengine
  vcs: git
- original: https://google.golang.org/appengine
  repo: https://github.com/golang/appengine
  vcs: git
- original: https://google.golang.org/grpc
  repo: https://github.com/grpc/grpc-go
  vcs: git
- original: https://google.golang.org/genproto
  repo: https://github.com/google/go-genproto
  vcs: git
- original: https://google.golang.org/genproto/googleapis/rpc/status
  repo: https://github.com/google/go-genproto
  vcs: git
  base: google.golang.org/genproto


```