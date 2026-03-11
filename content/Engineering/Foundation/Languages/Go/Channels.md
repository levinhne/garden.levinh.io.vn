---
title: "Channels"
tags:
  - go
  - channels
  - concurrency
  - goroutines
  - communication
---

# Channels trong Go

Trong ngôn ngữ lập trình Go, channel là một cơ chế để giao tiếp giữa các goroutine, được sử dụng để truyền dữ liệu từ một goroutine này sang goroutine khác.

> **"Don't communicate by sharing memory; share memory by communicating."** - Go Proverb

## Các loại Channel chính

### 1. Unbuffered Channel (Channel không có bộ nhớ đệm)

Khi một giá trị được gửi vào unbuffered channel, goroutine gửi phải đợi đến khi có một goroutine khác nhận giá trị đó. Nếu không có goroutine nhận giá trị đó thì chương trình bị deadlock và panic.

```go
ch := make(chan int)  // Unbuffered channel

go func() {
    ch <- 42  // Gửi giá trị 42 vào channel (block cho tới khi có receiver)
}()

value := <-ch  // Nhận giá trị từ channel
fmt.Println(value)  // Output: 42
```

**Đặc điểm:**
- Synchronous: Sender và receiver phải đồng bộ
- Zero capacity: Không lưu trữ giá trị
- Blocking: Send block cho đến khi có receive

**Ví dụ Deadlock:**
```go
ch := make(chan int)
ch <- 42  // Deadlock! Không có goroutine nào receive
```

### 2. Buffered Channel (Channel có bộ nhớ đệm)

Có thể lưu trữ nhiều giá trị trước khi bắt đầu blocking.

Luồng xử lý chỉ bị block trong hai trường hợp:
- Nếu channel rỗng → goroutine đọc ra sẽ chờ cho tới khi có dữ liệu được đẩy vào để lấy ra.
- Nếu channel đầy → goroutine gửi vào sẽ chờ cho tới khi có chỗ trống để đẩy dữ liệu vào.

```go
ch := make(chan string, 2)  // Buffered channel với bộ đệm 2

ch <- "hello"  // Không block
ch <- "world"  // Không block
// ch <- "!" // Sẽ block vì buffer đầy

fmt.Println(<-ch)  // Output: "hello"
fmt.Println(<-ch)  // Output: "world"
```

**Kiểm tra trạng thái buffer:**
```go
ch := make(chan int, 3)
ch <- 1
ch <- 2

fmt.Println("Length:", len(ch))      // Output: 2
fmt.Println("Capacity:", cap(ch))    // Output: 3
```

### 3. Directional Channel (Channel có hướng)

Định nghĩa hướng gửi và nhận dữ liệu của channel để tăng type safety.

```go
// Send-only channel
func send(ch chan<- int, value int) {
    ch <- value
    // value := <-ch  // Compile error! Cannot receive from send-only channel
}

// Receive-only channel
func receive(ch <-chan int) int {
    return <-ch
    // ch <- 42  // Compile error! Cannot send to receive-only channel
}

func main() {
    ch := make(chan int)
    go send(ch, 42)
    fmt.Println(receive(ch))  // Output: 42
}
```

**Ví dụ Pipeline:**
```go
func producer(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}

func consumer(ch <-chan int) {
    for num := range ch {
        fmt.Println("Consumed:", num)
    }
}

func main() {
    ch := make(chan int)
    go producer(ch)
    consumer(ch)
}
```

### 4. Select Statement (Lệnh select)

Cho phép chọn lựa trên nhiều channel cùng một lúc.

**Đặc điểm:**
- Select sẽ block chương trình lại cho tới khi có một operation ready.
- Có nhiều hơn một operation ready đồng thời → sẽ chọn random.
- Nếu Select không có mệnh đề nào sẽ dẫn đến panic.
- Nếu tất cả goroutine trong mệnh đề select bị block sẽ dẫn đến panic.

```go
ch1 := make(chan string)
ch2 := make(chan string)

go func() {
    time.Sleep(1 * time.Second)
    ch1 <- "message from channel 1"
}()

go func() {
    time.Sleep(2 * time.Second)
    ch2 <- "message from channel 2"
}()

select {
case msg1 := <-ch1:
    fmt.Println("Received", msg1)
case msg2 := <-ch2:
    fmt.Println("Received", msg2)
}
// Output: Received message from channel 1 (vì ch1 sẵn sàng trước)
```

**Select với Default:**
```go
select {
case msg := <-ch:
    fmt.Println("Received:", msg)
default:
    fmt.Println("No message received")  // Không block
}
```

**Select với Timeout:**
```go
select {
case msg := <-ch:
    fmt.Println("Received:", msg)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout!")
}
```

### 5. Closing Channels (Đóng channel)

Được sử dụng để thông báo rằng không còn dữ liệu được gửi vào channel nữa.

```go
ch := make(chan int)

go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)  // Đóng channel sau khi gửi xong
}()

for num := range ch {
    fmt.Println(num)  // Output: 0 1 2 3 4
}
```

**Kiểm tra channel đã đóng:**
```go
ch := make(chan int, 1)
ch <- 42
close(ch)

value, ok := <-ch
if ok {
    fmt.Println("Received:", value)  // Output: Received: 42
} else {
    fmt.Println("Channel closed")
}

value2, ok2 := <-ch
fmt.Println(value2, ok2)  // Output: 0 false (zero value và closed)
```

## Patterns và Use Cases

### 1. Fan-Out, Fan-In Pattern

