---
title: "Design Patterns"
tags:
  - Engineering
  - Software-Design
  - Design-Patterns
  - Go
  - Java
---

# Design Patterns

Design Patterns là các giải pháp tái sử dụng cho các vấn đề thường gặp trong thiết kế phần mềm. Được giới thiệu bởi Gang of Four (GoF) trong cuốn sách **"Design Patterns: Elements of Reusable Object-Oriented Software"** (1994).

## 📊 Tổng quan

Patterns được chia thành 3 nhóm chính theo mục đích sử dụng:

| Nhóm | Mục đích | Số lượng | Ví dụ phổ biến |
|------|----------|----------|----------------|
| **Creational** | Khởi tạo đối tượng | 5 | Factory, Builder, Singleton |
| **Structural** | Tổ chức cấu trúc | 7 | Adapter, Decorator, Proxy |
| **Behavioral** | Tương tác & trách nhiệm | 11 | Observer, Strategy, Command |

## 📦 Creational Patterns

> **Mục đích**: Cung cấp cơ chế khởi tạo đối tượng linh hoạt, tái sử dụng và bảo trì dễ dàng.

### Patterns có sẵn
- [[Observer-Pattern|👁️ Observer Pattern]] ✅

### Patterns cần bổ sung
- **Factory Method** - Tạo đối tượng qua interface, delegate việc khởi tạo cho subclass
- **Abstract Factory** - Tạo họ đối tượng liên quan mà không chỉ định concrete class
- **Builder** - Xây dựng đối tượng phức tạp theo từng bước (useful cho Go)
- **Singleton** - Đảm bảo chỉ 1 instance (⚠️ cẩn thận với anti-patterns & concurrency)
- **Prototype** - Clone đối tượng thay vì tạo mới

**Use cases**: Database connections, Config managers, Object pools, Complex object construction

## 🏗️ Structural Patterns

> **Mục đích**: Tổ chức class/object để tạo cấu trúc lớn hơn, linh hoạt và hiệu quả.

### Patterns cần bổ sung
- **Adapter** - Chuyển đổi interface (wrapper pattern, rất phổ biến trong Go)
- **Decorator** - Thêm behavior động mà không ảnh hưởng class gốc
- **Facade** - Interface đơn giản cho hệ thống phức tạp (API Gateway pattern)
- **Proxy** - Đại diện để kiểm soát truy cập (caching, lazy loading, security)
- **Composite** - Cấu trúc cây cho part-whole hierarchy
- **Bridge** - Tách abstraction và implementation
- **Flyweight** - Chia sẻ data để tiết kiệm memory (string interning)

**Use cases**: API design, Middleware, Database ORM, UI components, Caching layers

## 🎭 Behavioral Patterns

> **Mục đích**: Quản lý algorithms, relationships và responsibilities giữa objects.

### Patterns có sẵn
- [[Observer-Pattern|👁️ Observer Pattern]] ✅ - Event notification system

### Patterns cần bổ sung
- **Strategy** - Encapsulate algorithms, dễ swap (dependency injection)
- **Command** - Encapsulate request as object (undo/redo, queue, logging)
- **Chain of Responsibility** - Pass request qua chain of handlers (middleware)
- **State** - Thay đổi behavior theo state (state machine)
- **Template Method** - Định nghĩa skeleton, subclass override steps
- **Iterator** - Traverse collection mà không expose internal structure
- **Mediator** - Central communication hub (reduce coupling)
- **Memento** - Capture & restore state (snapshot, undo)
- **Visitor** - Add operations mà không modify class (double dispatch)
- **Interpreter** - Grammar & interpreter cho DSL

**Use cases**: HTTP middleware, State machines, Event buses, Undo/Redo systems, Workflow engines

## 🎯 Pattern Selection Guide

### Khi nào dùng pattern nào?

```
Cần tạo object linh hoạt? → Creational Patterns
├─ Object phức tạp? → Builder
├─ Nhiều variants? → Factory/Abstract Factory
└─ Cần clone? → Prototype

Cần tổ chức structure? → Structural Patterns
├─ Incompatible interfaces? → Adapter
├─ Add behavior động? → Decorator
├─ Simplify complex system? → Facade
└─ Control access? → Proxy

Cần quản lý behavior? → Behavioral Patterns
├─ Event notification? → Observer
├─ Swap algorithm? → Strategy
├─ Chain processing? → Chain of Responsibility
└─ State-dependent behavior? → State
```

## 🚀 Ưu tiên triển khai cho Go Backend

Theo thứ tự quan trọng cho Senior Backend Engineer:

1. **Strategy** ⭐⭐⭐ - Dependency injection, testability
2. **Builder** ⭐⭐⭐ - Functional options pattern (idiomatic Go)
3. **Adapter** ⭐⭐⭐ - Interface compatibility (database, external APIs)
4. **Decorator/Middleware** ⭐⭐⭐ - HTTP/gRPC interceptors
5. **Observer** ⭐⭐ - Event systems (đã có ✅)
6. **Factory** ⭐⭐ - Object creation logic
7. **Singleton** ⭐ - Config, logger (nhưng cẩn thận!)

## 🎓 Learning Path

### Bước 1: Foundation (SOLID first!)
- [[SOLID|⚡ SOLID Principles]] - Must understand trước khi học patterns

### Bước 2: Essential Patterns (Go-focused)
1. Strategy → Builder → Adapter
2. Observer (✅) → Decorator → Factory

### Bước 3: Advanced Patterns
3. Chain of Responsibility → State → Command

### Bước 4: Specialized
4. Remaining patterns theo nhu cầu dự án

## 📖 Tài nguyên học tập

### Sách
- **Design Patterns: Elements of Reusable Object-Oriented Software** (Gang of Four, 1994)
- **Head First Design Patterns** (Freeman & Freeman) - Dễ hiểu, nhiều hình ảnh
- **Go Design Patterns** (Mario Castro Contreras) - Specific cho Go

### Online Resources
- [Refactoring.Guru](https://refactoring.guru/design-patterns) - Visual diagrams, code examples
- [SourceMaking](https://sourcemaking.com/design_patterns) - UML diagrams
- [Go Patterns](https://github.com/tmrts/go-patterns) - Go implementations

### Practice
- Refactor existing code với patterns
- Code reviews - nhận diện patterns trong production code
- Side projects - thử nghiệm các patterns mới

## ⚠️ Anti-Patterns cần tránh

- **Overengineering**: Dùng pattern khi không cần thiết
- **Golden Hammer**: "Có búa thì đâu cũng thấy đinh" - dùng pattern quen thuộc cho mọi vấn đề
- **Singleton Abuse**: Tạo global state, khó test
- **Abstract Factory Overuse**: Quá nhiều abstraction layers
- **Observer Memory Leaks**: Không unsubscribe observers

## 🔗 Liên quan

- [[SOLID|⚡ SOLID Principles]] - Foundation principles
- [[../index|🏗️ Software Design]] - Parent section
- [[../../Foundation/Languages/Go/index|🔷 Go Programming]] - Language specific implementations
- [[../RabbitMQ|📬 RabbitMQ]] - Real-world Observer pattern usage
