You are given a `9 x 9` Sudoku board `board`. A Sudoku board is valid if the following rules are followed:

1. Each row must contain the digits `1-9` without duplicates.
2. Each column must contain the digits `1-9` without duplicates.
3. Each of the nine `3 x 3` sub-boxes of the grid must contain the digits `1-9` without duplicates.

Return `true` if the Sudoku board is valid, otherwise return `false`

**Note:** A board does not need to be full or be solvable to be valid.

**Example 1:**

![](https://imagedelivery.net/CLfkmk9Wzy8_9HRyug4EVA/0be40c5d-2d18-42b8-261b-13ca50de4100/public)

```java
Input: board =
[["1","2",".",".","3",".",".",".","."],
 ["4",".",".","5",".",".",".",".","."],
 [".","9","8",".",".",".",".",".","3"],
 ["5",".",".",".","6",".",".",".","4"],
 [".",".",".","8",".","3",".",".","5"],
 ["7",".",".",".","2",".",".",".","6"],
 [".",".",".",".",".",".","2",".","."],
 [".",".",".","4","1","9",".",".","8"],
 [".",".",".",".","8",".",".","7","9"]]

Output: true
```

**Example 2:**

```java
Input: board =
[["1","2",".",".","3",".",".",".","."],
 ["4",".",".","5",".",".",".",".","."],
 [".","9","1",".",".",".",".",".","3"],
 ["5",".",".",".","6",".",".",".","4"],
 [".",".",".","8",".","3",".",".","5"],
 ["7",".",".",".","2",".",".",".","6"],
 [".",".",".",".",".",".","2",".","."],
 [".",".",".","4","1","9",".",".","8"],
 [".",".",".",".","8",".",".","7","9"]]

Output: false
```

Explanation: There are two 1's in the top-left 3x3 sub-box.

**Constraints:**

- `board.length == 9`
- `board[i].length == 9`
- `board[i][j]` is a digit `1-9` or `'.'`.

## Hash set

1) Создаём массивы хеш таблицы размером 9 для строк, столбцов и квадратов судоку
2) Инициализируем хеш таблицы для всех массивов
3) Идёт по матрице двумя циклами
4) Если текущий эллемент `"."`, пропускаем ячейку
5) Проверяем есть ли уже значения в строках и столбца наших хеш сетов
6) Проверяем в квадрате, вычислив условный индекс квадрата: `(индекс строки/3)*3+индекс столбца/3`
7) Если значение есть хоть в одном из хеш сетов, то возвращаем `false`
8) Если нет, записываем значение в каждый хеш сет
9) Выходим из цикла и возвращаем `true`

```go
func isValidSudoku(board [][]byte) bool {
	cols := make([]map[byte]struct{}, 9)
	rows := make([]map[byte]struct{}, 9)
	boxes := make([]map[byte]struct{}, 9)

	for i := 0; i < 9; i++ {
		cols[i] = make(map[byte]struct{})
		rows[i] = make(map[byte]struct{})
		boxes[i] = make(map[byte]struct{})
	}

	for idxRow, row := range board {
		for idxCol, col := range row {
			if col == '.' {
				continue
			}

			squareIdx := (idxRow/3)*3 + idxCol/3 

			_, okCol := cols[idxCol][col]
			_, okRow := rows[idxRow][col]
			_, okBox := boxes[squareIdx][col] 

			if okCol || okRow || okBox {
				return false
			}

			cols[idxCol][col] = struct{}{}
			rows[idxRow][col] = struct{}{}
			boxes[squareIdx][col] = struct{}{}
		}
	}

	return true
}
```
**Сложность алгоритма: O(n2)**

#Arrays #Medium