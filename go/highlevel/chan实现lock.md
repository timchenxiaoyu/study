## 通过chan实现lock锁
```
package main

import (
	"errors"
	"fmt"
	"time"
	"sync"
)

type Lock struct {
	ch chan struct{}
}

func NewLock()Lock{
	return Lock{
		ch : make(chan struct{},1),
	}
}

func (l *Lock)Lock() {
	l.ch <- struct {}{}
}

func (l *Lock)Unlock(){
	<-l.ch
}

func (l *Lock)tryLock(t time.Duration) error{
	select {
	case <-time.After(t):
		return errors.New("timeout")
	case  l.ch<- struct {}{}:
		return nil
		
	}
}
func main() {
	var wg sync.WaitGroup
	wg.Add(20)
	tm := make(map[int]int)
	lock := NewLock()
	for i :=0 ;i<20 ;i++{
		go func(i int) {
			defer wg.Done()
			lock.Lock()
			tm[i]=i
			lock.Unlock()
		}(i)
	}
	wg.Wait()
	fmt.Println(tm)
}
```