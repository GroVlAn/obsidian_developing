
Design an algorithm to encode **a list of strings** to **a string**. The **encoded string** is then sent over the network and is **decoded** back to the **original list** of strings.

**Machine 1 (sender)** has the function:

```java
String encode(List<String> strs) {
    // ... your code
    return encoded_string;
}
```

**Machine 2 (receiver)** has the function:

```java
List<String> decode(String encoded_string) {
    // ... your code
    return decoded_strs;
}
```

So **Machine 1** does:

```java
String encoded_string = encode(strs);
```

and **Machine 2** does:

```java
List<String> decoded_strs = decode(encoded_string);
```

`decoded_strs` in Machine 2 should be the **same** as the input `strs` in Machine 1.

Implement the `encode` and `decode` methods.

**Example 1:**

```java
Input: strs = ["Hello","World"]

Output: ["Hello","World"]
```

**Explanation:**

```java
Solution solution = new Solution();
String encoded_string = solution.encode(strs);

// Machine 1 ---encoded_string---> Machine 2

List<String> decoded_strs = solution.decode(encoded_string);
```

  

**Example 2:**

```java
Input: strs = [""]

Output: [""]
```

  

**Constraints:**

- `0 <= strs.length < 100`
- `0 <= strs[i].length < 200`
- `strs[i]` contains any possible characters out of `256` valid ASCII characters.

## Solution

```go
type Solution struct {
	Delimiter string
}

func (s *Solution) Encode(strs []string) string {
	if len(s.Delimiter) == 0 {
		s.Delimiter = "|"
	}
	
	var sb strings.Builder

	for _, word := range strs {
		sb.WriteString(strconv.Itoa(len(word)))
		sb.WriteString(s.Delimiter)
		sb.WriteString(word)
	}

	return sb.String()
}

func (s *Solution) Decode(encoded string) []string {
	if len(s.Delimiter) == 0 {
		s.Delimiter = "|"
	}

	lastPos := 0

	res := make([]string, 0)

	for lastPos < len(encoded) {
		numPos := lastPos
		for string(encoded[numPos]) != s.Delimiter {
			numPos++
		} 

		currentLen, _ := strconv.Atoi(encoded[lastPos:numPos])
		res = append(res, encoded[numPos+1:numPos+currentLen+1])
		lastPos = numPos+currentLen + 1
	} 

	return res
}
```

#Arrays #Medium