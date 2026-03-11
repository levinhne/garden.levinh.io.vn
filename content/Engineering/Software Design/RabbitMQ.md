---
title: "RabbitMQ"
tags:
  - rabbitmq
  - message-broker
  - messaging
  - amqp
  - microservices
  - backend
---

RabbitMQ là một message broker mã nguồn mở, được viết bằng ngôn ngữ Erlang. Nó thực hiện việc quản lý và phân phối các thông điệp giữa các ứng dụng khác nhau, giúp chúng giao tiếp một cách hiệu quả và đáng tin cậy. RabbitMQ hỗ trợ nhiều giao thức nhưng phổ biến nhất là AMQP (Advanced Message Queuing Protocol).

## Các đặc điểm chính

### 1. Khả năng mở rộng và hiệu suất cao

RabbitMQ có thể xử lý một lượng lớn thông điệp và có thể mở rộng dễ dàng bằng cách thêm các nút vào cụm (cluster).

```bash
# Thêm node vào cluster
rabbitmqctl join_cluster rabbit@node1
```

### 2. Hỗ trợ nhiều giao thức

RabbitMQ hỗ trợ các giao thức phổ biến sau:

#### AMQP (Advanced Message Queuing Protocol)
AMQP cung cấp các tính năng như hàng đợi, trao đổi tin nhắn và bảo mật. Nó đảm bảo rằng các thông điệp được phân phối một cách đáng tin cậy và có thể được định tuyến linh hoạt.

**Đặc điểm:**
- Binary protocol: Hiệu quả hơn text-based protocols
- Reliable delivery: Đảm bảo message được deliver
- Flexible routing: Multiple exchange types
- Security: Authentication & encryption

#### MQTT (Message Queuing Telemetry Transport)
MQTT là một giao thức nhẹ nhàng cho các thiết bị có tài nguyên hạn chế và mạng không ổn định. Thường được sử dụng trong các ứng dụng IoT, MQTT giúp truyền thông điệp hiệu quả giữa các thiết bị kết nối.

**Use cases:**
- IoT devices (sensors, smart home)
- Mobile applications
- Low bandwidth networks

#### HTTP (Hypertext Transfer Protocol)
Via Management API và HTTP plugins.

#### WebSockets
Real-time communication cho web applications.

### 3. Khả năng chịu lỗi

RabbitMQ hỗ trợ tính năng clustering và monitoring, giúp đảm bảo rằng các thông điệp không bị mất khi có sự cố xảy ra.

**Features:**
- Message persistence: Lưu messages vào disk
- Queue mirroring: Replicate queues across nodes
- Automatic failover: Switch to backup node

### 4. Bảo mật

Hỗ trợ SSL/TLS, xác thực người dùng và quyền hạn, giúp đảm bảo an toàn thông tin.

```bash
# Enable SSL
ssl_options.cacertfile = /path/to/ca_certificate.pem
ssl_options.certfile   = /path/to/server_certificate.pem
ssl_options.keyfile    = /path/to/server_key.pem
```

### 5. Hệ sinh thái phong phú

Có nhiều plugins và công cụ hỗ trợ giúp quản lý và giám sát dễ dàng hơn.

**Popular plugins:**
- Management Plugin: Web UI
- Shovel Plugin: Message forwarding
- Federation Plugin: Link exchanges/queues
- MQTT Plugin: MQTT protocol support

## Cách thức hoạt động

### 1. Producer
Là các ứng dụng hoặc dịch vụ gửi thông điệp tới RabbitMQ.

```go
// Go example
ch.Publish(
    "exchange_name", // exchange
    "routing_key",   // routing key
    false,           // mandatory
    false,           // immediate
    amqp.Publishing{
        ContentType: "text/plain",
        Body:        []byte("Hello World!"),
    })
```

### 2. Queue
Là nơi lưu trữ các thông điệp tạm thời cho đến khi chúng được xử lý bởi consumer.

**Đặc điểm:**
- FIFO (First In First Out) by default
- Durable: Survive broker restart
- Exclusive: Used by only one connection
- Auto-delete: Deleted when no consumers

