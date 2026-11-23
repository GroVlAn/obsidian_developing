
Given an integer array `nums` and an integer `k`, return the `k` most frequent elements within the array.

The test cases are generated such that the answer is always **unique**.

You may return the output in **any order**.

**Example 1:**

```java
Input: nums = [1,2,2,3,3,3], k = 2

Output: [2,3]
```

**Example 2:**

```java
Input: nums = [7,7], k = 1

Output: [7]
```

**Constraints:**

- `1 <= nums.length <= 10^4`.
- `-1000 <= nums[i] <= 1000`
- `1 <= k <= number of distinct elements in nums`.

## Bucket Sort

1) Создаём хеш таблицу для хранения чисел и их количества в иходном массиве
2) Создаём массив массивов чисел размером с исходный массив + 1, здесь будут храниться массивы чисел, где индекс массива, это количество встречаемости в исходном массиве, а в массивах соответственно числа, которые встречаются `i` раз
3) Заполняем хеш таблицу, увеличивая на единицу значение, по ключу от числа из массива
4) Заполняем массив массивов проходясь циклом по хеш таблицу, обращаясь к индексу по ключу хеш таблицы и добавляем значение из хеш таблицы в массив по данном индексу
5) Идём циклом с конца массива массивов чисел, внутри идём циклом по массиву по текущему индексу и добавляем значение в результат, пока размер слайса результата не будет равен размеру требуемого количества чисел

```go
func topKFrequent(nums []int, k int) []int {
	count := make(map[int]int)
	freq := make([][]int, len(nums)+1)

	for _, num := range nums {
		count[num]++
	}
	for k, v := range count {
		freq[v] = append(freq[v], k)
	}

	result := make([]int, 0)

	for i := len(freq) - 1; i > 0; i-- {
		for _, num := range freq[i] {
			result = append(result, num)
			if len(result) == k {
				return result
			}
		}
	}
	
	return result
}
```

**Сложность алгоритма: O(n)**

#Arrays #Medium