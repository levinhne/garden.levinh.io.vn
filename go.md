Dưới đây là toàn bộ nội dung file Markdown, bạn có thể copy trực tiếp:

````md
# Tổng hợp cấu trúc dự án Go cho backend service

## 1. Cấu trúc tổng thể

```text
.
├── cmd/
│   └── api/
│       └── main.go           # Wires everything: DB, clients, DI, server
├── internal/
│   ├── <module>/             # Ví dụ: order, auth, user...
│   │   ├── delivery/         # Transport layer
│   │   │   ├── grpc/         # gRPC handlers
│   │   │   ├── http/         # HTTP/REST handlers
│   │   │   └── event/        # Message consumer (RabbitMQ/Kafka)
│   │   ├── app/              # Application layer
│   │   │   ├── command/      # Write use cases
│   │   │   ├── query/        # Read use cases
│   │   │   ├── dto/          # DTO dùng cho app layer (nếu cần)
│   │   │   └── ports/        # Interface cho external dependency
│   │   ├── domain/           # Core business
│   │   │   ├── entity.go
│   │   │   ├── repository.go
│   │   │   ├── errors.go
│   │   │   └── value_object.go
│   │   └── infrastructure/   # Technical adapters
│   │       ├── repository/
│   │       └── clients/
│   └── platform/             # Shared technical foundation
│       ├── logger/
│       ├── datastore/
│       ├── middleware/
│       ├── grpcx/
│       └── authctx/
├── proto/                    # Proto source
│   ├── user/v1/
│   ├── order/v1/
│   └── common/v1/
├── gen/
│   ├── go/                   # Generated *.pb.go
│   └── openapi/              # Generated openapi yaml
├── third_party/              # googleapis, plugin proto...
├── migrations/
└── go.mod
````

---

## 2. Ý nghĩa từng phần

### `cmd/api/main.go`

Chỉ nên làm:

* load config
* init logger
* init db / redis / broker / external clients
* wire dependencies
* start server

Không nên nhét business logic vào đây.

---

### `internal/<module>/delivery`

Nhận request từ bên ngoài.

#### `grpc/`

* parse protobuf request
* validate input cơ bản
* gọi app layer
* map error sang gRPC status

#### `http/`

* parse body/query/path
* validate request
* gọi app layer
* trả JSON/HTTP status

#### `event/`

* consume message từ RabbitMQ/Kafka
* parse event
* idempotency check nếu cần
* gọi use case tương ứng

`delivery` nên mỏng, không chứa business logic.

---

### `internal/<module>/app`

Điều phối use case.

#### `command/`

Chứa logic write:

* create
* update
* delete
* activate/deactivate

#### `query/`

Chứa logic read:

* get detail
* list
* search
* filter

#### `dto/`

Chứa DTO dùng trong app layer nếu cần tách khỏi transport model.

#### `ports/`

Chứa interface cho dependency bên ngoài:

* payment client
* user service client
* email sender
* tx manager

Đây là nơi orchestration diễn ra:

* gọi repository
* gọi external client
* quản lý transaction
* publish event

---

### `internal/<module>/domain`

Chứa nghiệp vụ cốt lõi.

#### Nên để:

* entity
* value object
* domain rule
* invariant
* repository interface
* domain errors

Ví dụ:

```go
type Order struct {
    ID     string
    Status OrderStatus
}

func (o *Order) Cancel() error {
    if o.Status == OrderStatusShipped {
        return ErrCannotCancelShippedOrder
    }
    o.Status = OrderStatusCancelled
    return nil
}
```

Không nên để domain chỉ là struct rỗng.

---

### `internal/<module>/infrastructure`

Triển khai kỹ thuật cụ thể.

#### `repository/`

* MySQL/PostgreSQL/Mongo/Redis implementation

#### `clients/`

* gRPC client
* REST client
* SDK wrapper

Đây là adapter layer, không nên chứa business workflow.

---

### `internal/platform`

Chứa phần kỹ thuật dùng chung toàn app.

#### `logger/`

Nên để:

* logger setup/factory
* wrapper cho slog/zap/zerolog
* helper thêm field chung
* context helper cho logger

Không nên để:

* log message business cụ thể
* audit log nghiệp vụ từng module

#### `datastore/`

Nên để:

* khởi tạo DB pool
* redis client
* transaction helper
* shared DB config

#### `middleware/`

Nên để:

* auth middleware
* request logging
* recovery
* rate limit
* tracing/metrics hook

#### `grpcx/`

Helper liên quan gRPC:

* metadata helper
* interceptor helper
* error mapping gRPC status
* client/server setup helper

#### `authctx/`

Dùng để set/get auth info trong context theo kiểu transport-agnostic:

* `WithUserID`
* `UserIDFromContext`

Tốt hơn `ctxutil` vì rõ nghĩa hơn.

---

## 3. Có cần `pkg/` không?

Không bắt buộc.

Với backend service thông thường:

* `internal/` là đủ
* không cần `pkg/` nếu bạn không định export reusable package cho project khác

### Khi nào nên có `pkg/`

Chỉ khi bạn có package thật sự muốn reuse bên ngoài repo, ví dụ:

* `pkg/grpcx`
* `pkg/httpx`
* `pkg/retry`

Nếu không, rất dễ biến `pkg/` thành chỗ nhét helper linh tinh.

---

## 4. `grpcx` thường để làm gì?

`grpcx` thường là package helper cho gRPC, ví dụ:

* error mapping sang gRPC status
* metadata helper
* interceptor helper
* common dial/server options
* request/response helper đặc thù gRPC

Ví dụ:

```go
func ToStatus(err error) error
func UserIDFromMetadata(ctx context.Context) (string, bool)
```

Không nên để business logic hoặc handler module cụ thể vào đây.

---

## 5. `ExtractUserIDFromContext` nên để ở đâu?

Tùy hàm đó phụ thuộc cái gì.

### Nếu đọc từ gRPC metadata

Nên để trong `grpcx`.

Ví dụ:

```go
func UserIDFromMetadata(ctx context.Context) (string, bool)
```

### Nếu đọc từ `context.Context` đã được set sẵn

Nên để trong `authctx`.

Ví dụ:

```go
func WithUserID(ctx context.Context, userID string) context.Context
func UserIDFromContext(ctx context.Context) (string, bool)
```

### Cách tốt nhất

* `grpcx` đọc từ metadata
* `authctx` lưu/đọc user id dùng chung cho toàn app

Flow:

* interceptor parse metadata
* set user id vào `authctx`
* handler/app chỉ đọc từ `authctx`

Như vậy app layer không bị dính transport.

---

## 6. Proto nên để ở đâu?

Nên tách:

* proto source
* generated code

### Gợi ý

```text
proto/
  user/v1/
  order/v1/
  common/v1/