### 3. Consumer
Là các ứng dụng hoặc dịch vụ nhận và xử lý thông điệp từ RabbitMQ.

```go
// Go example
msgs, err := ch.Consume(
    "queue_name", // queue
    "",           // consumer
    true,         // auto-ack
    false,        // exclusive
    false,        // no-local
    false,        // no-wait
    nil,          // args
)

for msg := range msgs {
    log.Printf("Received: %s", msg.Body)
}
```

### 4. Exchange
Là nơi nhận thông điệp từ Producer và định tuyến chúng tới các hàng đợi (queue) dựa trên các quy tắc định tuyến.

**4 loại Exchange chính:**
- Direct
- Topic
- Fanout
- Headers

### 5. Virtual Host (vhost)
Cung cấp những cách riêng biệt để các ứng dụng dùng chung một RabbitMQ instance. Những user khác nhau có thể có quyền khác nhau đối với vhost khác nhau. Queue và Exchange có thể được tạo, vì vậy chúng chỉ tồn tại trong một vhost.

```bash
# Tạo vhost
rabbitmqctl add_vhost my_vhost

# Gán quyền
rabbitmqctl set_permissions -p my_vhost user_name ".*" ".*" ".*"
```

## Architecture Diagrams

### Basic Architecture

```
┌──────────┐
│ Producer │
└────┬─────┘
     │
     ▼
┌─────────┐     ┌───────┐
│Exchange │────▶│Queue 1│
└─────────┘     └───┬───┘
     │              │
     │          ┌───▼────────┐
     │          │ Consumer 1 │
     │          └────────────┘
     │
     │          ┌───────┐
     └─────────▶│Queue 2│
                └───┬───┘
                    │
                ┌───▼────────┐
                │ Consumer 2 │
                └────────────┘
```

### Direct Exchange

Routing dựa trên **exact match** của routing key.

```
                                    routing_key="error"
┌──────────┐   routing_key="error"  ┌──────────────┐
│Producer 1│──────────────────────▶ │              │
└──────────┘                        │    Direct    │    ┌────────┐
                                    │   Exchange   │───▶│Queue 1 │──▶ Consumer (errors only)
┌──────────┐   routing_key="info"   │  (logs_ex)   │    └────────┘
│Producer 2│──────────────────────▶ │              │    binding: routing_key="error"
└──────────┘                        │              │
                                    └──────┬───────┘
┌──────────┐   routing_key="warn"          │           ┌────────┐
│Producer 3│──────────────────────▶        ├──────────▶│Queue 2 │──▶ Consumer (info only)
└──────────┘                               │           └────────┘
                                           │           binding: routing_key="info"
                                           │
                                           │           ┌────────┐
                                           └──────────▶│Queue 3 │──▶ Consumer (warnings only)
                                                       └────────┘
                                                       binding: routing_key="warn"

Kết quả:
- Message với routing_key="error" → chỉ đến Queue 1
- Message với routing_key="info"  → chỉ đến Queue 2
- Message với routing_key="warn"  → chỉ đến Queue 3
```

### Topic Exchange

Routing dựa trên **pattern matching** với wildcards (`*` và `#`).

```
                                    routing_key="user.*.created"
┌──────────┐  routing_key=           ┌──────────────┐
│Producer 1│  "user.profile.created" │              │
└──────────┘──────────────────────▶  │    Topic     │    ┌────────┐
                                     │   Exchange   │───▶│Queue 1 │──▶ Consumer
┌──────────┐  routing_key=           │  (topic_ex)  │    └────────┘
│Producer 2│  "user.account.created" │              │    binding: "user.*.created"
└──────────┘──────────────────────▶  │              │    (khớp: user.[bất kỳ].created)
                                     │              │
┌──────────┐  routing_key=           └──────┬───────┘
│Producer 3│  "order.item.shipped"          │           ┌────────┐
└──────────┘──────────────────────▶         ├──────────▶│Queue 2 │──▶ Consumer
                                            │           └────────┘
┌──────────┐  routing_key=                  │           binding: "order.#"
│Producer 4│  "order.payment.completed"     │           (khớp: order.[0 hoặc nhiều từ])
└──────────┘──────────────────────▶         │
                                            │           ┌────────┐
                                            └──────────▶│Queue 3 │──▶ Consumer
                                                        └────────┘
                                                        binding: "#.created"
                                                        (khớp: [bất kỳ].created)

Wildcards:
- * (star)  : Khớp chính xác 1 từ
- # (hash)  : Khớp 0 hoặc nhiều từ

Kết quả:
- "user.profile.created"      → Queue 1 (user.*.created) và Queue 3 (#.created)
- "user.account.created"      → Queue 1 (user.*.created) và Queue 3 (#.created)
- "order.item.shipped"        → Queue 2 (order.#)
- "order.payment.completed"   → Queue 2 (order.#)
```

