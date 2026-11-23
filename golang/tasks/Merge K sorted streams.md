### Задача

> У нас есть несколько источников данных. Каждый источник отдаёт числа в отсортированном порядке.
> 
> Нужно объединить их в один отсортированный поток.

Источники:

```
func source(values []int) <-chan int {
	out := make(chan int)

	go func() {
		defer close(out)

		for _, value := range values {
			out <- value
		}
	}()

	return out
}
```

Тест:

```
streams := []<-chan int{
	source([]int{1, 4, 7, 10}),
	source([]int{2, 5, 8, 11}),
	source([]int{3, 6, 9, 12}),
}

result := MergeStreams(streams)

for value := range result {
	fmt.Println(value)
}
```

Ожидаемый порядок:

```
1
2
3
4
5
6
7
8
9
10
11
12
```

Точка входа:

```
func MergeStreams(
	streams []<-chan int,
) <-chan int {
	// TODO
	return nil
}
```

### Follow-up

> Теперь streams может быть 10 000. Что будет с вашим решением?

> Что произойдёт, если один из source перестал отправлять данные и никогда не закрывает channel?