gen/
  go/
```

### Vì sao không nên để proto trong `internal/`

Vì proto là API contract, không phải implementation nội bộ.

---

## 7. Trong `user/v1/` nên đặt tên file thế nào?

Cách gọn, rõ ràng:

```text
proto/user/v1/
├── service.proto
└── user.proto
```

### `service.proto`

Chứa:

* `service UserService`
* request/response message cho RPC

Ví dụ:

```proto
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
}
```

### `user.proto`

Chứa:

* `User`
* `UserProfile`
* enum
* type dùng chung

Ví dụ:

```proto
message User {
  string id = 1;
  string email = 2;
  string full_name = 3;
}
```

### Có nên dùng `store.proto` không?

Chỉ nên dùng nếu bạn thực sự muốn biểu diễn storage/read-model riêng.

Nếu message đó là API model dùng chung, `user.proto` rõ hơn `store.proto`.

---

## 8. Generated OpenAPI YAML nên để ở đâu?

Nên để ở:

```text
gen/openapi/
```

Ví dụ:

```text
gen/openapi/
├── user/v1/user.openapi.yaml
├── order/v1/order.openapi.yaml
└── api.openapi.yaml
```

### Vì sao?

Vì `openapi.yaml` là generated artifact, không phải source gốc.

### Không nên để trong:

* `proto/`
* `internal/`

Nếu cần serve docs ra ngoài, có thể copy sang:

* `docs/openapi/`
* `public/openapi/`

Nhưng bản generate gốc vẫn nên nằm ở `gen/openapi/`.

---

## 9. Một số nguyên tắc quan trọng

### Dependency direction

Nên đi theo chiều:

```text
delivery -> app -> domain
infrastructure -> app/domain
```

Không đi ngược lại.

---

### Handler phải mỏng

Handler chỉ nên:

* parse request
* validate cơ bản
* gọi app
* map response/error

Không nên chứa business logic.

---

### App layer để orchestration

App layer nên:

* điều phối flow
* gọi repository/client
* quản lý transaction
* publish event

Không nên ôm hết domain rule.

---

### Domain giữ invariant

Entity/value object nên có hành vi, không chỉ dữ liệu.

---

### Infrastructure chỉ implement

Repository/client implementation không nên tự quyết định business flow.

---

## 10. Kết luận

Structure này rất hợp với:

* modular monolith
* clean architecture
* DDD/CQRS theo kiểu thực dụng
* backend service có HTTP + gRPC + event-driven flow

Các điểm nên chốt:

* không bắt buộc phải có `pkg/`
* dùng `internal/platform` cho shared technical code
* `grpcx` cho gRPC helper
* `authctx` cho auth info trong context
* `proto/` cho source proto
* `gen/go/` cho `pb.go`
* `gen/openapi/` cho OpenAPI generated files
* trong `user/v1/` nên ưu tiên `service.proto` + `user.proto`

## 11. Convention gợi ý cuối cùng

```text
.
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── platform/
│   │   ├── logger/
│   │   ├── datastore/
│   │   ├── middleware/
│   │   ├── grpcx/
│   │   └── authctx/
│   ├── user/
│   │   ├── delivery/
│   │   ├── app/
│   │   │   ├── command/
│   │   │   ├── query/
│   │   │   ├── dto/
│   │   │   └── ports/
│   │   ├── domain/
│   │   └── infrastructure/
│   └── order/
├── proto/
│   ├── user/v1/
│   │   ├── service.proto
│   │   └── user.proto
│   ├── order/v1/
│   └── common/v1/
├── gen/
│   ├── go/
│   └── openapi/
├── third_party/
├── migrations/
└── go.mod
```

```

Nếu muốn, tôi có thể chuyển tiếp phần này thành một bản “guideline nội bộ cho team” viết ngắn gọn hơn, sạch hơn, kiểu đem bỏ thẳng vào repo dưới tên `ARCHITECTURE.md`.
```
