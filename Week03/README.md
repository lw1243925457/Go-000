# 第三周 学习笔记
***
## 作业
### 作业要求
&ensp;&ensp;&ensp;&ensp;基于 errgroup 实现一个 http server 的启动和关闭 ，以及 linux signal 信号的注册和处理，要保证能够 一个退出，全部注销退出。

### 作业大致代码如下：

```go
package main

import (
	"context"
	"golang.org/x/sync/errgroup"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
)

func main() {
	eg := errgroup.Group{}
	serverErr := make(chan error, 1)
	sigC := make(chan os.Signal, 1)

	s := http.Server{Addr: ":8088"}

	eg.Go(func() error {
		go func() {
			serverErr <- s.ListenAndServe()
		}()
		select {
		case err := <-serverErr:
			close(sigC)
			close(serverErr)
			return err
		}
	})

	eg.Go(func() error {
		signal.Notify(sigC,
			syscall.SIGINT|syscall.SIGTERM|syscall.SIGKILL)
		<-sigC
		return s.Shutdown(context.TODO())
	})

	log.Println(eg.Wait())
}
```
