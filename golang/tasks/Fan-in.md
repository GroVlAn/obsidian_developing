 ### Задача

> Есть несколько независимых worker'ов. Каждый worker возвращает свой channel результатов.
> 
> Нужно объединить результаты в один channel.

Начальный worker:

```
type Result struct {
	TaskID int
	Value  int
}

func worker(
	ctx context.Context,
	id int,
) <-chan Result {
	out := make(chan Result)

	go func() {
		defer close(out)

		for i := 0; i < 10; i++ {
			select {
			case out <- Result{
				TaskID: id,
				Value:  i,
			}:
			case <-ctx.Done():
				return
			}
		}
	}()

	return out
}
```

Точка входа:

```
func FanIn(
	ctx context.Context,
	inputs ...<-chan Result,
) <-chan Result {
	// TODO
	return nil
}
```

Использование:

```
ctx := context.Background()

result := FanIn(
	ctx,
	worker(ctx, 1),
	worker(ctx, 2),
	worker(ctx, 3),
)

for r := range result {
	fmt.Println(r)
}
```

Требования:

- объединить все каналы;
- закрыть результирующий channel после завершения всех input channels;
- поддержать cancellation;
- не оставить goroutine после выхода consumer.

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

func main() {
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer cancel()

	result := FanIn(
		ctx,
		worker(ctx, 1),
		worker(ctx, 2),
		worker(ctx, 3),
	)

	for r := range result {
		fmt.Println(r)
	}
}

func FanIn(
	ctx context.Context,
	inputs ...<-chan Result,
) <-chan Result {
	res := make(chan Result)

	var wg sync.WaitGroup

	for _, ch := range inputs {
		wg.Go(func() {
			for {
				select {
				case <-ctx.Done():
					return
				case val, ok := <-ch:
					if !ok {
						return
					}

					select {
					case <-ctx.Done():
						return
					case res <- val:
					}
				}
			}
		})
	}

	go func() {
		wg.Wait()
		close(res)
	}()

	return res
}

type Result struct {
	TaskID int
	Value  int
}

func worker(
	ctx context.Context,
	id int,
) <-chan Result {
	out := make(chan Result)

	go func() {
		defer close(out)

		for i := 0; i < 10; i++ {
			select {
			case out <- Result{
				TaskID: id,
				Value:  i,
			}:
			case <-ctx.Done():
				return
			}
		}
	}()

	return out
}

```