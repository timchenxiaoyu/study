os是golang针对操作系统适配的系统调用的使用的工具包这篇文章主要介绍的File包的使用

### OpenFile
先介绍OpenFile这个方法是因为其他方法都是调用这个方法去完成
```go
import (
    "fmt"
    "os"
    "reflect"
)

func main() {
	f, _ := os.OpenFile("test.txt", os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0777)
	defer f.Close()
	fmt.Println(f.Stat())
}
```
O_RDWR也就是说用读写的权限，O_CREATE如果文件存在则忽略，
不存则创建，O_TRUNC文件存在截取长度为0，就是会覆盖更新
总共的读写权限包括
```go
	O_RDONLY int = syscall.O_RDONLY // 只读
	O_WRONLY int = syscall.O_WRONLY //只写
	O_RDWR   int = syscall.O_RDWR   // 读写
	O_APPEND int = syscall.O_APPEND //追加写
	O_CREATE int = syscall.O_CREAT  // 不存在创建
	O_EXCL   int = syscall.O_EXCL   // 配合 O_CREATE创建文件
	O_SYNC   int = syscall.O_SYNC   // 打开IO同步
	O_TRUNC  int = syscall.O_TRUNC  // 截断打开
```


### Create
Create是创建文件的函数
```go

import (
    "fmt"
    "os"
    "reflect"
)

func main() {
    f, _ := os.Create("test.txt")
    defer f.Close()
    fmt.Println(reflect.ValueOf(f).Type()) //*os.File
}
```
上面的方法Create方法本质就是
```go
func Create(name string) (*File, error) {
	return OpenFile(name, O_RDWR|O_CREATE|O_TRUNC, 0666)
}
```
就是通过OpenFile去创建问题。

### Open
这个方法容易让人入坑的，因为这个方法是以只读打开文件的
```go
import (
    "fmt"
    "os"
)

func main() {
f, _ := os.Open("widuu_2.go")
	defer f.Close()
	fmt.Println(f.Stat())
}
```
深入看一下Open这个方法。
```go
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}
```
它是只读的方式打开的，如果此时你write的方式写是不会报错的，但是不会写入成功的

### Stat
Stat这个方式检查文件的属性
```go
import (
    "fmt"
    "os"
)

func main() {
    f, _ := os.Stat("1.go")
    fmt.Println(f.Size())
}
```
补充一个通过Stat判断文件或者路径是否存在
```go
func PathExist(_path string) bool {
	_, err := os.Stat(_path)
	if err != nil && os.IsNotExist(err) {
		return false
	}
	return true
}
```