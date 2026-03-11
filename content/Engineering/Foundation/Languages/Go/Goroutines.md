---
title: "Goroutines"
tags:
  - go
  - goroutines
  - concurrency
  - parallelism
  - performance
---

# Goroutines trong Go

Goroutines là một khái niệm trong ngôn ngữ lập trình Go, được thiết kế để xử lý các tác vụ đồng thời một cách hiệu quả và dễ dàng. Goroutines cung cấp một phương pháp nhẹ nhàng để chạy các chức năng đồng thời và quản lý nhiều tác vụ mà không gặp phải các vấn đề phức tạp về luồng truyền thống.

## Đặc điểm của Goroutines

### 1. Nhẹ và hiệu quả

Goroutine rất nhẹ và tiêu thụ ít tài nguyên của hệ thống hơn so với luồng (threads) truyền thống. Một số chương trình Go có thể chạy hàng ngàn Goroutines mà không gặp vấn đề tài nguyên.

**So sánh Stack Size:**
- **Thread**: 1MB - 8MB trong vùng nhớ stack (tùy ngôn ngữ), **không thể resize**
- **Goroutine**: Chỉ **2KB** khi khởi tạo và **có thể resize** động

```go
// Ví dụ: Chạy 10,000 goroutines
func main() {
    for i := 0; i < 10000; i++ {
        go func(id int) {
            time.Sleep(time.Second)
            fmt.Printf("Goroutine %d completed\n", id)
        }(i)
    }
    
    time.Sleep(2 * time.Second)
    // Tổng memory footprint vẫn rất nhỏ!
}
```

### 2. Quản lý tự động

Việc quản lý Goroutines được thực hiện bởi **Go runtime scheduler**, giúp tối ưu hóa hiệu suất và đảm bảo tính ổn định.

**Go Scheduler:**
- **M:N scheduling**: M goroutines được map vào N OS threads
- **Work stealing**: Cân bằng tải tự động giữa các threads
- **Cooperative scheduling**: Goroutines tự động yield tại các điểm an toàn

```go
// Runtime tự động quản lý việc schedule goroutines
runtime.GOMAXPROCS(4) // Sử dụng 4 OS threads
```

### 3. Giao tiếp qua kênh (Channel)

Goroutines thường giao tiếp với nhau thông qua [[Channels|📡 channels]], một phương thức an toàn và hiệu quả để truyền dữ liệu giữa các Goroutines.

```go
func main() {
    ch := make(chan string)
    
    go func() {
        ch <- "Hello from goroutine"
    }()
    
    msg := <-ch
    fmt.Println(msg)
}
```

## Cách sử dụng Goroutines

### Ví dụ cơ bản

```go
package main

import (
    "fmt"
    "time"
)

func sayHello() {
    fmt.Println("Hello")
}

func main() {
    // Khởi chạy một Goroutine
    go sayHello()

    // Chờ một chút để Goroutine có thời gian hoàn thành
    time.Sleep(time.Second)
    fmt.Println("World")
}
```

**Output:**
```
Hello
World
```

### Ví dụ với Anonymous Function

```go
func main() {
    go func() {
        fmt.Println("Hello from anonymous goroutine")
    }()
    
    time.Sleep(time.Second)
}
```

### Ví dụ với Parameters

```go
func printNumbers(name string, numbers []int) {
    for _, num := range numbers {
        fmt.Printf("%s: %d\n", name, num)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go printNumbers("Goroutine 1", []int{1, 2, 3, 4, 5})
    go printNumbers("Goroutine 2", []int{10, 20, 30, 40, 50})
    
    time.Sleep(time.Second)
}
```

## So sánh Goroutines với Threads và Coroutines

| Tiêu chí | Goroutines | Threads | Coroutines |
|----------|------------|---------|------------|
| **Nhẹ và hiệu quả** | Rất nhẹ và hiệu quả (2KB) | Tốn tài nguyên hơn (1-8MB) | Nhẹ và hiệu quả hơn threads |
| **Quản lý tài nguyên** | Tự động quản lý bởi Go runtime | Yêu cầu hệ điều hành quản lý | Quản lý bởi runtime hoặc thư viện |
| **Giao tiếp** | Thông qua channels | Thông qua bộ nhớ chung và mutex | Không có cơ chế giao tiếp chuẩn |
| **Context switching** | Nhanh và nhẹ nhàng (~200ns) | Tốn kém hơn (~1-2μs) | Nhẹ nhàng hơn threads |
| **Ứng dụng** | Tác vụ đồng thời nhẹ và I/O | Tác vụ nặng và tính toán | Tác vụ đồng thời nhẹ và I/O |
| **Scalability** | Có thể chạy hàng nghìn cùng lúc | Giới hạn bởi OS resources | Phụ thuộc implementation |
| **Preemptive** | Cooperative (Go 1.14+: preemptive) | Preemptive | Cooperative |

