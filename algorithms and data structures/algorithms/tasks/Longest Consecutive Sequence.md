Given an array of integers `nums`, return _the length_ of the longest consecutive sequence of elements that can be formed.

A _consecutive sequence_ is a sequence of elements in which each element is exactly `1` greater than the previous element. The elements do _not_ have to be consecutive in the original array.

You must write an algorithm that runs in `O(n)` time.

**Example 1:**

```java
Input: nums = [2,20,4,10,3,4,5]

Output: 4
```

Explanation: The longest consecutive sequence is `[2, 3, 4, 5]`.

**Example 2:**

```java
Input: nums = [0,3,2,5,4,6,1,1]

Output: 7
```

**Constraints:**

- `0 <= nums.length <= 100,000`
- `-10^9 <= nums[i] <= 10^9`

## Has Set

1) Создаём хеш сет и заполняем значениями из массива
2) Создаём переменную для результата равную 0
3) Идём циклом по хеш сету
4) Проверяем есть ли значение меньше на 1 чем текущее, если да, то это первый элемент
5) Первый элемент сейчас length
6) Проверяем есть ли в хеш сет элемент, равный сумме текущего и length и если есть, увеличиваем length на 1, если нет выходим из внутреннего цикла
7) Если length больше чем текущий сохранённый результат, записываем length в результат

```go
func longestConsecutive(nums []int) int {
	numSet := make(map[int]struct{})

	for _, num := range nums {
		numSet[num] = struct{}{}
	} 

	longest := 0
	for num := range numSet {
		if _, ok := numSet[num-1]; !ok {
			length := 1
			for {
				if _, exists := numSet[num+length]; exists {
					length++
				} else {
					break
				}
			}
			if longest < length {
				longest = length
			}
		}
	}

	return longest
}
```
**Сложность алгоритма: O(n)**

#Arrays #Medium