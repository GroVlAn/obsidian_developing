### Задача

> Нужно обработать список пользователей через внешний API.
> 
> API разрешает максимум 20 одновременных запросов.
> 
> При этом использовать worker pool нельзя.

Дано:

```
type User struct {
	ID int
}

func CallExternalAPI(
	ctx context.Context,
	user User,
) error {
	select {
	case <-time.After(200 * time.Millisecond):
		return nil

	case <-ctx.Done():
		return ctx.Err()
	}
}
```

Точка входа:

```
func ProcessUsers(
	ctx context.Context,
	users []User,
) error {
	// TODO
	return nil
}
```

Требование:

```
At most 20 calls to CallExternalAPI may execute simultaneously.
```

### Follow-up

> Какое средство синхронизации Go вы бы использовали?


**Решение:**

```go

type User struct {
	ID int
}

func ProcessUsers(
	ctx context.Context,
	users []User,
) error {
	semaphore := make(chan struct{}, maxProcesses)

	g, ctx := errgroup.WithContext(ctx)

	acquire := func(ctx context.Context) error {
		select {
		case <-ctx.Done():
			return ctx.Err()
		case semaphore <- struct{}{}:
		}

		return nil
	}

	release := func() {
		<-semaphore
	}

	for _, user := range users {
		g.Go(func() error {
			if err := acquire(ctx); err != nil {
				return err
			}
			defer release()

			return CallExternalAPI(ctx, user)
		})
	}

	return g.Wait()
}

func CallExternalAPI(
	ctx context.Context,
	user User,
) error {
	select {
	case <-time.After(200 * time.Millisecond):
		return nil

	case <-ctx.Done():
		return ctx.Err()
	}
}

```