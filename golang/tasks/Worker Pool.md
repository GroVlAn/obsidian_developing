### Задача

> Нужно реализовать worker pool для обработки фоновых задач.
> 
> В системе может одновременно выполняться только `N` задач.

Начальные структуры:

```
type Task struct {
	ID int
}

type WorkerPool struct {
	// TODO
}
```

Обработчик:

```
func processTask(ctx context.Context, task Task) error {
	fmt.Println("processing", task.ID)

	select {
	case <-time.After(100 * time.Millisecond):
		return nil

	case <-ctx.Done():
		return ctx.Err()
	}
}
```

Интерфейс:

```
func NewWorkerPool(workers int) *WorkerPool {
	// TODO
	return nil
}

func (p *WorkerPool) Submit(
	ctx context.Context,
	task Task,
) error {
	// TODO
	return nil
}

func (p *WorkerPool) Shutdown(
	ctx context.Context,
) error {
	// TODO
	return nil
}
```

Использование:

```
pool := NewWorkerPool(5)

for i := 0; i < 100; i++ {
	err := pool.Submit(
		context.Background(),
		Task{ID: i},
	)

	if err != nil {
		fmt.Println(err)
	}
}

ctx, cancel := context.WithTimeout(
	context.Background(),
	5*time.Second,
)
defer cancel()

err := pool.Shutdown(ctx)
if err != nil {
	fmt.Println(err)
}
```

Требования:

- одновременно работает максимум `workers`;
- `Submit` после shutdown должен возвращать ошибку;
- уже принятые задачи должны быть обработаны;
- `Shutdown` должен дождаться завершения;
- если context shutdown закончился — вернуть ошибку;
- не должно быть panic из-за отправки в закрытый channel;
- не должно быть goroutine leak.

### Follow-up

> Что происходит, если `Submit` и `Shutdown` вызываются одновременно?