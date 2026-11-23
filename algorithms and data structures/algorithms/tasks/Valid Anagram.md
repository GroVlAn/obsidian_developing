Given two strings `s` and `t`, return `true` if the two strings are anagrams of each other, otherwise return `false`.

An **anagram** is a string that contains the exact same characters as another string, but the order of the characters can be different.

**Example 1:**

```java
Input: s = "racecar", t = "carrace"

Output: true
```

**Example 2:**

```java
Input: s = "jar", t = "jam"

Output: false
```

## Hash Table (Using Array)

1) Проверяем длину строк, если она отличается возвращаем `false`
2) Создаём массив длиной равной 26, так как в задаче используется только английский алфавит
3) В цикле от 0 до длины первой строки берём индекс нашего массива вычисляемого вычитанием из текущего символа кода символа 'a' `s[i]-'a'` и `t[i]-'a'`
4) Увеличиваем значение вычисленного индекса символа из первой строки – s и вычитаем из вычисленного индекса второй строки – t
5) Идём циклом по массиву подсчёта символов, если хотя бы один элемент массива не 0, значит это не анаграммы, и возвращаем `false`
6) Если мы вышли из цикла, значит это анаграммы, возвращаем `true`

```go
func isAnagram(s string, t string) bool {
	if len(s) != len(t) {
		return false
	}

	characters := [26]int{}

	for i := 0; i < len(s); i++ {
		characters[s[i]-'a']++
		characters[t[i]-'a']--
	}

	for _, ch := range characters {
		if ch != 0 {
			return false
		}
	}

	return true
}
```

**Сложность алгоритма по времени  O(n+m)**
**Сложность по памяти O(1) // так как всего 26 символов**

## Hash Map

1) Проверяем длину строк, если она отличается возвращаем `false`
2) Создаём две хеш таблицы, где ключ это руна, а значение это число
3) Увеличиваем значение в хеш таблице по ключу, где ключ для первой хеш таблицы будет символ первой строки, а ключ для второй хеш таблицы символ находящийся на i-ом индексе второй строки
4) Циклом идём по первой хеш таблице, проверяем текущее значение хеш таблицы со значение с таким же ключом из второй таблицы, если они не совпадают, значит это не анаграммы и возвращаем `false`
5) Если вышли из цикла, значит это анаграммы и возвращаем `true`

```go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false
    }

    countS, countT := make(map[rune]int), make(map[rune]int)
    for i, ch := range s {
        countS[ch]++
        countT[rune(t[i])]++
    }

    for k, v := range countS {
        if countT[k] != v {
            return false
        }
    }
    return true
}
```

**Сложность алгоритма по времени  O(n+m)**
**Сложность по памяти O(1) // так как всего 26 символов**

#Arrays #Easy