## Khi nào nên sử dụng Goroutines

### 1. Xử lý I/O không đồng bộ

Goroutines rất phù hợp cho các tác vụ I/O không đồng bộ như truy cập mạng, đọc/ghi tệp, nơi mà thời gian chờ là chủ yếu.

```go
func fetchURL(url string, ch chan<- string) {
    resp, err := http.Get(url)
    if err != nil {
        ch <- fmt.Sprintf("Error: %v", err)
        return
    }
    defer resp.Body.Close()
    
    ch <- fmt.Sprintf("Fetched %s: Status %d", url, resp.StatusCode)
}

func main() {
    urls := []string{
        "https://google.com",
        "https://github.com",
        "https://golang.org",
    }
    
    ch := make(chan string)
    
    for _, url := range urls {
        go fetchURL(url, ch)
    }
    
    for range urls {
        fmt.Println(<-ch)
    }
}
```

### 2. Tác vụ nền nhẹ nhàng

Các tác vụ nền không yêu cầu nhiều tài nguyên và không cần phải tính toán nặng.

```go
func backgroundLogger(messages <-chan string) {
    for msg := range messages {
        log.Printf("[BACKGROUND] %s", msg)
    }
}

func main() {
    logChan := make(chan string, 100)
    
    // Start background logger
    go backgroundLogger(logChan)
    
    // Main application logic
    logChan <- "Application started"
    // ... do work ...
    logChan <- "Processing complete"
}
```

### 3. Xử lý đồng thời

Khi cần quản lý nhiều tác vụ đồng thời mà không gặp vấn đề phức tạp của đa luồng.

```go
func processData(data []int) int {
    sum := 0
    for _, v := range data {
        sum += v
    }
    return sum
}

func main() {
    data := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    mid := len(data) / 2
    
    ch := make(chan int)
    
    // Process first half
    go func() {
        ch <- processData(data[:mid])
    }()
    
    // Process second half
    go func() {
        ch <- processData(data[mid:])
    }()
    
    sum1, sum2 := <-ch, <-ch
    fmt.Printf("Total: %d\n", sum1+sum2)
}
```

### 4. Ứng dụng máy chủ (Server-side applications)

Máy chủ web, máy chủ API, nơi mà việc xử lý nhiều kết nối cùng một lúc là cần thiết.

```go
func handleConnection(conn net.Conn) {
    defer conn.Close()
    
    // Handle request
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Println("Error reading:", err)
        return
    }
    
    // Process and respond
    response := fmt.Sprintf("Received: %s", string(buffer[:n]))
    conn.Write([]byte(response))
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatal(err)
    }
    defer listener.Close()
    
    fmt.Println("Server listening on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Println("Error accepting:", err)
            continue
        }
        
        // Handle each connection in a separate goroutine
        go handleConnection(conn)
    }
}
```

## Patterns với Goroutines

### 1. WaitGroup Pattern

Đợi nhiều goroutines hoàn thành.

```go
func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }
    
    wg.Wait()
    fmt.Println("All workers completed")
}
```

### 2. Worker Pool Pattern

Giới hạn số lượng goroutines đang chạy.

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job)
        time.Sleep(time.Second)
        results <- job * 2
    }
}

func main() {
    numJobs := 10
    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)
    
    // Start 3 workers
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    
    // Send jobs
    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)
    
    // Collect results
    for a := 1; a <= numJobs; a++ {
        fmt.Printf("Result: %d\n", <-results)
    }
}
```

### 3. Timeout Pattern

Giới hạn thời gian chờ của goroutine.

```go
func longRunningTask(result chan<- string) {
    time.Sleep(3 * time.Second)
    result <- "Task completed"
}

