---
title: "Observer Pattern"
tags:
  - Engineering
  - Software-Design
  - Design-Patterns
  - Behavioral
  - Go
  - Java
---

# Observer Pattern

## Giới thiệu

**Observer pattern** là một trong những **Design Patterns** thuộc nhóm **Behavioral**. Nó cho phép đối tượng (gọi là **Subject**) thông báo đến các đối tượng khác (gọi là **Observer**) về những thay đổi trong trạng thái của nó mà không cần biết cụ thể các đối tượng này là ai.

## Các thành phần chính của Observer pattern

1. **Subject**: Đây là đối tượng mà các Observer đăng ký để nhận thông báo. Nó duy trì một danh sách các Observer và cung cấp các phương thức để thêm, xóa và thông báo cho các Observer khi có sự thay đổi.

2. **Observer**: Đây là giao diện hoặc lớp cơ sở mà tất cả các Observer cụ thể phải triển khai. Mỗi khi trạng thái của Subject thay đổi, các Observer sẽ nhận được thông báo qua phương thức `update`.

3. **ConcreteSubject**: Đây là một lớp cụ thể triển khai Subject. Nó lưu trữ trạng thái của đối tượng mà các Observer quan tâm và thông báo cho tất cả Observer mỗi khi trạng thái thay đổi.

4. **ConcreteObserver**: Đây là các lớp cụ thể triển khai giao diện Observer. Mỗi khi nhận được thông báo từ Subject, các ConcreteObserver sẽ cập nhật trạng thái của mình.

## Cách hoạt động

- Các Observer đăng ký (hoặc hủy đăng ký) để nhận thông báo từ Subject.
- Khi trạng thái của Subject thay đổi, nó sẽ thông báo cho tất cả các Observer đã đăng ký.
- Mỗi Observer có thể tự quyết định làm gì với thông báo mà nó nhận được (ví dụ: cập nhật giao diện người dùng, ghi log, v.v.).

## Ưu điểm

- **Giảm sự kết nối chặt chẽ (Decoupling)**: Subject không cần thiết phải biết chi tiết về các Observer, chỉ cần biết rằng chúng thực hiện một giao diện cụ thể.
- **Linh hoạt**: Dễ dàng thêm hoặc bớt các Observer mà không ảnh hưởng đến Subject hoặc các Observer khác.

## Nhược điểm

- **Hiệu suất**: Nếu có quá nhiều Observer đăng ký hoặc Subject thay đổi trạng thái quá thường xuyên, điều này có thể dẫn đến việc quá nhiều thông báo được gửi đi, làm giảm hiệu suất của hệ thống.
- **Khó kiểm soát**: Quản lý số lượng lớn Observer có thể trở nên phức tạp, đặc biệt là khi chúng có các trạng thái khác nhau hoặc khi có các vòng lặp tham chiếu giữa các Observer.

Observer pattern thường được sử dụng trong các hệ thống mà trạng thái của một đối tượng cần được các đối tượng khác biết đến ngay lập tức, ví dụ như các hệ thống event-driven hoặc trong các mô hình MVC (Model-View-Controller).

## Ví dụ cụ thể

Giả sử bạn có ứng dụng dự báo thời tiết. Có một đối tượng **WeatherData** (Subject) cung cấp dữ liệu thời tiết và các thành phần khác như màn hình hiển thị, logger, hoặc các thành phần khác là các Observer. Mỗi khi dữ liệu thời tiết thay đổi, **WeatherData** sẽ thông báo cho tất cả các Observer đã đăng ký để chúng có thể cập nhật trạng thái của mình tương ứng.

## Ví dụ trong Java

```java
// Observer interface khai báo phương thức cập nhật,
// mà các đối tượng Observer cụ thể sẽ triển khai.
interface Observer {
    void update(String data);
}

// Subject interface khai báo một tập hợp các phương thức
// để quản lý các đối tượng Observer.
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// ConcreteSubject có chứa trạng thái quan trọng
// và thông báo cho các Observer khi trạng thái thay đổi.
class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String state;

    // Các phương thức quản lý Observer
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    // Phương thức thông báo cho tất cả các observer về thay đổi
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(state);
        }
    }

    // Một phương thức cụ thể để thay đổi trạng thái
    public void setState(String state) {
        this.state = state;
        notifyObservers();
    }

    public String getState() {
        return state;
    }
}

// ConcreteObserver thực hiện các hành động cụ thể để phản ứng lại
// các thay đổi trong ConcreteSubject mà nó đang theo dõi.
class ConcreteObserver implements Observer {
    private String observerState;
    private ConcreteSubject subject;

    public ConcreteObserver(ConcreteSubject subject) {
        this.subject = subject;
    }

    // Phương thức cập nhật sẽ được gọi khi có thông báo từ Subject
    @Override
    public void update(String data) {
        this.observerState = data;
        System.out.println("Observer's state updated to: " + observerState);
    }
}

// Cách sử dụng Observer pattern
public class Main {
    public static void main(String[] args) {
        // Tạo một ConcreteSubject
        ConcreteSubject subject = new ConcreteSubject();

        // Tạo các ConcreteObserver và đăng ký chúng với Subject
        ConcreteObserver observer1 = new ConcreteObserver(subject);
        ConcreteObserver observer2 = new ConcreteObserver(subject);

        subject.attach(observer1);
        subject.attach(observer2);

        // Thay đổi trạng thái của Subject
        subject.setState("New State");
        // Output:
        // Observer's state updated to: New State
        // Observer's state updated to: New State

        // Xóa một Observer
        subject.detach(observer1);

        // Thay đổi trạng thái của Subject lần nữa
        subject.setState("Another State");
        // Output:
        // Observer's state updated to: Another State
    }
}
```

