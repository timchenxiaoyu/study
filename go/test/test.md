## test
测试分为功能测试，性能测试和并发测试
功能测试就是测试功能是否好使，性能测试则是指性能如何，跑的快不快
而并发测试则是针对多核的测试，检测是否能能并行执行
下面用一个例子来看

```
package bench

import "sync"

func Add(x int, y int) (z int) {
	z = x + y
	return
}


type ForTest struct {
	num int ;
	sync.Mutex
}

func (this * ForTest) Loops() {
	for i:=0 ; i  < 10000 ; i++ {
		//this.Lock()
		this.num++
		//this.Unlock()
	}
}

```

上面是一个测试的例子

## 功能测试
首先是功能测试

```go
package bench

import (
	"testing"
)

type AddArray struct {
	result  int;
	add_1   int;
	add_2   int;
}

func TestAdd(t * testing.T) {
	var test_data = [3] AddArray {
		{ 2, 1, 1},
		{ 5, 2, 3},
		{ 4, 2, 2},
	}
	for _ , v := range test_data {
		if v.result != Add(v.add_1, v.add_2) {
			t.Errorf("Add( %d , %d ) != %d \n", v.add_1 , v.add_2 , v.result);
		}
	}
}
```

这个很容易理解，就是查看结果是否是我所期望的

## 性能测试

```go
func BenchmarkLoops(b *testing.B) {
	var test ForTest
	ptr := &test
	// 必须循环 b.N 次 。 这个数字 b.N 会在运行中调整，以便最终达到合适的时间消耗。方便计算出合理的数据。 （ 免得数据全部是 0 ）
	b.Log(b.N)
	for i:=0 ; i<b.N ; i++ {
		//if 10000 != ptr.Loops(){
		//	b.Errorf("result is not true")
		//}
		ptr.Loops()
	}
	b.Log("----------",ptr.num)

}
```

通过循去测试性能如何，主要看op/s,就是每秒的操作数。

## 并发测试

上面的性能测试，只能测试在单个协程下执行的效率
如果是想在多核下测试，则需要通过下面的demo
```go
// 测试并发效率
func BenchmarkLoopsParallel(b *testing.B) {
	var test ForTest
	ptr := &test
	b.Log(b.N)
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			ptr.Loops()
		}

	})
	b.Log(ptr.num)
}
```
查看多核下运行的效果，如果上面的直接执行会发现输出num结果出现问题，就是在多核执行的情况下，会出现
临界区访问冲突的问题，可以通过上面加锁的方式去避免这种问题的产生。

还有就是直接在多核运行下直接报错譬如下面
```go
var m = make(map[int]int,0)
func Maptest(){
	m[1] = 1
}


func BenchmarkMaptestParallel(b *testing.B) {

	b.Log(b.N)
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Maptest()
		}

	})
}
```
而Maptest在性能测试时不会有问题的。