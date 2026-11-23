Given a string `s` which consists of lowercase or uppercase letters, return the length of the **longest palindrome** that can be built with those letters.

Letters are **case sensitive**, for example, `"Aa"` is not considered a palindrome.

**Example 1:**

**Input:** s = "abccccdd"
**Output:** 7
**Explanation:** One longest palindrome that can be built is "dccaccd", whose length is 7.

**Example 2:**

**Input:** s = "a"
**Output:** 1
**Explanation:** The longest palindrome that can be built is "a", whose length is 1.

**Constraints:**

- `1 <= s.length <= 2000`
- `s` consists of lowercase **and/or** uppercase English letters only.

## Array

1) Создаём массив на 52 элемента
2) Идём по строке в цикле
3) Если это буква между 'a' и 'z', вычисляем её индекс как `char -'a'`, если нет `char - 'A' + 26` и увеличиваем значение по этому индексу
4) Объявляем переменные для результата и проверки на наличии нечётного количества символов
5) В цикле увеличиваем результат на большее чётное от количества на данном индексе
6) Если количество нечётное, ставим флаг для определения встречи нечётного числа символов в `true`
7) Выходим из цикла, и если было нечётное количество, то добавляем единицу к результату

```go
func longestPalindrome(s string) int {
    counts := [52]int{}

    for _, ch := range s {
        if ch >= 'a' && ch <= 'z' {
            counts[ch - 'a']++
        } else {
            counts[ch-'A'+26]++
        }
    }

    var result int
    var hasOdd bool

    for _, count := range counts {
        result += count / 2 * 2

        if count%2 == 1 {
            hasOdd = true
        }
    }

    if hasOdd {
        result++
    }

    return result
}
```

#two_pointers #Easy