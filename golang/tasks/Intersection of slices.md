На вход подаются два неупорядоченных слайса любой длины. Надо написать функцию, которая возвращает их пересечение

Надо посчитать количество появлений элементов первого массива (лучше брать тот, что покороче) — используем для этого словарь. Потом пройтись по второму массиву и вычитать из словаря те элементы, которые есть в нем. По ходу добавляем в результат те элементы, у которых частота появлений больше нуля.

Решение:

```go
package main

import "fmt"

func main() {
	slice1 := []int{223, 32, 85, 2, 1943, 33, 4515, 5123, 5135, 33, 95, 0, 33, 455}
	slice2 := []int{23, 52, 0, 33, 22, 59, 2233, 95, 85, 3, 22, 2, 90, 33, 59, 33}

	intersect := intersection(slice1, slice2)

	fmt.Println(intersect)
}

func intersection(slice1, slice2 []int) []int {
	uniq := make(map[int]struct{})

	result := make([]int, 0)
	if len(slice1) > len(slice2) {
		slice2, slice1 = slice1, slice2
	}

	for _, num := range slice1 {
		uniq[num] = struct{}{}
	}

	for _, num := range slice2 {
		if _, ok := uniq[num]; ok {
			result = append(result, num)
			delete(uniq, num)
		}
	}

	return result
}

```