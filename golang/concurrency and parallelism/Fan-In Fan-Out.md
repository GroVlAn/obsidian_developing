Паттерн Fan-Out позволяет распределить вычисления на несколько горутин, для параллельной обработки данных. Таким образом ты можешь увеличить пропускную способность.

Предположим что у тебя есть функция add(), которая требует много ресурсов для выполнения. Чтобы оптимизировать ее работу, можно увеличить количество горутин.

## Fan-out
```go
package main

import (
    "fmt"
    "time"
)

// generator — создает канал с данными
func generator(doneCh chan struct{}, numbers []int) chan int {
    dataStream := make(chan int)

    go func() {
        defer close(dataStream)
        for _, num := range numbers {
            select {
            case <-doneCh:
                return
            case dataStream <- num:
            }
        }
    }()

    return dataStream
}

// add — добавляет 1 к каждому значению
func add(doneCh chan struct{}, inputCh chan int) chan int {
    resultStream := make(chan int)

    go func() {
        defer close(resultStream)
        for num := range inputCh {
            // Имитация более затратной работы
            time.Sleep(time.Second)
            result := num + 1

            select {
            case <-doneCh:
                return
            case resultStream <- result:
            }
        }
    }()

    return resultStream
}

// multiply — умножает каждое значение на 2
func multiply(doneCh chan struct{}, inputCh chan int) chan int {
    resultStream := make(chan int)

    go func() {
        defer close(resultStream)
        for num := range inputCh {
            result := num * 2

            select {
            case <-doneCh:
                return
            case resultStream <- result:
            }
        }
    }()

    return resultStream
}

// fanOut — создает несколько горутин add для параллельной обработки данных
func fanOut(doneCh chan struct{}, inputCh chan int, workers int) []chan int {
    resultChannels := make([]chan int, workers)
    
    for i := 0; i < workers; i++ {
        resultChannels[i] = add(doneCh, inputCh)
    }

    return resultChannels
}
```

## Fan-in

```go
func fanIn(doneCh chan struct{}, channels ...chan int) chan int {
    finalStream := make(chan int)
    var wg sync.WaitGroup

    for _, ch := range channels {
        chCopy := ch //old version go
        wg.Add(1)
        
        go func() {
            defer wg.Done()
            for value := range chCopy {
                select {
                case <-doneCh:
                    return
                case finalStream <- value:
                }
            }
        }()
    }

    go func() {
        wg.Wait()
        close(finalStream)
    }()

    return finalStream
}
```