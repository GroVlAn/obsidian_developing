### Задача

> Реализуйте потокобезопасную map для хранения пользовательских данных.

```
type ConcurrentMap[K comparable, V any] struct {
	// TODO
}

func NewConcurrentMap[K comparable, V any]() *ConcurrentMap[K, V] {
	// TODO
	return nil
}

func (m *ConcurrentMap[K, V]) Get(
	key K,
) (V, bool) {
	// TODO
	panic("TODO")
}

func (m *ConcurrentMap[K, V]) Set(
	key K,
	value V,
) {
	// TODO
}

func (m *ConcurrentMap[K, V]) Delete(
	key K,
) {
	// TODO
}
```

Тест:

```
m := NewConcurrentMap[string, int]()

var wg sync.WaitGroup

for i := 0; i < 1000; i++ {
	wg.Add(1)

	go func(i int) {
		defer wg.Done()

		m.Set("counter", i)
	}(i)
}

wg.Wait()
```

### Follow-up

> Чтений в 100 раз больше записей. Что можно улучшить?

> Теперь 100 000 операций в секунду. Все операции идут по одному ключу. Что делать?


**Решение**

```go

var (
	ErrInvalidCapacity = errors.New("capacity must be more 0")
)

type ConcurrentMap[K comparable, V any] struct {
	m    map[K]V
	rwMu sync.RWMutex
}

func NewConcurrentMap[K comparable, V any]() *ConcurrentMap[K, V] {
	return &ConcurrentMap[K, V]{
		m: make(map[K]V),
	}
}

func NewConcurrentMapWithCapacity[K comparable, V any](capacity int) (*ConcurrentMap[K, V], error) {
	if capacity <= 0 {
		return nil, ErrInvalidCapacity
	}

	return &ConcurrentMap[K, V]{
		m: make(map[K]V, capacity),
	}, nil
}

func (m *ConcurrentMap[K, V]) Get(
	key K,
) (V, bool) {
	m.rwMu.RLock()
	defer m.rwMu.RUnlock()
	v, ok := m.m[key]

	return v, ok
}

func (m *ConcurrentMap[K, V]) Set(
	key K,
	value V,
) {
	m.rwMu.Lock()
	defer m.rwMu.Unlock()
	m.m[key] = value
}

func (m *ConcurrentMap[K, V]) Delete(
	key K,
) {
	m.rwMu.Lock()
	defer m.rwMu.Unlock()
	delete(m.m, key)
}
```