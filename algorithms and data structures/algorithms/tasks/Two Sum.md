Given an array of integers `nums` and an integer `target`, return the indices `i` and `j` such that `nums[i] + nums[j] == target` and `i != j`.

You may assume that _every_ input has exactly one pair of indices `i` and `j` that satisfy the condition.

Return the answer with the smaller index first.

**Example 1:**

```java
Input: 
nums = [3,4,5,6], target = 7

Output: [0,1]
```

Explanation: `nums[0] + nums[1] == 7`, so we return `[0, 1]`.

**Example 2:**

```java
Input: nums = [4,5,6], target = 10

Output: [0,2]
```

**Example 3:**

```java
Input: nums = [5,5], target = 10

Output: [0,1]
```

## Hash Map (Two Pass)

1) Создаём хеш таблицу и заполняем её, используя входящий массив, где ключ будет значение массива, а значение индекс числа из массива
2) Идём циклом по входящему массиву, достаём значение из хеш таблицы вычитанием из искомого числа, текущего числа из массива
3) Если в хеш таблице есть значение под таким числом и индекс из хеш таблицы не совпадает с текущим индексом массива, значит это и есть индексы чья сумма равняется таргету
4) Иначе таких индексов нет

```go
func twoSum(nums []int, target int) []int {
    indices := make(map[int]int)

    for i, n := range nums {
        indices[n] = i
    }

    for i, n := range nums {
        diff := target - n
        if j, found := indices[diff]; found && j != i {
            return []int{i, j}
        }
    }
    return []int{}
}
```

**Сложность алгоритма: O(n)**

### Через один цикл

```go
func twoSum(nums []int, target int) []int {
	indices := make(map[int]int, len(nums))

	for i, n := range nums {
		if j, ok := indices[target-n]; ok {
			return []int{j, i}
		}

		indices[n] = i
	} 

	return nil
}
```

**Сложность алгоритма: O(n)**

#Arrays #Easy