### Задача

> Нужно реализовать in-memory cache для сервиса. Кэш хранит результаты дорогих операций.
> 
> У кэша есть ограничение по количеству элементов. Когда capacity достигнута, необходимо удалить элемент, который использовался дольше всего назад.
> 
> Кроме того, каждая запись имеет TTL.

Начальный код:

```
package cache

import (
	"time"
)

type Cache[K comparable, V any] struct {
	// TODO
}

func New[K comparable, V any](
	capacity int,
	ttl time.Duration,
) *Cache[K, V] {
	// TODO
	return nil
}

func (c *Cache[K, V]) Get(key K) (V, bool) {
	// TODO
	panic("TODO")
}

func (c *Cache[K, V]) Set(key K, value V) {
	// TODO
}

func (c *Cache[K, V]) Delete(key K) {
	// TODO
}
```

Пример использования:

```
func main() {
	cache := New[string, string](2, time.Second)

	cache.Set("user:1", "Vlad")
	cache.Set("user:2", "Alex")

	value, ok := cache.Get("user:1")

	fmt.Println(value, ok)

	cache.Set("user:3", "Bob")

	_, ok = cache.Get("user:2")

	fmt.Println(ok)
}
```

Требования:

- `Get` — `O(1)` в среднем;
- `Set` — `O(1)` в среднем;
- capacity;
- TTL;
- expired value нельзя вернуть;
- cache должен быть безопасен при concurrent access.

### Follow-up

> Сейчас `Get` и `Set` вызываются из сотен goroutine. Что нужно изменить?

> Как удалять expired entries, если кэш никто не читает?

> Что произойдёт, если capacity = 1?

Решение:

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"log"
	"sync"
	"time"
)

var (
	ErrInvalidCapacity = errors.New("capacity must be greater than 0")
	ErrInvalidTTL      = errors.New("TTL must be greater than 0")
)

func main() {

	cache, err := New[string, string](context.Background(), 2, 10*time.Second)
	if err != nil {
		log.Fatalf("fatal initializing cache: %s", err.Error())
	}

	cache.Set("user:1", "Vlad")
	cache.Set("user:2", "Alex")

	value, ok := cache.Get("user:1")

	fmt.Println(value, ok)

	cache.Set("user:3", "Bob")

	_, ok = cache.Get("user:2")

	fmt.Println(ok)
}

type Cache[K comparable, V any] struct {
	data     map[K]*Node[K, V]
	dataList *DataList[K, V]
	capacity int
	ttl      time.Duration

	mu sync.Mutex
}

func New[K comparable, V any](
	ctx context.Context,
	capacity int,
	ttl time.Duration,
) (*Cache[K, V], error) {
	if capacity <= 0 {
		return nil, ErrInvalidCapacity
	}

	if ttl <= 0 {
		return nil, ErrInvalidTTL
	}

	c := &Cache[K, V]{
		data:     make(map[K]*Node[K, V]),
		dataList: NewDataList[K, V](),
		capacity: capacity,
		ttl:      ttl,
	}

	go c.cleanup(ctx)

	return c, nil
}

func (c *Cache[K, V]) Get(key K) (val V, ok bool) {
	c.mu.Lock()
	defer c.mu.Unlock()

	node, ok := c.data[key]
	if !ok {
		return
	}

	if !node.ttl.After(time.Now()) {
		c.dataList.RemoveNode(node)
		delete(c.data, key)

		return val, false
	}

	c.dataList.MoveToEnd(node)

	return node.v, ok
}

func (c *Cache[K, V]) Set(key K, value V) {
	c.mu.Lock()
	defer c.mu.Unlock()

	node, ok := c.data[key]
	if ok {
		node.v = value
		node.ttl = time.Now().Add(c.ttl)
		c.dataList.MoveToEnd(node)

		return
	}

	if c.dataList.Len() == c.capacity {
		k := c.dataList.RemoveFirst()
		delete(c.data, k)
	}

	c.data[key] = c.dataList.SetToEnd(key, value, time.Now().Add(c.ttl))
}

func (c *Cache[K, V]) Delete(key K) {
	c.mu.Lock()
	defer c.mu.Unlock()

	node, ok := c.data[key]
	if !ok {
		return
	}

	c.dataList.RemoveNode(node)
	delete(c.data, key)
}

func (c *Cache[K, V]) cleanup(ctx context.Context) {
	ticker := time.NewTicker(c.ttl)
	defer ticker.Stop()

	for {
		select {
		case <-ctx.Done():
			return
		case <-ticker.C:
			c.mu.Lock()
			now := time.Now()
			c.dataList.Walk(func(node *Node[K, V]) {
				if !node.ttl.After(now) {
					c.dataList.RemoveNode(node)
					delete(c.data, node.k)
				}
			})
			c.mu.Unlock()
		}
	}
}

type Node[K comparable, V any] struct {
	next *Node[K, V]
	prev *Node[K, V]
	ttl  time.Time
	k    K
	v    V
}

type DataList[K comparable, V any] struct {
	head   *Node[K, V]
	tail   *Node[K, V]
	length int
}

func NewDataList[K comparable, V any]() *DataList[K, V] {
	return &DataList[K, V]{}
}

func (dl *DataList[K, V]) IsEmpty() bool {
	return dl.length == 0
}

func (dl *DataList[K, V]) Len() int {
	return dl.length
}

func (dl *DataList[K, V]) SetToEnd(k K, v V, ttl time.Time) *Node[K, V] {
	n := &Node[K, V]{
		k:   k,
		v:   v,
		ttl: ttl,
	}

	if dl.IsEmpty() {
		dl.head = n
		dl.tail = n
		dl.length++

		return n
	}

	dl.tail.next = n
	n.prev = dl.tail
	dl.tail = n
	dl.length++

	return n
}

func (dl *DataList[K, V]) RemoveFirst() (key K) {
	if dl.IsEmpty() {
		return
	}
	key = dl.head.k

	if dl.length == 1 {
		dl.head = nil
		dl.tail = nil
		dl.length--
		return
	}

	dl.head = dl.head.next
	dl.head.prev = nil
	dl.length--

	return key
}

func (dl *DataList[K, V]) RemoveNode(node *Node[K, V]) {
	if dl.IsEmpty() {
		return
	}

	switch {
	case dl.length == 1 || node.prev == nil:
		dl.RemoveFirst()
		node.prev = nil
		node.next = nil
		return
	case node == dl.tail:
		node.prev.next = nil
		dl.tail = dl.tail.prev
	default:
		node.prev.next = node.next
		node.next.prev = node.prev
	}

	dl.length--
	node.prev = nil
	node.next = nil
}

func (dl *DataList[K, V]) MoveToEnd(node *Node[K, V]) V {
	v := node.v

	if node == dl.tail {
		return v
	}

	dl.RemoveNode(node)
	node.prev = dl.tail
	dl.tail.next = node
	dl.tail = node

	return v
}

func (dl *DataList[K, V]) Walk(fn func(node *Node[K, V])) {
	if dl.IsEmpty() {
		return
	}

	for current := dl.head; current != nil; {
		next := current.next
		fn(current)
		current = next
	}
}

```