### Задача

> Реализуйте очередь ограниченного размера.
> 
> Producers добавляют задачи. Consumers забирают задачи.
> 
> Если очередь заполнена, producer должен ждать освобождения места.
> 
> Если очередь пуста, consumer должен ждать появления задачи.

```
type Queue[T any] struct {
	// TODO
}

func NewQueue[T any](capacity int) *Queue[T] {
	// TODO
	return nil
}

func (q *Queue[T]) Push(
	ctx context.Context,
	value T,
) error {
	// TODO
	return nil
}

func (q *Queue[T]) Pop(
	ctx context.Context,
) (T, error) {
	// TODO
	panic("TODO")
}

func (q *Queue[T]) Close() error {
	// TODO
	return nil
}
```

Требования:

- bounded memory;
- cancellation;
- `Close()` должен разбудить ожидающих producers/consumers;
- повторный `Close()` не должен ломать программу.

### Follow-up

> Что происходит с элементами, которые уже находятся в очереди при `Close()`?

**Решение:**

```go
package main

import (
	"context"
	"errors"
	"sync"
)

var (
	ErrClosedQueue     = errors.New("queue already closed")
	ErrInvalidCapacity = errors.New("capacity must be more 0")
)

func main() {}

type Node[T any] struct {
	val  T
	next *Node[T]
}

type Queue[T any] struct {
	head     *Node[T]
	tail     *Node[T]
	length   int
	capacity int
	closed   bool

	mu         sync.Mutex
	isNotFull  chan struct{}
	isNotEmpty chan struct{}
}

func NewQueue[T any](capacity int) (*Queue[T], error) {
	if capacity <= 0 {
		return nil, ErrInvalidCapacity
	}

	return &Queue[T]{
		capacity:   capacity,
		isNotFull:  make(chan struct{}),
		isNotEmpty: make(chan struct{}),
	}, nil
}

func (q *Queue[T]) isNotFullSignal() {
	if q.closed {
		return
	}
	close(q.isNotFull)
	q.isNotFull = make(chan struct{})
}

func (q *Queue[T]) isNotEmptySignal() {
	if q.closed {
		return
	}
	close(q.isNotEmpty)
	q.isNotEmpty = make(chan struct{})
}

func (q *Queue[T]) Push(
	ctx context.Context,
	value T,
) error {
	n := &Node[T]{
		val: value,
	}
	for {
		q.mu.Lock()

		if q.closed {
			q.mu.Unlock()
			return ErrClosedQueue
		}

		if q.length < q.capacity {
			if q.length == 0 {
				q.head = n
				q.tail = n
			} else {
				q.tail.next = n
				q.tail = n
			}
			q.length++

			q.isNotEmptySignal()
			q.mu.Unlock()
			return nil

		}

		isNotFull := q.isNotFull
		q.mu.Unlock()

		select {
		case <-ctx.Done():
			return ctx.Err()
		case <-isNotFull:
		}
	}
}

func (q *Queue[T]) Pop(
	ctx context.Context,
) (val T, err error) {
	for {
		q.mu.Lock()

		if q.length > 0 {
			n := q.head
			val = q.head.val

			q.head = n.next
			q.length--

			if q.length == 0 {
				q.tail = nil
			}

			n.next = nil

			q.isNotFullSignal()
			q.mu.Unlock()
			return val, nil
		}

		if q.closed {
			q.mu.Unlock()
			return val, ErrClosedQueue
		}

		isNotEmpty := q.isNotEmpty
		q.mu.Unlock()

		select {
		case <-ctx.Done():
			return val, ctx.Err()
		case <-isNotEmpty:
		}
	}
}

func (q *Queue[T]) Close() error {
	q.mu.Lock()
	defer q.mu.Unlock()

	if q.closed {
		return nil
	}

	q.closed = true

	close(q.isNotEmpty)
	close(q.isNotFull)

	return nil
}

```