## Phân tích mã

1. **Observer Interface**: Interface Observer khai báo phương thức `update(String data)`, mà tất cả các Observer cụ thể phải triển khai.

2. **Subject Interface**: Interface Subject khai báo các phương thức `attach`, `detach` và `notifyObservers` để quản lý các Observer.
   - `attach` và `detach` để thêm và xóa các Observer khỏi danh sách.
   - `notifyObservers` để thông báo cho tất cả các Observer về sự thay đổi.

3. **ConcreteSubject**: `ConcreteSubject` triển khai `Subject`, lưu trữ trạng thái và danh sách các Observer.
   - Khi trạng thái thay đổi, phương thức `notifyObservers` sẽ được gọi để thông báo cho tất cả các Observer đã đăng ký bằng cách gọi phương thức `update` trên mỗi Observer.

4. **ConcreteObserver**: `ConcreteObserver` triển khai `Observer` có một phương thức `update` để cập nhật trạng thái của nó khi nhận được thông báo từ `ConcreteSubject`.

5. **MainClass**: Trong hàm `main`, tạo một `ConcreteSubject` và hai `ConcreteObserver`.
   - Các Observer được đăng ký với Subject.
   - Khi trạng thái của Subject thay đổi, các Observer sẽ nhận được thông báo và cập nhật trạng thái của mình.

## Ví dụ trong Go

```go
package main

import (
	"fmt"
)

// Observer interface
type Observer interface {
	Update(data string)
}

// Subject interface
type Subject interface {
	RegisterObserver(observer Observer)
	RemoveObserver(observer Observer)
	NotifyObservers()
}

// ConcreteSubject struct
type ConcreteSubject struct {
	observers []Observer
	state     string
}

func (s *ConcreteSubject) RegisterObserver(observer Observer) {
	s.observers = append(s.observers, observer)
}

func (s *ConcreteSubject) RemoveObserver(observer Observer) {
	for i, o := range s.observers {
		if o == observer {
			s.observers = append(s.observers[:i], s.observers[i+1:]...)
			break
		}
	}
}

func (s *ConcreteSubject) NotifyObservers() {
	for _, observer := range s.observers {
		observer.Update(s.state)
	}
}

func (s *ConcreteSubject) SetState(state string) {
	s.state = state
	s.NotifyObservers()
}

func (s *ConcreteSubject) GetState() string {
	return s.state
}

// ConcreteObserver struct
type ConcreteObserver struct {
	name string
}

func (o *ConcreteObserver) Update(data string) {
	fmt.Printf("%s received update: %s\n", o.name, data)
}

// Main function to demonstrate the pattern
func main() {
	// Create a new ConcreteSubject
	subject := &ConcreteSubject{}

	// Create two observers
	observer1 := &ConcreteObserver{name: "Observer 1"}
	observer2 := &ConcreteObserver{name: "Observer 2"}

	// Register the observers with the subject
	subject.RegisterObserver(observer1)
	subject.RegisterObserver(observer2)

	// Change the state of the subject
	subject.SetState("New State 1")
	// Output:
	// Observer 1 received update: New State 1
	// Observer 2 received update: New State 1

	// Remove observer1
	subject.RemoveObserver(observer1)

	// Change the state again
	subject.SetState("New State 2")
	// Output:
	// Observer 2 received update: New State 2
}
```

## Use Cases phổ biến

1. **Event Handling Systems**: GUI frameworks, event-driven architectures
2. **MVC Architecture**: Model thông báo cho View khi dữ liệu thay đổi
3. **Pub/Sub Systems**: Message queues, notification services
4. **Reactive Programming**: RxJS, React state management
5. **Real-time Updates**: Chat applications, live dashboards

## Liên quan

- [[Design-Patterns|🎨 Design Patterns]]
- [[SOLID|⚡ SOLID Principles]]
- [[RabbitMQ|📬 RabbitMQ]]
- [[Software Design|🏗️ Software Design]]
