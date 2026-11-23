### Задача

> Из внешнего источника приходят миллионы объектов.
> 
> В БД мы можем записывать их только батчами по 1000 элементов.

Источник:

```
type Item struct {
	ID int
}

func ReadItems(ctx context.Context) <-chan Item {
	out := make(chan Item)

	go func() {
		defer close(out)

		for i := 0; i < 10_000_000; i++ {
			select {
			case out <- Item{ID: i}:
			case <-ctx.Done():
				return
			}
		}
	}()

	return out
}
```

Функция сохранения:

```
func SaveBatch(
	ctx context.Context,
	items []Item,
) error {
	// эмуляция БД
	time.Sleep(100 * time.Millisecond)

	return nil
}
```

Точка входа:

```
func Process(
	ctx context.Context,
	items <-chan Item,
	workers int,
	batchSize int,
) error {
	// TODO
	return nil
}
```

Требования:

- batch size = 1000;
- максимум 8 batch'ей одновременно;
- нельзя загрузить все 10 млн объектов в память;
- ошибка одного batch должна остановить обработку;
- context cancellation должен работать.
  
Решение:

```go
package main

import (
	"context"
	"fmt"
	"os/signal"
	"syscall"
	"time"

	"golang.org/x/sync/errgroup"
)

func main() {
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer cancel()

	items := ReadItems(ctx)

	if err := Process(ctx, items, 8, 1000); err != nil {
		fmt.Println("Error: ", err)
	}
}

func Process(
	ctx context.Context,
	items <-chan Item,
	workers int,
	batchSize int,
) error {
	g, ctx := errgroup.WithContext(ctx)

	dataCh := make(chan Item, batchSize)

	producer := func() error {
		defer close(dataCh)

		for {
			select {
			case <-ctx.Done():
				return ctx.Err()
			case item, ok := <-items:
				if !ok {
					return nil
				}

				select {
				case <-ctx.Done():
					return ctx.Err()
				case dataCh <- item:
				}
			}
		}
	}

	g.Go(producer)

	consumer := func() error {
		data := make([]Item, 0, batchSize)

		flush := func() error {
			if len(data) == 0 {
				return nil
			}

			if err := SaveBatch(ctx, data); err != nil {
				return err
			}

			data = make([]Item, 0, batchSize)

			return nil
		}

		for {
			select {
			case <-ctx.Done():
				return ctx.Err()
			case item, ok := <-dataCh:
				if !ok {
					return flush()
				}

				data = append(data, item)
				if len(data) == batchSize {
					if err := flush(); err != nil {
						return err
					}
				}
			}
		}
	}

	for range workers {
		g.Go(consumer)
	}

	return g.Wait()
}

type Item struct {
	ID int
}

func ReadItems(ctx context.Context) <-chan Item {
	out := make(chan Item)

	go func() {
		defer close(out)

		for i := 0; i < 10_000_000; i++ {
			select {
			case out <- Item{ID: i}:
			case <-ctx.Done():
				return
			}
		}
	}()

	return out
}

func SaveBatch(
	ctx context.Context,
	items []Item,
) error {
	ticker := time.NewTicker(100 * time.Millisecond)
	defer ticker.Stop()

	for {
		select {
		case <-ctx.Done():
			return ctx.Err()
		case <-ticker.C:
			fmt.Println(len(items))

			return nil
		}
	}
}

```