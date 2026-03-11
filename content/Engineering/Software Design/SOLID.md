---
title: "SOLID Principles"
tags:
  - Engineering
  - Software-Design
  - Go
  - OOP
  - Best-Practices
---

# SOLID Principles

## Giới thiệu

**SOLID** là một tập hợp năm nguyên tắc thiết kế phần mềm được đề xuất bởi **Robert C. Martin** (Uncle Bob) vào năm 2000. Những nguyên tắc này giúp phát triển phần mềm dễ bảo trì, dễ mở rộng và giảm thiểu lỗi.

## Nguyên tắc 1: Single Responsibility Principle (SRP)

### Nguyên tắc trách nhiệm đơn lẻ

**Định nghĩa**: Một class chỉ nên có một lý do để thay đổi, nghĩa là nó chỉ nên chịu trách nhiệm cho một nhiệm vụ duy nhất.

**Ý nghĩa**: Việc tuân thủ nguyên tắc này giúp giảm sự phụ thuộc phức tạp của mã nguồn và làm cho việc bảo trì dễ dàng hơn. Khi một class có nhiều trách nhiệm, việc thay đổi một trách nhiệm có thể ảnh hưởng đến nhiều trách nhiệm khác.

**Ví dụ**: Một struct `Invoice` không nên chịu trách nhiệm cả về việc tính toán và in hóa đơn. Thay vào đó, chúng ta có thể tách ra thành hai struct riêng biệt.

```go
package main

import (
	"fmt"
)

type Invoice struct {
	Amount float64
	Tax    float64
}

func (i Invoice) CalculateTotal() float64 {
	return i.Amount + i.Tax
}

type InvoicePrinter struct{}

func (ip InvoicePrinter) PrintInvoice(i Invoice) {
	fmt.Printf("Invoice Amount: %.2f\n", i.Amount)
	fmt.Printf("Tax: %.2f\n", i.Tax)
	fmt.Printf("Total: %.2f\n", i.CalculateTotal())
}

func main() {
	inv := Invoice{Amount: 100, Tax: 10}
	printer := InvoicePrinter{}
	printer.PrintInvoice(inv)
}
```

Ở đây, `Invoice` chỉ chịu trách nhiệm về dữ liệu hóa đơn và tính toán, trong khi `InvoicePrinter` chịu trách nhiệm in hóa đơn.

## Nguyên tắc 2: Open/Closed Principle (OCP)

### Nguyên tắc mở rộng/đóng

**Định nghĩa**: Các thực thể phần mềm (class, module, function, v.v.) nên mở để mở rộng nhưng đóng để thay đổi.

**Ý nghĩa**: Nguyên tắc này khuyến khích mở rộng chức năng của một thực thể mà không cần thay đổi mã gốc, giúp giảm thiểu lỗi khi thêm chức năng mới và tăng cường khả năng tái sử dụng.

**Ví dụ**: Thay vì sửa đổi struct gốc khi thêm một loại động vật mới, chúng ta có thể mở rộng chương trình bằng cách thêm các struct mới.

```go
package main

import (
	"fmt"
)

type Animal interface {
	Speak() string
}

type Dog struct{}

func (d Dog) Speak() string {
	return "Woof!"
}

type Cat struct{}

func (c Cat) Speak() string {
	return "Meow!"
}

func PrintAnimalSound(a Animal) {
	fmt.Println(a.Speak())
}

func main() {
	dog := Dog{}
	cat := Cat{}

	PrintAnimalSound(dog)
	PrintAnimalSound(cat)
}
```

Ở đây, chúng ta có thể thêm nhiều loại động vật khác mà không cần thay đổi hàm `PrintAnimalSound`.

## Nguyên tắc 3: Liskov Substitution Principle (LSP)

### Nguyên tắc thay thế Liskov

**Định nghĩa**: Các đối tượng của lớp con phải có thể thay thế cho các đối tượng lớp cha mà không thay đổi tính đúng đắn của chương trình.

**Ý nghĩa**: Điều này đảm bảo rằng các lớp con có thể được sử dụng mà không cần quan tâm đến chi tiết thực thi của chúng, giúp cho việc mở rộng và duy trì hệ thống dễ dàng hơn.

**Ví dụ**: Giả sử chúng ta có một lớp `Bird` đại diện cho các loài chim và một lớp con `Penguin` đại diện cho loài chim cánh cụt. Trong đó, lớp `Bird` có một phương thức `Fly`.

