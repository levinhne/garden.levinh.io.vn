---
title: "Context Package"
tags:
  - go
  - context
  - concurrency
  - goroutines
  - cancellation
---

# Context Package trong Go

Context trong Go là một gói chuẩn được sử dụng để truyền thông tin về thời gian hết hạn, tín hiệu hủy, và các giá trị khác giữa goroutines. Điều này rất hữu ích khi bạn cần quản lý và điều phối nhiều goroutines trong một ứng dụng.

## Khái niệm cơ bản

### 1. Tạo Context

#### `context.Background()`
Trả về một `Context` trống, thường được sử dụng như là root context trong ứng dụng.

```go
ctx := context.Background()
```

#### `context.TODO()`
Trả về `Context` trống nhưng được sử dụng khi bạn chưa chắc chắn về context cần dùng.

```go
ctx := context.TODO()
```

### 2. Context with Cancel

Sử dụng `context.WithCancel(parent context)` để tạo ra một context con có thể bị hủy bỏ. Khi bạn gọi hàm cancel, tất cả các context con cũng bị hủy bỏ.

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

// Sử dụng ctx trong goroutines
go func(ctx context.Context) {
    select {
    case <-ctx.Done():
        fmt.Println("Context cancelled:", ctx.Err())
        return
    }
}(ctx)

// Hủy context khi cần
cancel()
```

### 3. Context with Deadline

Sử dụng `context.WithDeadline(parent context, deadline time.Time)` để tạo ra một context có thời gian hết hạn cụ thể. Sau thời gian này, context sẽ bị hủy bỏ tự động.

```go
deadline := time.Now().Add(1 * time.Minute)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()

// Context sẽ tự động hủy sau 1 phút
select {
case <-time.After(2 * time.Minute):
    fmt.Println("Task completed")
case <-ctx.Done():
    fmt.Println("Deadline exceeded:", ctx.Err())
}
```

### 4. Context with Timeout

Sử dụng `context.WithTimeout(parent context, timeout time.Duration)` để tạo ra một context có thời gian chờ cụ thể. Sau thời gian chờ, context sẽ tự động bị hủy.

```go
ctx, cancel := context.WithTimeout(context.Background(), 1*time.Minute)
defer cancel()

// Context sẽ tự động hủy sau 1 phút
select {
case <-ctx.Done():
    fmt.Println("Timeout:", ctx.Err())
}
```

### 5. Context with Value

Sử dụng `context.WithValue(parent context, key, val)` để thêm giá trị vào context. Giá trị này có thể được truy cập trong các goroutines khác thông qua context đó.

```go
type contextKey string

const userIDKey contextKey = "userID"

ctx := context.WithValue(context.Background(), userIDKey, "user123")
value := ctx.Value(userIDKey)

if userID, ok := value.(string); ok {
    fmt.Println("User ID:", userID)
}
```

## Ví dụ thực tế

### Ví dụ 1: Hủy Goroutine với Cancel

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    go func(ctx context.Context) {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("Goroutine hủy bỏ")
                return
            default:
                fmt.Println("Goroutine đang chạy")
                time.Sleep(1 * time.Second)
            }
        }
    }(ctx)

    time.Sleep(3 * time.Second)
    cancel() // Hủy bỏ goroutine sau 3 giây
    time.Sleep(1 * time.Second) // Chờ goroutine kết thúc
}
```

Trong ví dụ trên, một goroutine được tạo ra và sẽ in ra "Goroutine đang chạy" mỗi giây. Sau 3 giây, chúng ta gọi `cancel()` để hủy bỏ context, khiến goroutine in ra "Goroutine hủy bỏ" và kết thúc.

### Ví dụ 2: Multiple Workers với Timeout

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup

    // Tạo một context với thời gian chờ là 3 giây
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    worker := func(ctx context.Context, id int) {
        defer wg.Done()
        for {
            select {
            case <-ctx.Done():
                fmt.Printf("Worker %d stopping: %v\n", id, ctx.Err())
                return
            default:
                fmt.Printf("Worker %d working\n", id)
                time.Sleep(1 * time.Second)
            }
        }
    }

    // Tạo và chạy 3 worker goroutines
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go worker(ctx, i)
    }

    // Chờ cho tất cả worker goroutines kết thúc
    wg.Wait()
    fmt.Println("All workers stopped")
}
```

**Output:**
```
Worker 1 working
Worker 2 working
Worker 3 working
Worker 1 working
Worker 2 working
Worker 3 working
Worker 1 working
Worker 2 working
Worker 3 working
Worker 1 stopping: context deadline exceeded
Worker 2 stopping: context deadline exceeded
Worker 3 stopping: context deadline exceeded
All workers stopped
```

Trong ví dụ này:

1. Một context với thời gian chờ 3 giây được tạo ra.
2. Ba worker goroutines được khởi tạo, mỗi goroutine sẽ liên tục in ra thông báo "Worker [id] working" mỗi giây.
3. Khi thời gian chờ của context hết, context sẽ hủy và tất cả các worker sẽ dừng lại và in ra thông báo "Worker [id] stopping".
4. Chương trình chính sẽ chờ cho đến khi tất cả các worker kết thúc trước khi in ra "All workers stopped".

### Ví dụ 3: HTTP Request với Timeout

```go
package main

