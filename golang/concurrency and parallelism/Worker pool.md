Организует набор горутин для параллельной обработки задач, что особенно полезно для оптимизации производительности.

```go
package blocks

import (
	"context"
	"sync"
)  

func WorkerPool[T, R any](
	ctx context.Context,
	workerCount int,
	input <-chan T,
	transform func(ctx context.Context, e T) R,
) <-chan R {
	result := make(chan R)
	var wg sync.WaitGroup
	
	for range workerCount {
		wg.Add(1)
		
		go func() {
			defer wg.Done()  
			
			for {
				select {
				case <-ctx.Done():
					return
				case v, ok := <-input:
					if !ok {
						return
					}  
					
					r := transform(ctx, v)
					
					select {
					case <-ctx.Done():
						return
					case result <- r:
					}
				}
			}
		}()
	}
	
	go func() {
		wg.Wait()
		close(result)
	}()
	
	return result
}
```