Given an integer array `nums`, return an array `output` where `output[i]` is the product of all the elements of `nums` except `nums[i]`.

Each product is **guaranteed** to fit in a **32-bit** integer.

Follow-up: Could you solve it in O(n)O(n) time without using the division operation?

**Example 1:**

```java
Input: nums = [1,2,4,6]

Output: [48,24,12,8]
```

**Example 2:**

```java
Input: nums = [-1,0,1,2,3]

Output: [0,-6,0,0,0]
```

**Constraints:**

- `2 <= nums.length <= 100,000`
- `-30 <= nums[i] <= 30`
- The product of any prefix or suffix of `nums` is **guaranteed** to fit in a **32-bit** integer.

## Prefix & Postfix

1) Создаём префикс и постфикс равные 1
2) Создаём слайс из нулевых элементов размером с исходный слайс
3) Идём слева на права, присвоив заранее первому элементу значение префикса
4) В цикле на каждом шаге умножаем префикс на `i` элемент минус один и присваиваем `i-ому` элементу результирующего массива
5) Во втором цикле идём с права налево, с предпоследнего элемента и в цикле постфикс умножаем на `i-ый` элемент исходного массива плюс один и умножаем текущий элемент результирующего массива на постфикс

```go
func productExceptSelf(nums []int) []int {
	prefix, postfix := 1, 1
	res := make([]int, len(nums))

	res[0] = prefix
	for i := 1; i < len(nums); i++ {
		prefix *= nums[i-1]
		res[i] = prefix
	}
	
	for i := len(nums) - 2; i >= 0; i-- {
		postfix *= nums[i+1]
		res[i] *= postfix
	}

	return res
}
```

**Сложность алгоритма:  O(n)**

#Arrays #Medium