### Fanout Exchange

Broadcast message đến **tất cả queues** được bind (bỏ qua routing key).

```
                                     ┌──────────────┐
┌──────────┐  routing_key=(ignored)  │              │
│Producer 1│──────────────────────▶  │   Fanout     │    ┌────────┐
└──────────┘                         │   Exchange   │───▶│Queue 1 │──▶ Consumer 1
                                     │  (fanout_ex) │    └────────┘
┌──────────┐  routing_key=(ignored)  │              │
│Producer 2│──────────────────────▶  │              │    ┌────────┐
└──────────┘                         │              │───▶│Queue 2 │──▶ Consumer 2
                                     │              │    └────────┘
                                     │              │
                                     │              │    ┌────────┐
                                     │              │───▶│Queue 3 │──▶ Consumer 3
                                     └──────────────┘    └────────┘

Kết quả:
- Mọi message đều được gửi đến TẤT CẢ queues (Queue 1, 2, 3)
- Routing key bị bỏ qua hoàn toàn
- Tất cả consumers đều nhận được cùng một message

Use case: Broadcasting, notifications, cache invalidation
```

### Headers Exchange

Routing dựa trên **message headers** thay vì routing key.

```
                              headers: {format:"pdf", type:"report"}
┌──────────┐                         ┌──────────────┐
│Producer 1│─────────────────────▶   │   Headers    │    ┌────────┐
└──────────┘                         │   Exchange   │───▶│Queue 1 │──▶ Consumer
                              headers: {format:"json"}    └────────┘
┌──────────┐                         │ (header_ex)  │    binding: x-match=all
│Producer 2│─────────────────────▶   │              │    {format:"pdf", type:"report"}
└──────────┘                         │              │
                                     └──────┬───────┘
                              headers:      │           ┌────────┐
┌──────────┐                  {type:"image"}├──────────▶│Queue 2 │──▶ Consumer
│Producer 3│─────────────────────▶          │           └────────┘
└──────────┘                                │           binding: x-match=any
                                            │           {format:"jpeg", type:"image"}

Matching rules:
- x-match=all : TẤT CẢ headers phải khớp
- x-match=any : ÍT NHẤT 1 header phải khớp

Kết quả:
- {format:"pdf", type:"report"}  → Queue 1 (all match)
- {format:"json"}                → Không match queue nào
- {type:"image"}                 → Queue 2 (any match)
```

## Lợi ích khi dùng RabbitMQ

### 1. Giảm tải hệ thống
Các ứng dụng có thể giao tiếp một cách không đồng bộ, giảm thiểu thời gian chờ và tăng hiệu suất.

**Ví dụ:**
- User upload video → Queue → Background processing
- API request → Queue → Batch processing

### 2. Tách biệt thành phần
Giúp tách biệt các thành phần trong hệ thống, làm cho việc phát triển, bảo trì và mở rộng dễ dàng hơn.

**Benefits:**
- Loose coupling
- Independent scaling
- Easy to maintain
- Service isolation

### 3. Đảm bảo truyền tin đáng tin cậy
Với khả năng xử lý lỗi và lưu trữ tin nhắn tạm thời, RabbitMQ đảm bảo rằng các thông điệp không bị mất trong quá trình truyền.

