Given a string `s`, return `true` if it is a **palindrome**, otherwise return `false`.

A **palindrome** is a string that reads the same forward and backward. It is also case-insensitive and ignores all non-alphanumeric characters.

**Note:** Alphanumeric characters consist of letters `(A-Z, a-z)` and numbers `(0-9)`.

**Example 1:**

```java
Input: s = "Was it a car or a cat I saw?"

Output: true
```

Explanation: After considering only alphanumerical characters we have "wasitacaroracatisaw", which is a palindrome.

**Example 2:**

```java
Input: s = "tab a cat"

Output: false
```

Explanation: "tabacat" is not a palindrome.

**Constraints:**

- `1 <= s.length <= 1000`
- `s` is made up of only printable ASCII characters.

## Two pointers

1) Проверяем на длину, если меньше двух, значит это уже валидный палиндром
2) Создаём два указателя, на начала и конец строки
3) Пока левый указатель меньше чем правый выполняем цикл
4) Пока левый указатель не является буквой или числом и пока он меньше правого увеличиваем его, так же с правым, только уменьшаем
5) Если символ на левом указателе, и правом не равны, значит это не палиндром, возвращаем `false`
6) Иначе увеличиваем левый указатель и уменьшаем правый
7) Если вышли из цикла, значит это палиндром возвращаем `true`

```go
func isPalindrome(s string) bool {
	if len(s) < 2 {
		return true
	}
	
	s = strings.ToLower(s)
	var l int
	r := len(s) - 1
	
	for l < r {
		for !(unicode.IsLetter(rune(s[l])) || 
		unicode.IsDigit(rune(s[l]))) && l < r {
			l++
		}
		for !(unicode.IsLetter(rune(s[r])) || 
		unicode.IsDigit(rune(s[r]))) && r > l {
			r--
		} 
		if rune(s[l]) != rune(s[r]) {
			return false
		}
		l++
		r--
	}
	
	return true
}

```

#two_pointers #Easy 