Даны n каналов типа chan int. Надо написать функцию, которая смерджит все данные из этих каналов в один и вернет его.

Мы хотим, чтобы результат работы функции выглядел примерно так:

```go
for num := range joinChannels(a, b, c) {

       fmt.Println(num)

}
```

Решение:

```go
package main

import (
	"context"
	"fmt"
	"os/signal"
	"sync"
	"syscall"
)

const (
	secondsInterval = 1
)

func main() {
	var (
		a chan int = make(chan int)
		b chan int = make(chan int)
		c chan int = make(chan int)
	)

	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer cancel()

	producer := func(out chan<- int, start int) {
		defer close(out)
		for i := start; i < start+1_000_000; i++ {
			select {
			case <-ctx.Done():
				return
			case out <- i:
			}
		}
	}

	go producer(a, 0)
	go producer(b, 1_000_000)
	go producer(c, 2_000_000)

	for n := range joinChannels(ctx, a, b, c) {
		fmt.Println(n)
	}
}

func joinChannels(ctx context.Context, channels ...<-chan int) <-chan int {
	res := make(chan int)
	var wg sync.WaitGroup

	forward := func(ch <-chan int) func() {
		return func() {
			for {
				select {
				case <-ctx.Done():
					return
				case v, ok := <-ch:
					if !ok {
						return
					}

					select {
					case <-ctx.Done():
						return
					case res <- v:
					}
				}
			}
		}
	}

	for _, ch := range channels {
		wg.Go(forward(ch))
	}

	go func() {
		wg.Wait()
		close(res)
	}()

	return res
}

```