**Reliability features:**
- Publisher confirms
- Consumer acknowledgments
- Message persistence
- Dead letter exchanges

## Các loại Exchange

### 1. Direct Exchange

**Mô tả:** Định tuyến thông điệp đến các queue có routing key (khóa định tuyến) chính xác khớp với routing key của thông điệp.

**Sử dụng khi nào:** Khi cần định tuyến các thông điệp đến một hàng đợi cụ thể dựa trên một khóa định tuyến.

```bash
# Tạo một direct exchange
rabbitmqadmin declare exchange name=direct_logs type=direct

# Tạo hai hàng đợi
rabbitmqadmin declare queue name=queue1
rabbitmqadmin declare queue name=queue2

# Liên kết hàng đợi với exchange bằng các khóa routing key cụ thể
rabbitmqadmin declare binding source=direct_logs destination=queue1 routing_key="info"
rabbitmqadmin declare binding source=direct_logs destination=queue2 routing_key="error"

# Gửi tin nhắn với khóa routing key là `info`
rabbitmqadmin publish exchange=direct_logs routing_key=info payload="This is an info message"

# Gửi tin nhắn với khóa routing key là `error`
rabbitmqadmin publish exchange=direct_logs routing_key=error payload="This is an error message"
```

**Use case:** Log routing dựa trên severity level.

### 2. Topic Exchange

**Mô tả:** Định tuyến các thông điệp đến các queue dựa trên một mẫu routing key. Các khóa định tuyến có thể chứa các ký tự đại diện như `*` (đại diện cho một từ) và `#` (đại diện không, một hoặc nhiều từ).

**Sử dụng khi nào:** Khi bạn cần định tuyến các thông điệp dựa trên mẫu phức tạp của khóa định tuyến.

```bash
# Tạo một topic exchange
rabbitmqadmin declare exchange name=topic_logs type=topic

# Tạo hai hàng đợi
rabbitmqadmin declare queue name=queue1
rabbitmqadmin declare queue name=queue2

# Liên kết hàng đợi với exchange bằng các mẫu routing key
# log.*: khớp với routing key có pattern `log` và theo sau là chính xác một từ
# Hợp lệ: log.info, log.error, log.debug
# Không hợp lệ: log, log.log.info...
rabbitmqadmin declare binding source=topic_logs destination=queue1 routing_key="log.*"

# order.#: khớp với tất cả các key bắt đầu bằng order.
rabbitmqadmin declare binding source=topic_logs destination=queue2 routing_key="order.#"

# Gửi tin nhắn với khóa routing key là `log.info`
rabbitmqadmin publish exchange=topic_logs routing_key=log.info payload="This is an info log message"

# Gửi tin nhắn với khóa routing key là `order.created`
rabbitmqadmin publish exchange=topic_logs routing_key=order.created payload="Order has been created"
```

**Wildcard patterns:**
- `*` (star): Exactly one word
- `#` (hash): Zero or more words

**Examples:**
- `*.orange.*` → matches `quick.orange.rabbit`
- `lazy.#` → matches `lazy.orange.elephant` and `lazy.pink.rabbit.large`

### 3. Fanout Exchange

**Mô tả:** Định tuyến các thông điệp đến tất cả các queue được liên kết với exchange này mà không cần quan tâm đến routing key.

**Sử dụng khi nào:** Khi cần gửi cùng một thông điệp đến nhiều hàng đợi một lúc.

```bash
# Tạo một fanout exchange
rabbitmqadmin declare exchange name=fanout_logs type=fanout

# Tạo hai hàng đợi
rabbitmqadmin declare queue name=queue1
rabbitmqadmin declare queue name=queue2

# Liên kết các hàng đợi với exchange (không cần khóa routing key)
rabbitmqadmin declare binding source=fanout_logs destination=queue1
rabbitmqadmin declare binding source=fanout_logs destination=queue2

# Gửi tin nhắn (không cần khóa routing key)
rabbitmqadmin publish exchange=fanout_logs payload="This is a broadcast message"
```

**Use cases:**
- Broadcasting notifications
- Cache invalidation
- System-wide announcements

