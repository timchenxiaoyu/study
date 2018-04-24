## 并发执行md5
```go
package main

import (
	"fmt"
	"path/filepath"
	"crypto/md5"
	"os"
	"sync"
	"io/ioutil"

)

type result struct {
	path string
	sum  [md5.Size]byte
	err  error
}


func main() {
	results := make(chan result)
	var wg sync.WaitGroup

	go func() {
	filepath.Walk("/Users/chenxy/go/src/workspace/test", func(path string,info os.FileInfo,err error) error {
		if err != nil{
			return err
		}

		if info.IsDir(){
			return nil
		}
		fmt.Println(path)
		wg.Add(1)
		go func() {
			defer wg.Done()
			data, err := ioutil.ReadFile(path)
			if err !=nil{
				//return err
			}
			select {
			case results<- result{path:path,
				sum: md5.Sum(data),
			}:
			}

		}()


		return nil
	})
	wg.Wait()
	close(results)
	}()



	for i := range results{
		fmt.Println(i.path,i.sum)
	}

}
```