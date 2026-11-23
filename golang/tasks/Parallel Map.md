### Задача

> Есть набор элементов. Каждый элемент нужно обработать независимо.
> 
> Обработка дорогая, поэтому её нужно выполнять параллельно.
> 
> При этом одновременно может выполняться не более `workers` операций.

Дано:

```
type Item struct {
	ID   int
	Name string
}

type Result struct {
	ID     int
	Status bool
}
```

Обработчик:

```
func process(
	ctx context.Context,
	item Item,
) (Result, error) {
	select {
	case <-time.After(
		time.Duration(rand.IntN(500)) * time.Millisecond,
	):
		return Result{
			ID:     item.ID,
			Status: true,
		}, nil

	case <-ctx.Done():
		return Result{}, ctx.Err()
	}
}
```

Точка входа:

```
func ProcessAll(
	ctx context.Context,
	items []Item,
	workers int,
) ([]Result, error) {
	// TODO
	return nil, nil
}
```

Тестовые данные:

```
items := make([]Item, 1000)

for i := range items {
	items[i] = Item{
		ID:   i,
		Name: fmt.Sprintf("item-%d", i),
	}
}

results, err := ProcessAll(
	context.Background(),
	items,
	10,
)
```

Требования:

- максимум 10 обработок одновременно;
- сохранить порядок результатов относительно `items`;
- ошибка одного worker должна остановить остальные;
- cancellation должен корректно завершать обработку;
- не должно быть goroutine leaks.