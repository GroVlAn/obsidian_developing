Даны два канала. В первый пишутся числа. Нужно, чтобы числа читались из первого по мере поступления, что-то с ними происходило (допустим, возводились в квадрат) и результат записывался во второй канал.

Решение:

```go
package main

import (
	"context"
	"fmt"
	"math/rand/v2"
	"os/signal"
	"syscall"
)

const (
	maxNumbers = 1_000_000
	maxRandoms = 100
)

func main() {
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer cancel()

	rands := generator(ctx, maxNumbers, maxRandoms)

	squares := square(ctx, rands)

	for {
		select {
		case <-ctx.Done():
			return
		case v, ok := <-squares:
			if !ok {
				return
			}

			fmt.Println(v)
		}
	}
}

func generator(ctx context.Context, maxNums int, maxRandom int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)
		for range maxNums {
			select {
			case <-ctx.Done():
				return
			case out <- rand.IntN(maxRandom):
			}
		}
	}()

	return out
}

func square(ctx context.Context, nums <-chan int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)
		for {
			select {
			case <-ctx.Done():
				return
			case v, ok := <-nums:
				if !ok {
					return
				}

				select {
				case <-ctx.Done():
					return
				case out <- v * v:
				}
			}
		}
	}()

	return out
}

```