```go
package main

import "fmt"

type Bird struct {
	Name string
}

func (b Bird) Fly() {
	fmt.Println(b.Name, "is flying")
}

type Penguin struct {
	Bird
}

func (p Penguin) Fly() {
	fmt.Println(p.Name, "can't fly")
}

func MakeBirdFly(b Bird) {
	b.Fly()
}

func main() {
	eagle := Bird{Name: "Eagle"}
	penguin := Penguin{Bird{Name: "Penguin"}}

	MakeBirdFly(eagle)
	MakeBirdFly(penguin.Bird)
}
```

### Phân tích

- **Bird**: là lớp cơ bản, đại diện cho các loài chim, với khả năng bay (`Fly`).
- **Penguin**: là con của lớp `Bird`. Tuy nhiên loài chim cánh cụt không thể bay, nên phương thức `Fly` của nó sẽ in ra thông báo rằng không thể bay.

**Vấn đề**: Điều này phá vỡ nguyên tắc LSP vì lớp `Penguin` không thể thay thế hoàn toàn cho lớp `Bird`.

**Cách khắc phục**: Tạo một interface `Flyer` dành riêng cho các đối tượng có khả năng bay, và tránh việc kế thừa khi hành vi của lớp con không phù hợp với lớp cha.

## Nguyên tắc 4: Interface Segregation Principle (ISP)

### Nguyên tắc phân tách giao diện

**Định nghĩa**: Không nên ép buộc client phải thực hiện các giao diện mà họ không sử dụng. Thay vào đó, các giao diện nên được thiết kế nhỏ gọn và chuyên biệt.

**Ý nghĩa**: Nguyên tắc này giúp giảm sự phụ thuộc và sự phức tạp trong hệ thống bằng cách tạo ra các giao diện nhỏ và có mục đích cụ thể.

**Ví dụ**: Không nên ép buộc một struct thực hiện các phương thức mà nó không cần.

```go
package main

import (
	"fmt"
)

type Printer interface {
	Print() string
}

type Scanner interface {
	Scan() string
}

type MultiFunctionDevice interface {
	Printer
	Scanner
}

type SimplePrinter struct{}

func (sp SimplePrinter) Print() string {
	return "Printing..."
}

type AdvancedPrinterScanner struct{}

func (aps AdvancedPrinterScanner) Print() string {
	return "Printing..."
}

func (aps AdvancedPrinterScanner) Scan() string {
	return "Scanning..."
}

func main() {
	sp := SimplePrinter{}
	fmt.Println(sp.Print())

	aps := AdvancedPrinterScanner{}
	fmt.Println(aps.Print())
	fmt.Println(aps.Scan())
}
```

Ở đây, `SimplePrinter` chỉ cần thực hiện interface `Printer`, không cần phải implement phương thức `Scan`.

## Nguyên tắc 5: Dependency Inversion Principle (DIP)

### Nguyên tắc đảo ngược sự phụ thuộc

**Định nghĩa**: Các module cấp cao không nên phụ thuộc vào các module cấp thấp. Cả hai nên phụ thuộc vào các abstraction.

**Ý nghĩa**: DIP khuyến khích sử dụng interface hoặc abstract class để giảm sự phụ thuộc giữa các module, từ đó tăng cường tính linh hoạt và khả năng tái sử dụng của hệ thống.

**Ví dụ**: Các module cấp cao không nên phụ thuộc vào các module cấp thấp, mà cả hai nên phụ thuộc vào abstraction.

```go
package main

import (
	"fmt"
)

// Abstraction
type Database interface {
	Connect() string
}

// Concrete implementation
type MySQLDatabase struct{}

func (db MySQLDatabase) Connect() string {
	return "Connected to MySQL"
}

// High-level module
type Application struct {
	Database Database
}

func (app Application) Start() {
	fmt.Println(app.Database.Connect())
}

func main() {
	mysql := MySQLDatabase{}
	app := Application{Database: mysql}
	app.Start()
}
```

Trong ví dụ này, `Application` không phụ thuộc trực tiếp vào `MySQLDatabase`, mà phụ thuộc vào abstraction `Database`, giúp cho việc thay đổi hoặc mở rộng ứng dụng dễ dàng hơn mà không ảnh hưởng đến các phần khác của chương trình.

## Kết luận

Tuân thủ **SOLID** giúp tạo ra phần mềm có cấu trúc tốt, dễ bảo trì và mở rộng. Đây là những nguyên tắc cơ bản giúp lập trình viên thiết kế hệ thống linh hoạt và tránh các vấn đề thường gặp như mã phức tạp, khó bảo trì hoặc lỗi phát sinh khi mở rộng hệ thống.

## Liên quan

- [[Go|🔷 Go Programming]]
- [[RabbitMQ|📬 RabbitMQ]]
- [[Software Design|🏗️ Software Design]]
