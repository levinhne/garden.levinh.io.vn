---
title: "Go Programming"
tags:
  - go
  - golang
  - programming
  - backend
---

# Go Programming Language

Tài liệu và kiến thức về Go (Golang) - ngôn ngữ lập trình hiện đại cho backend systems.

## 📚 Chủ đề chính

### Core Concepts
- [[Pointers|🎯 Pointers]]: Con trỏ trong Go
- [[Channels|📡 Channels]]: Communication giữa goroutines
- **Goroutines & Concurrency**: Lập trình đồng thời
- **Interfaces**: Tính trừu tượng và polymorphism
- **Error Handling**: Xử lý lỗi trong Go

### Advanced Topics
- [[Context|🎯 Context Package]]: Quản lý timeout và cancellation
- **Generics**: Go 1.18+ generic programming
- **Reflection**: Runtime type inspection
- **Memory Management**: Garbage collection, escape analysis
- **Performance Optimization**: Profiling, benchmarking

### Backend Development
- **gRPC**: Remote procedure calls
- **Protocol Buffers**: Serialization
- **Database**: SQL, ORM patterns
- **HTTP Servers**: net/http, gin, echo
- **Microservices**: Design patterns

### Testing
- **Unit Testing**: testing package
- **Table-Driven Tests**: Best practices
- **Mocking**: gomock, testify
- **Benchmarking**: Performance testing

## 🎯 Tại sao chọn Go?

### Ưu điểm
- ⚡ **Performance**: Compiled language, fast execution
- 🔄 **Concurrency**: Goroutines & channels
- 🛠️ **Tooling**: Built-in formatter, linter, test runner
- 📦 **Standard Library**: Comprehensive packages
- 🚀 **Deployment**: Single binary, cross-compilation
- 📖 **Simplicity**: Easy to learn, readable code

### Use Cases
- Microservices & APIs
- Cloud-native applications
- DevOps tools (Docker, Kubernetes)
- CLI applications
- Network servers
- Distributed systems

## 📝 Best Practices

### Code Organization
```go
project/
├── cmd/           # Main applications
├── internal/      # Private code
├── pkg/           # Public libraries
├── api/           # API definitions (protobuf)
├── configs/       # Configuration files
└── scripts/       # Build scripts
```

### Naming Conventions
- Use `camelCase` for local variables
- Use `PascalCase` for exported names
- Keep names short but meaningful
- Avoid stuttering: `user.UserID` → `user.ID`

### Error Handling
```go
// Always check errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// Use custom error types when needed
type ValidationError struct {
    Field string
    Error string
}
```

## 🔗 Tài liệu tham khảo

### Official Resources
- [Go Official Documentation](https://go.dev/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Blog](https://go.dev/blog/)

### Community
- [Go Forum](https://forum.golangbridge.org/)
- [r/golang](https://reddit.com/r/golang)
- [Gophers Slack](https://gophers.slack.com/)

### Books
- "The Go Programming Language" - Donovan & Kernighan
- "Concurrency in Go" - Katherine Cox-Buday
- "Let's Go" - Alex Edwards

## 🏷️ Related Topics

- [[Languages|💻 Programming Languages]]
- [[Software Design|🧠 Software Design]]
- [[Testing Tools|🧪 Testing Tools]]

---

*Go: Simple, fast, and reliable software at scale* 🚀