import (
    "context"
    "fmt"
    "io"
    "net/http"
    "time"
)

func fetchURL(ctx context.Context, url string) (string, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return "", err
    }

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return "", err
    }

    return string(body), nil
}

func main() {
    // Tạo context với timeout 2 giây
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    result, err := fetchURL(ctx, "https://example.com")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Println("Response:", result[:100], "...")
}
```

### Ví dụ 4: Database Query với Context

```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"

    _ "github.com/lib/pq"
)

func queryDatabase(ctx context.Context, db *sql.DB, query string) error {
    // Query sẽ tự động hủy nếu context timeout
    rows, err := db.QueryContext(ctx, query)
    if err != nil {
        return err
    }
    defer rows.Close()

    for rows.Next() {
        var id int
        var name string
        if err := rows.Scan(&id, &name); err != nil {
            return err
        }
        fmt.Printf("ID: %d, Name: %s\n", id, name)
    }

    return rows.Err()
}

func main() {
    db, err := sql.Open("postgres", "connection_string")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    // Timeout 5 giây cho query
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    if err := queryDatabase(ctx, db, "SELECT id, name FROM users"); err != nil {
        fmt.Println("Query failed:", err)
    }
}
```

## Best Practices

### 1. Luôn truyền Context làm tham số đầu tiên

```go
// Good
func ProcessData(ctx context.Context, data string) error {
    // ...
}

// Bad
func ProcessData(data string, ctx context.Context) error {
    // ...
}
```

### 2. Không lưu Context trong struct

```go
// Bad
type Server struct {
    ctx context.Context
}

// Good
type Server struct {
    // other fields
}

func (s *Server) Handle(ctx context.Context) {
    // use ctx as parameter
}
```

### 3. Sử dụng type-safe keys cho WithValue

```go
type contextKey string

const (
    userIDKey    contextKey = "userID"
    requestIDKey contextKey = "requestID"
)

ctx := context.WithValue(ctx, userIDKey, "123")
```

### 4. Luôn defer cancel()

```go
ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
defer cancel() // Đảm bảo resources được giải phóng
```

### 5. Check ctx.Done() trong long-running operations

```go
func LongRunningTask(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            // Do work
            time.Sleep(100 * time.Millisecond)
        }
    }
}
```

## Context Errors

Context package cung cấp 2 error types:

```go
// context.Canceled: khi cancel() được gọi
if err == context.Canceled {
    fmt.Println("Operation was cancelled")
}

// context.DeadlineExceeded: khi timeout/deadline vượt quá
if err == context.DeadlineExceeded {
    fmt.Println("Operation timed out")
}
```

## Use Cases

### 1. HTTP Server Request Handling
```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context() // Context tự động có timeout từ server
    
    // Sử dụng ctx cho database queries, API calls, etc.
    result, err := fetchData(ctx)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    w.Write(result)
}
```

### 2. Pipeline Processing
```go
func pipeline(ctx context.Context, data <-chan int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        for v := range data {
            select {
            case <-ctx.Done():
                return
            case out <- v * 2:
            }
        }
    }()
    
    return out
}
```

### 3. Fan-out Pattern
```go
func fanOut(ctx context.Context, workers int, data <-chan int) []<-chan int {
    channels := make([]<-chan int, workers)
    
    for i := 0; i < workers; i++ {
        channels[i] = worker(ctx, data)
    }
    
    return channels
}
```

## Tham khảo

- [[Pointers|🎯 Pointers]]
- [[Goroutines|🔄 Goroutines]]
- [[Channels|📡 Channels]]
- [Go Context Package Documentation](https://pkg.go.dev/context)

## Tags liên quan

#go #context #concurrency #goroutines #timeout #cancellation #best-practices