### 4. Headers Exchange

**Mô tả:** Định tuyến các thông điệp dựa trên các thuộc tính header của thông điệp thay vì dựa vào routing key.

**Sử dụng khi nào:** Khi cần định tuyến các thông điệp dựa trên các thuộc tính phức tạp hoặc nhiều thuộc tính khác nhau.

```bash
# Tạo một header exchange
rabbitmqadmin declare exchange name=header_logs type=headers

# Tạo hai hàng đợi
rabbitmqadmin declare queue name=queue1
rabbitmqadmin declare queue name=queue2

# Liên kết hàng đợi với exchange bằng các điều kiện header
# Có hai loại matching rules, được xác định bởi thuộc tính x-match
# x-match=all: Tất cả các cặp key-value trong header phải khớp
# x-match=any: Chỉ cần một trong các cặp key-value trong header khớp
rabbitmqadmin declare binding source=header_logs destination=queue1 \
  arguments='{"x-match":"all", "format":"pdf", "type":"report"}'

rabbitmqadmin declare binding source=header_logs destination=queue2 \
  arguments='{"x-match":"any", "format":"jpeg", "type":"image"}'

# Gửi tin nhắn với header khớp với queue1
rabbitmqadmin publish exchange=header_logs \
  properties='{"headers":{"format":"pdf", "type":"report"}}' \
  payload="This is a PDF report"

# Gửi tin nhắn với header khớp với queue2
rabbitmqadmin publish exchange=header_logs \
  properties='{"headers":{"format":"jpeg"}}' \
  payload="This is a JPEG image"
```

**Matching rules:**
- `x-match=all`: ALL headers must match
- `x-match=any`: AT LEAST ONE header must match

## Code Examples

### Go Producer Example

```go
package main

import (
    "log"
    "github.com/streadway/amqp"
)

func main() {
    // Connect to RabbitMQ
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer conn.Close()

    // Create channel
    ch, err := conn.Channel()
    if err != nil {
        log.Fatalf("Failed to open channel: %v", err)
    }
    defer ch.Close()

    // Declare exchange
    err = ch.ExchangeDeclare(
        "logs",   // name
        "fanout", // type
        true,     // durable
        false,    // auto-deleted
        false,    // internal
        false,    // no-wait
        nil,      // arguments
    )
    if err != nil {
        log.Fatalf("Failed to declare exchange: %v", err)
    }

    // Publish message
    body := "Hello RabbitMQ!"
    err = ch.Publish(
        "logs", // exchange
        "",     // routing key
        false,  // mandatory
        false,  // immediate
        amqp.Publishing{
            ContentType: "text/plain",
            Body:        []byte(body),
        })
    
    if err != nil {
        log.Fatalf("Failed to publish: %v", err)
    }
    
    log.Printf("Sent: %s", body)
}
```

### Go Consumer Example

```go
package main

import (
    "log"
    "github.com/streadway/amqp"
)

func main() {
    // Connect to RabbitMQ
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer conn.Close()

    // Create channel
    ch, err := conn.Channel()
    if err != nil {
        log.Fatalf("Failed to open channel: %v", err)
    }
    defer ch.Close()

    // Declare exchange
    err = ch.ExchangeDeclare(
        "logs",   // name
        "fanout", // type
        true,     // durable
        false,    // auto-deleted
        false,    // internal
        false,    // no-wait
        nil,      // arguments
    )
    if err != nil {
        log.Fatalf("Failed to declare exchange: %v", err)
    }

    // Declare queue
    q, err := ch.QueueDeclare(
        "",    // name (empty = auto-generated)
        false, // durable
        false, // delete when unused
        true,  // exclusive
        false, // no-wait
        nil,   // arguments
    )
    if err != nil {
        log.Fatalf("Failed to declare queue: %v", err)
    }

    // Bind queue to exchange
    err = ch.QueueBind(
        q.Name, // queue name
        "",     // routing key
        "logs", // exchange
        false,
        nil,
    )
    if err != nil {
        log.Fatalf("Failed to bind queue: %v", err)
    }

    // Consume messages
    msgs, err := ch.Consume(
        q.Name, // queue
        "",     // consumer
        true,   // auto-ack
        false,  // exclusive
        false,  // no-local
        false,  // no-wait
        nil,    // args
    )
    if err != nil {
        log.Fatalf("Failed to register consumer: %v", err)
    }

    forever := make(chan bool)

    go func() {
        for msg := range msgs {
            log.Printf("Received: %s", msg.Body)
        }
    }()

    log.Printf("Waiting for messages...")
    <-forever
}
```

