### Написать генератор случайных чисел

В принципе, легкая задача, на базовые знания по асинхронному взаимодействию в Go. Для решения я бы использовал небуфферезированный канал. Будем асинхронно писать туда случайные числа и закроем его, когда закончим писать.

Плюс ее можно использовать в немного измененном виде в задаче на слияние N каналов

Решение:

```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"time"
)

const (
	seconds = 2
)

func main() {
	randNums := generator(context.Background(), seconds*time.Second)

	for num := range randNums {
		fmt.Println(num)
	}
}

func generator(ctx context.Context, interval time.Duration) <-chan int {
	randomNumbers := make(chan int)

	go func() {
		defer close(randomNumbers)
		after := time.After(interval)

		for {
			select {
			case <-ctx.Done():
				return
			case <-after:
				return
			case randomNumbers <- rand.Int():
			}
		}
	}()

	return randomNumbers
}

```