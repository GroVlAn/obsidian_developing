### Задача

> У нас есть сервис пользователей. Запрос пользователя может одновременно прийти в 100 HTTP handlers.
> 
> Если пользователь ещё не находится в кэше, каждый handler идёт в БД.
> 
> Нужно сделать так, чтобы для одного `userID` одновременно выполнялся только один запрос в БД.

Дано:

```
type User struct {
	ID   string
	Name string
}

func loadUserFromDB(
	ctx context.Context,
	id string,
) (User, error) {

	fmt.Println("DB REQUEST:", id)

	select {
	case <-time.After(time.Second):
		return User{
			ID:   id,
			Name: "Vlad",
		}, nil

	case <-ctx.Done():
		return User{}, ctx.Err()
	}
}
```

Нужно реализовать:

```
type UserLoader struct {
	// TODO
}

func NewUserLoader() *UserLoader {
	// TODO
	return nil
}

func (l *UserLoader) Get(
	ctx context.Context,
	id string,
) (User, error) {
	// TODO
	panic("TODO")
}
```

Тест:

```
loader := NewUserLoader()

var wg sync.WaitGroup

for i := 0; i < 100; i++ {
	wg.Add(1)

	go func() {
		defer wg.Done()

		_, err := loader.Get(
			context.Background(),
			"user-123",
		)

		if err != nil {
			fmt.Println(err)
		}
	}()
}

wg.Wait()
```

В идеальном случае:

```
DB REQUEST: user-123
```

появляется один раз.