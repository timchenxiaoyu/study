通过golang去下载文件
```go
package main  

  

import (  

    "fmt"  

    "io"  

    "net/http"  

    "os"  

)  

  

var (  

    url = "http://127.0.0.1:1789/src/qq.exe"  

)  

  

func main() {  

    res, err := http.Get(url)  

    if err != nil {  

        panic(err)  

    }  

    f, err := os.Create("qq.exe")  

    if err != nil {  

        panic(err)  

    }  

    io.Copy(f, res.Body)  

}  

```
