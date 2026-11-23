
EasyTopicsCompany TagsHints

Given an integer array `nums`, return `true` if any value appears **more than once** in the array, otherwise return `false`.

**Example 1:**

```java
Input: nums = [1, 2, 3, 3]

Output: true
```

  

**Example 2:**

```java
Input: nums = [1, 2, 3, 4]

Output: false
```

## Hash set
_____

1) Создаём хэш таблицу, размером длины входящего массива
2) В цикле первым шагом проверяем, если текущее число в массиве, если есть возвращаем `true`
3) Если нет, вносим значение в хеш таблицу
4) Если вышли из цикла, дубликатов нет, возвращаем `false`

```go
func hasDuplicate(nums []int) bool {
	uniqueNums := make(map[int]struct{}, len(nums))

	for _, n := range nums {
		if _, ok := uniqueNums[n]; ok {
			return true
		}
		uniqueNums[n] = struct{}{}
	}
	
	return false
}
```

**Сложность алгоритма O(n)**
## Hash Set Length

1) Создаём хеш таблицу размером длины входящего массива
2) Заполняем значениями из входящего массива
3) Сравниваем длину хеш таблицы и входящего массива, хеш таблица меньше, значит дубликаты есть, возвращаем `true`, если нет возвращаем `false`

```go
func hasDuplicate(nums []int) bool {
    uniqueNums := make(map[int]struct{}, len(nums))
    
    for _, num := range nums {
        seen[num] = struct{}{}
    }
    
    return len(seen) < len(nums)
}
```

**Сложность алгоритма O(n)**

#Arrays #Easy