func main() {
    result := make(chan string)
    
    go longRunningTask(result)
    
    select {
    case res := <-result:
        fmt.Println(res)
    case <-time.After(2 * time.Second):
        fmt.Println("Timeout!")
    }
}
```

### 4. Pipeline Pattern

Kết nối nhiều goroutines thành pipeline.

```go
func generator(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

func main() {
    // Pipeline: generator -> square
    nums := generator(1, 2, 3, 4, 5)
    squares := square(nums)
    
    for s := range squares {
        fmt.Println(s) // Output: 1 4 9 16 25
    }
}
```

## Synchronization và Communication

### 1. Channels (Preferred)

```go
// Unbuffered channel - synchronous
ch := make(chan int)

// Buffered channel - asynchronous
ch := make(chan int, 10)
```

### 2. Mutex

```go
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}
```

### 3. Atomic Operations

```go
var counter int64

func increment() {
    atomic.AddInt64(&counter, 1)
}
```

## Best Practices

### 1. Tránh Goroutine Leaks

```go
// Bad - goroutine leak
func leak() <-chan int {
    ch := make(chan int)
    go func() {
        ch <- 42 // Block forever nếu không có receiver
    }()
    return ch
}

// Good - sử dụng context
func noLeak(ctx context.Context) <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch)
        select {
        case ch <- 42:
        case <-ctx.Done():
            return
        }
    }()
    return ch
}
```

### 2. Graceful Shutdown

```go
func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d shutting down\n", id)
            return
        default:
            // Do work
            time.Sleep(100 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }
    
    time.Sleep(2 * time.Second)
    cancel() // Graceful shutdown
    time.Sleep(time.Second)
}
```

### 3. Error Handling

```go
type Result struct {
    Value int
    Err   error
}

func worker(id int) Result {
    if id < 0 {
        return Result{Err: fmt.Errorf("invalid id: %d", id)}
    }
    return Result{Value: id * 2}
}

func main() {
    results := make(chan Result)
    
    for i := -1; i < 3; i++ {
        go func(id int) {
            results <- worker(id)
        }(i)
    }
    
    for i := 0; i < 4; i++ {
        result := <-results
        if result.Err != nil {
            fmt.Printf("Error: %v\n", result.Err)
        } else {
            fmt.Printf("Result: %d\n", result.Value)
        }
    }
}
```

### 4. Đặt tên rõ ràng

```go
// Good
go processUserData(userID)
go fetchAPIData(endpoint)

// Bad
go func() { /* ... */ }()  // Anonymous, khó debug
```

### 5. Limit số lượng Goroutines

```go
// Sử dụng semaphore hoặc worker pool
sem := make(chan struct{}, 10) // Max 10 concurrent goroutines

for i := 0; i < 1000; i++ {
    sem <- struct{}{}  // Acquire
    go func(id int) {
        defer func() { <-sem }()  // Release
        processTask(id)
    }(i)
}
```

## Debugging Goroutines

### 1. Runtime Information

```go
fmt.Println("NumGoroutine:", runtime.NumGoroutine())
fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))
```

### 2. Stack Traces

```go
// Print all goroutine stacks
buf := make([]byte, 1<<16)
stackSize := runtime.Stack(buf, true)
fmt.Printf("%s\n", buf[:stackSize])
```

### 3. Race Detector

```bash
go run -race main.go
go test -race
```

## Performance Tips

### 1. Goroutine Pool cho Short-lived Tasks

```go
// Thay vì tạo goroutine mới cho mỗi task
for range tasks {
    go process() // Expensive!
}

// Dùng worker pool
for w := 0; w < numWorkers; w++ {
    go worker(tasks) // Reuse goroutines
}
```

### 2. Buffered Channels cho High Throughput

```go
// Unbuffered - nhiều context switching
ch := make(chan int)

// Buffered - giảm blocking
ch := make(chan int, 1000)
```

### 3. Avoid Goroutines cho Simple Tasks

```go
// Overkill
go fmt.Println("Hello")

// Just do it
fmt.Println("Hello")
```

## Tham khảo

- [[Channels|📡 Channels]]
- [[Context|🎯 Context Package]]
- [[Pointers|🎯 Pointers]]
- [Concurrency in Go](https://go.dev/blog/pipelines)
- [Go Concurrency Patterns](https://go.dev/talks/2012/concurrency.slide)

## Tags liên quan

#go #goroutines #concurrency #parallelism #performance #patterns #best-practices
