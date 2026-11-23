
Given an array of strings `strs`, group all _anagrams_ together into sublists. You may return the output in **any order**.

An **anagram** is a string that contains the exact same characters as another string, but the order of the characters can be different.

**Example 1:**

```java
Input: strs = ["act","pots","tops","cat","stop","hat"]

Output: [["hat"],["act", "cat"],["stop", "pots", "tops"]]
```

**Example 2:**

```java
Input: strs = ["x"]

Output: [["x"]]
```

**Example 3:**

```java
Input: strs = [""]

Output: [[""]]
```

## Hash Table

1. Создаём хеш таблицу, где ключ это массив из 26 числе, где каждое это индекс буквы, а значение массив строк
2. Заполняем хеш таблицу
	2.1  Вынесем в отдельную функцию, заполнения массива значением символ мину 'a' `code[str-'a']++`
	2.2 Используем полученный массив как ключ для хеш таблицы, по которому добавляем к массиву текущую строчку
3. Во втором цикле полученные массивы из хеш таблицы записываем в результат

```go
func groupAnagrams(strs []string) [][]string {
	anagrams := make(map[[26]int][]string) 

	for _, str := range strs {
		anagrams[indexAnagram(str)] = append(anagrams[indexAnagram(str)], str)
	}
	
	result := make([][]string, 0)

	for _, anagram := range anagrams {
		result = append(result, anagram)
	}

	return result
}

  

func indexAnagram(str string) [26]int {
	code := [26]int{}
	
	for _, str := range str {
		code[str-'a']++
	}

	return code
}
```

**Сложность алгоритма: O(m∗n)**

#Arrays #Medium