**Fan-Out:** Phân phối công việc cho nhiều workers
```go
func fanOut(input <-chan int, numWorkers int) []<-chan int {
    channels := make([]<-chan int, numWorkers)
    
    for i := 0; i < numWorkers; i++ {
        channels[i] = worker(input, i)
    }
    
    return channels
}

func worker(input <-chan int, id int) <-chan int {
    output := make(chan int)
    
    go func() {
        defer close(output)
        for num := range input {
            result := num * 2  // Process data
            fmt.Printf("Worker %d processed: %d\n", id, result)
            output <- result
        }
    }()
    
    return output
}
```

**Fan-In:** Kết hợp kết quả từ nhiều channels
```go
func fanIn(channels ...<-chan int) <-chan int {
    output := make(chan int)
    var wg sync.WaitGroup
    
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for num := range c {
                output <- num
            }
        }(ch)
    }
    
    go func() {
        wg.Wait()
        close(output)
    }()
    
    return output
}
```

### 2. Pipeline Pattern

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
    
    for result := range squares {
        fmt.Println(result)  // Output: 1 4 9 16 25
    }
}
```

### 3. Worker Pool Pattern

```go
type Job struct {
    ID   int
    Data string
}

type Result struct {
    Job    Job
    Output string
}

func worker(id int, jobs <-chan Job, results chan<- Result) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job.ID)
        time.Sleep(time.Second)
        
        result := Result{
            Job:    job,
            Output: fmt.Sprintf("Processed: %s", job.Data),
        }
        results <- result
    }
}

func main() {
    numWorkers := 3
    jobs := make(chan Job, 10)
    results := make(chan Result, 10)
    
    // Start workers
    for i := 1; i <= numWorkers; i++ {
        go worker(i, jobs, results)
    }
    
    // Send jobs
    for i := 1; i <= 5; i++ {
        jobs <- Job{ID: i, Data: fmt.Sprintf("task-%d", i)}
    }
    close(jobs)
    
    // Collect results
    for i := 1; i <= 5; i++ {
        result := <-results
        fmt.Printf("Result: %s\n", result.Output)
    }
}
```

### 4. Done Channel Pattern

```go
func doWork(done <-chan bool) <-chan int {
    result := make(chan int)
    
    go func() {
        defer close(result)
        for {
            select {
            case <-done:
                return
            case result <- rand.Intn(100):
                time.Sleep(100 * time.Millisecond)
            }
        }
    }()
    
    return result
}

func main() {
    done := make(chan bool)
    result := doWork(done)
    
    for i := 0; i < 5; i++ {
        fmt.Println(<-result)
    }
    
    close(done)  // Signal to stop
}
```

### 5. Rate Limiting với Channel

```go
func rateLimiter(requests <-chan int, limit int, duration time.Duration) <-chan int {
    results := make(chan int)
    ticker := time.NewTicker(duration / time.Duration(limit))
    
    go func() {
        defer close(results)
        defer ticker.Stop()
        
        for req := range requests {
            <-ticker.C  // Wait for rate limit
            results <- req
        }
    }()
    
    return results
}

func main() {
    requests := make(chan int, 10)
    
    // Send 10 requests
    for i := 1; i <= 10; i++ {
        requests <- i
    }
    close(requests)
    
    // Rate limit: 5 requests per second
    limited := rateLimiter(requests, 5, time.Second)
    
    start := time.Now()
    for req := range limited {
        fmt.Printf("Request %d processed at %v\n", req, time.Since(start))
    }
}
```

## Best Practices

### 1. Sender đóng channel, không phải receiver
```go
// Good
go func() {
    defer close(ch)
    for i := 0; i < 10; i++ {
        ch <- i
    }
}()

// Bad - receiver không nên close channel
for val := range ch {
    // ...
}
close(ch)  // Dangerous!
```

### 2. Không gửi vào channel đã đóng
```go
ch := make(chan int)
close(ch)
// ch <- 1  // Panic: send on closed channel
```

### 3. Đọc từ channel đã đóng là an toàn
```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
close(ch)

fmt.Println(<-ch)  // Output: 1
fmt.Println(<-ch)  // Output: 2
fmt.Println(<-ch)  // Output: 0 (zero value)
```

### 4. Sử dụng buffered channel cho producer-consumer
```go
// Buffer size = số lượng items có thể produce trước khi block
ch := make(chan int, 100)
```

### 5. Tránh goroutine leaks
```go
// Bad - goroutine leak nếu không đọc từ channel
func leak() <-chan int {
    ch := make(chan int)
    go func() {
        ch <- 42  // Block forever nếu không có receiver
    }()
    return ch
}

// Good - sử dụng context hoặc done channel
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

## Channel vs Mutex

| Channel | Mutex |
|---------|-------|
| Truyền ownership của dữ liệu | Shared memory với lock |
| Higher level abstraction | Lower level primitive |
| Communicate by passing data | Protect shared data |
| Goroutine-safe by design | Requires careful locking |

**Khi nào dùng Channel:**
- Passing ownership
- Distributing work
- Communication giữa goroutines

**Khi nào dùng Mutex:**
- Caching
- State management
- Đơn giản hơn cho shared state

## Tham khảo

- [[Context|🎯 Context Package]]
- [[Pointers|🎯 Pointers]]
- [Go Concurrency Patterns](https://go.dev/blog/pipelines)
- [Share Memory By Communicating](https://go.dev/blog/codelab-share)

## Tags liên quan

#go #channels #concurrency #goroutines #patterns #communication #synchronization
