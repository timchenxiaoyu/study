# iptables

## iptables

NF_IP_PRE_ROUTING: 接收的数据包刚进来，还没有经过路由选择，即还不知道数据包是要发给本机还是其它机器。

NF_IP_LOCAL_IN: 已经经过路由选择，并且该数据包的目的IP是本机，进入本地数据包处理流程。

NF_IP_FORWARD: 已经经过路由选择，但该数据包的目的IP不是本机，而是其它机器，进入forward流程。

NF_IP_LOCAL_OUT: 本地程序要发出去的数据包刚到IP层，还没进行路由选择。

NF_IP_POST_ROUTING: 本地程序发出去的数据包，或者转发（forward）的数据包已经经过了路由选择，即将交由下层发送出去。

```go
        |
                                    | Incoming             ++---------------------++
                                    ↓                      || raw                 ||
                           +-------------------+           || connection tracking ||
                           | NF_IP_PRE_ROUTING |= = = = = =|| mangle              ||
                           +-------------------+           || nat (DNAT)          ||
                                    |                      ++---------------------++
                                    |
                                    ↓                                                ++------------++
                           +------------------+                                      || mangle     ||
                           |                  |         +----------------+           || filter     ||
                           | routing decision |-------->| NF_IP_LOCAL_IN |= = = = = =|| security   ||
                           |                  |         +----------------+           || nat (SNAT) ||
                           +------------------+                 |                    ++------------++
                                    |                           |
                                    |                           ↓
                                    |                  +-----------------+
                                    |                  | local processes |
                                    |                  +-----------------+
                                    |                           |
                                    |                           |                    ++---------------------++
 ++------------++                   ↓                           ↓                    || raw                 ||
 || mangle     ||           +---------------+          +-----------------+           || connection tracking ||
 || filter     ||= = = = = =| NF_IP_FORWARD |          | NF_IP_LOCAL_OUT |= = = = = =|| mangle              ||
 || security   ||           +---------------+          +-----------------+           || nat (DNAT)          ||
 ++------------++                   |                           |                    || filter              ||
                                    |                           |                    || security            ||
                                    ↓                           |                    ++---------------------++
                           +------------------+                 |
                           |                  |                 |
                           | routing decision |<----------------+
                           |                  |
                           +------------------+
                                    |
                                    |
                                    ↓
                           +--------------------+           ++------------++
                           | NF_IP_POST_ROUTING |= = = = = =|| mangle     ||
                           +--------------------+           || nat (SNAT) ||
                                    |                       ++------------++
                                    | Outgoing
                                    ↓
```
以NF_IP_PRE_ROUTING为例，数据包到了这个点之后，会先执行raw表中PREROUTING(chain)里的rule，然后执行connection tracking，
接着再执行mangle表中PREROUTING(chain)里的rule，最后执行nat (DNAT)表中PREROUTING(chain)里的rule。

以filter表为例，它只能注册在NF_IP_LOCAL_IN、NF_IP_FORWARD和NF_IP_LOCAL_OUT上，所以它只支持INPUT、FORWARD和OUTPUT这三个chain。

以收到目的IP是本机的数据包为例，它的传输路径为：NF_IP_PRE_ROUTING -> NF_IP_LOCAL_IN，那么它首先要依次经过NF_IP_PRE_ROUTING上
注册的raw、connection tracking 、mangle和nat (DNAT)，然后经过NF_IP_LOCAL_IN上注册的mangle、filter、security和nat (SNAT)。

## 连接追踪（Connection Tracking）
Connection Tracking发生在NF_IP_PRE_ROUTING和NF_IP_LOCAL_OUT这两个地方，一旦开启该功能，Connection Tracking模块将会追踪每个
数据包（被raw表中的rule标记过的除外），维护所有的连接状态，然后这些状态可以供其它表中的rule引用，
用户空间的程序也可以通过/proc/net/ip_conntrack来获取连接信息。下面是所有的连接状态：
NEW: 当检测到一个不和任何现有连接关联的新包时，如果该包是一个合法的建立连接的数据包（比如TCP的sync包或者任意的UDP包），
一个新的连接将会被保存，并且标记为状态NEW。

ESTABLISHED: 对于状态是NEW的连接，当检测到一个相反方向的包时，连接的状态将会由NEW变成ESTABLISHED，表示连接成功建立。
对于TCP连接，意味着收到了一个SYN/ACK包， 对于UDP和ICMP，任何反方向的包都可以。

RELATED: 数据包不属于任何现有的连接，但它跟现有的状态为ESTABLISHED的连接有关系，对于这种数据包，将会创建一个新的连接，
且状态被标记为RELATED。这种连接一般是辅助连接，比如FTP的数据传输连接（FTP有两个连接，另一个是控制连接），或者和某些连接有关的ICMP报文。

INVALID: 数据包不和任何现有连接关联，并且不是一个合法的建立连接的数据包，对于这种连接，将会被标记为INVALID，一般这种都是垃圾数据包，
比如收到一个TCP的RST包，但实际上没有任何相关的TCP连接，或者别的地方误发过来的ICMP包。

UNTRACKED: 被raw表里面的rule标记为不需要tracking的数据包，这种连接将会标记成UNTRACKED。