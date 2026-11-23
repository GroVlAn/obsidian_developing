### Задача

> У нас есть сервис аналитики. Он получает события пользователей из message broker.
> 
> Нужно определить `K` наиболее популярных типов событий.
> 
> Количество событий заранее неизвестно и может быть очень большим.

Для задачи считаем, что broker уже преобразован в Go channel.

```
type Event struct {
	ID        string
	UserID    string
	EventType string
}
```

Источник:

```
func consumeEvents() <-chan Event {
	out := make(chan Event)

	go func() {
		defer close(out)

		for i := 0; i < 10_000_000; i++ {
			out <- Event{
				ID:        strconv.Itoa(i),
				UserID:    fmt.Sprintf("user-%d", i%100_000),
				EventType: []string{
					"login",
					"logout",
					"purchase",
					"search",
					"view",
				}[i%5],
			}
		}
	}()

	return out
}
```

Точка входа:

```
func TopKEvents(
	events <-chan Event,
	k int,
) []string {
	// TODO
	return nil
}
```

Использование:

```
events := consumeEvents()

result := TopKEvents(events, 3)

fmt.Println(result)
```

Требования:

- нельзя преобразовывать весь channel в `[]Event`;
- поток потенциально может быть бесконечным;
- дополнительная память должна быть ограничена.

### Follow-up

> Что изменится, если нужно получать текущий Top K каждые 10 секунд, не останавливая обработку событий?