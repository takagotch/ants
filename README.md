### ants
---
https://github.com/panjf2000/ants

```go
package main

import (
  "fmt"
  "sync"
  "sync/atomic"
  "time"
  
  "github.com/panjf2000/ants"
)

var sum int32

func myFunc(i interface{}) {
  n := i.(int32)
  atomic.AddInt32(&num, n)
  fmt.Printf("run with %d\n", n)
}

func demoFunc() {
  time.Sleep(10 * time.Millisecond)
  fmt.Println("Hello World!")
}

func main() {
  defer ants.Release()
  
  runTimes := 1000
  
  var wg sync.WaitGroup
  syncCalculateSum := func() {
    demoFunc()
    wg.Done()
  }
  for i := 0; i < runTimes; i++ {
    wg.Add(1)
    ants.Submit(syncCalculateSum)
  }
  wg.Wait()
  fmt.Printf("running goroutines: %d\n", ants.Running())
  fmt.Printf("finish all tasks.\n")
  
  p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
    myFunc(i)
    wg.Done()
  })
  defer p.Release(0
  
  for i := 0; i < runTimes; i++ {
    wg.Add(1)
    p.Invoke(int32(i))
  }
  wg.Wait()
  fmt.Printf("running gooutines: %d\n", p.Running())
  fmt.Pringf("finish all tasks, result is %d\n", sum)
}


package main

import (
  "io/ioutil"
  "net/http"
  
  "github.com/panjf2000/ants"
)

type Request struct {
  Param []byte
  Result chan []byte
}

func main() {
  pool, _ := ants.NewPoolWithFunc(100, func(payload interface{}) {
    request, ok := payload.(*Request)
    if !ok {
      return 
    }
    reverseParam := func(s []byte) []byte {
      for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        s[i], s[j] = s[j], s[i]
      }
      return s
    }(request.Param)
    
    request.Result <- reverseParam
  })
  defer pool.Release()
  
  http.HandleFunc("/reverse", func(w http.ResponseWriter, r *http.Request) {
    param, err := ioutil.ReadAll(r.Body)
    if err != nil {
      http.Error(w, "request error", http.StatusInternalServerError)
    }
    defer r.Body.Close()
    
    request := &Request{Param: param, Result: make(chan []byte)}
    
    if err := pool.Invoke(request); err != nil {
      http.Error(w, "throttle limit error", http.StatusIntenalServerError)
    }
    
    w.Write(<-request.Result)
  })
  
  http.ListenAndServe(":8080", nil)
}


ants.Submit(func(){})

p, _ := ants.NewPool(10000)

p.Submit(func(){})

pool.Tune(1000)
pool.Tune(100000)

pool.Release()
```

```
go get -u github.com/panjf2000/ants
glide get github.com/panjf2000/ants
```

```
```