## Best Practices

### 1. Message Persistence

```go
// Declare durable queue
ch.QueueDeclare(
    "task_queue", // name
    true,         // durable
    false,        // delete when unused
    false,        // exclusive
    false,        // no-wait
    nil,          // arguments
)

// Mark messages as persistent
ch.Publish(
    "",           // exchange
    "task_queue", // routing key
    false,        // mandatory
    false,        // immediate
    amqp.Publishing{
        DeliveryMode: amqp.Persistent, // Mark as persistent
        ContentType:  "text/plain",
        Body:         []byte(body),
    })
```

### 2. Fair Dispatch (Prefetch)

```go
// Set prefetch count
err = ch.Qos(
    1,     // prefetch count
    0,     // prefetch size
    false, // global
)
```

### 3. Manual Acknowledgment

```go
// Consumer with manual ack
msgs, err := ch.Consume(
    q.Name, // queue
    "",     // consumer
    false,  // auto-ack = false
    false,  // exclusive
    false,  // no-local
    false,  // no-wait
    nil,    // args
)

for msg := range msgs {
    log.Printf("Received: %s", msg.Body)
    
    // Process message
    processMessage(msg.Body)
    
    // Manually acknowledge
    msg.Ack(false)
}
```

### 4. Connection Management

```go
// Use connection pooling
// Reuse connections
// Handle reconnection

func createConnection() (*amqp.Connection, error) {
    for i := 0; i < 5; i++ {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        if err == nil {
            return conn, nil
        }
        time.Sleep(time.Second * time.Duration(i+1))
    }
    return nil, errors.New("failed to connect after retries")
}
```

### 5. Dead Letter Exchange (DLX)

```go
// Queue with DLX
args := amqp.Table{
    "x-dead-letter-exchange":    "dlx_exchange",
    "x-dead-letter-routing-key": "dlx",
}

ch.QueueDeclare(
    "main_queue",
    true,  // durable
    false, // delete when unused
    false, // exclusive
    false, // no-wait
    args,  // arguments
)
```

## Common Use Cases

### 1. Task Queue (Work Queue)
Distribute time-consuming tasks among multiple workers.

### 2. Pub/Sub Pattern
Deliver messages to multiple consumers.

### 3. RPC (Remote Procedure Call)
Request-reply pattern over messaging.

### 4. Event-Driven Architecture
Decouple services via events.

### 5. Log Aggregation
Collect logs from multiple services.

## Monitoring & Management

### Management UI
Access at: `http://localhost:15672`
- Default credentials: `guest/guest`

### CLI Commands

```bash
# List queues
rabbitmqctl list_queues

# List exchanges
rabbitmqctl list_exchanges

# List bindings
rabbitmqctl list_bindings

# Check cluster status
rabbitmqctl cluster_status

# Add user
rabbitmqctl add_user username password

# Set permissions
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"
```

## Performance Tips

1. **Use appropriate exchange type** for your routing needs
2. **Enable prefetch** to limit unacknowledged messages
3. **Use lazy queues** for large backlogs
4. **Monitor queue lengths** to detect bottlenecks
5. **Cluster properly** for high availability
6. **Use message TTL** to prevent queue buildup
7. **Implement circuit breakers** in consumers

## Tham khảo

- [[Software Design|🧠 Software Design]]
- [[Cloud|☁️ Cloud Platforms]]
- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [AMQP Protocol](https://www.amqp.org/)

## Tags liên quan

#rabbitmq #message-broker #amqp #messaging #microservices #async #distributed-systems #backend
