---
title: "Python Functions"
tags:
  - Python
  - Programming
  - Functions
  - Type-Hints
---

# Định nghĩa và cách dùng hàm trong Python

Hàm trong Python là một khối mã có thể tái sử dụng để thực hiện một nhiệm vụ cụ thể, giúp mã dễ đọc, tổ chức và bảo trì hơn.

## 1. Định nghĩa hàm

Hàm được định nghĩa bằng từ khóa `def`, theo sau là tên hàm và cặp ngoặc tròn chứa các tham số (nếu có).

### Cấu trúc cơ bản

```python
def ten_ham(tham_so_1, tham_so_2, ...):
    # Thân hàm
    return gia_tri_tra_ve
```

**Các thành phần:**
- **Tên hàm**: Nên đặt tên rõ ràng, tuân theo quy tắc `snake_case`
- **Tham số**: Các giá trị được truyền vào hàm để xử lý
- **return**: Trả về giá trị từ hàm (không bắt buộc nếu hàm không cần trả về gì)

## 2. Ví dụ về hàm cơ bản

### Hàm không có tham số và không trả về giá trị

```python
def chao():
    print("Xin chào!")

chao()  # Gọi hàm
# Output: Xin chào!
```

### Hàm có tham số và có giá trị trả về

```python
def tong(a, b):
    return a + b

ket_qua = tong(3, 5)
print(ket_qua)  # Output: 8
```

## 3. Hàm với tham số mặc định

Có thể đặt giá trị mặc định cho tham số trong hàm. Nếu không truyền giá trị cho tham số khi gọi hàm, nó sẽ dùng giá trị mặc định.

```python
def chao(ten="bạn"):
    print(f"Xin chào, {ten}!")

chao()         # Output: Xin chào, bạn!
chao("Nam")    # Output: Xin chào, Nam!
```

### Best practices với default parameters

```python
# ❌ Tránh dùng mutable objects làm default
def them_vao_danh_sach(item, danh_sach=[]):
    danh_sach.append(item)
    return danh_sach

# ✅ Dùng None và khởi tạo bên trong hàm
def them_vao_danh_sach(item, danh_sach=None):
    if danh_sach is None:
        danh_sach = []
    danh_sach.append(item)
    return danh_sach
```

## 4. Hàm với tham số không xác định

### *args - Tham số vị trí không xác định

Sử dụng `*args` khi muốn truyền một số lượng tham số không xác định cho hàm.

```python
def tong_tat_ca(*args):
    return sum(args)

print(tong_tat_ca(1, 2, 3, 4))  # Output: 10
print(tong_tat_ca(5, 10))       # Output: 15
```

### **kwargs - Tham số từ khóa không xác định

Sử dụng `**kwargs` khi muốn truyền các tham số dưới dạng từ khóa.

```python
def chi_tiet(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

chi_tiet(ten="Nam", tuoi=25, dia_chi="Hà Nội")
# Output:
# ten: Nam
# tuoi: 25
# dia_chi: Hà Nội
```

### Kết hợp *args và **kwargs

```python
def ham_linh_hoat(*args, **kwargs):
    print("Positional arguments:", args)
    print("Keyword arguments:", kwargs)

ham_linh_hoat(1, 2, 3, ten="Nam", tuoi=25)
# Output:
# Positional arguments: (1, 2, 3)
# Keyword arguments: {'ten': 'Nam', 'tuoi': 25}
```

## 5. Type Hints - Gợi ý kiểu dữ liệu

Type hints giúp chỉ định kiểu dữ liệu của tham số và giá trị trả về, giúp mã rõ ràng hơn và IDE hỗ trợ tốt hơn.

### Cú pháp cơ bản

```python
def ten_ham(tham_so: kieu_du_lieu) -> kieu_tra_ve:
    return gia_tri_tra_ve
```

### Ví dụ với kiểu dữ liệu cơ bản

```python
def tong(a: int, b: int) -> int:
    return a + b

def chia(a: float, b: float) -> float:
    return a / b

def in_loi_nhan() -> None:
    print("Đây là thông báo.")
```

### Type hints với kiểu phức tạp

```python
from typing import List, Dict, Tuple, Optional, Union

def tao_danh_sach_so() -> List[int]:
    return [1, 2, 3, 4]

def lay_thong_tin() -> Dict[str, Union[str, int]]:
    return {"ten": "Nam", "tuoi": 25}

def tim_kiem(keyword: str) -> Optional[str]:
    # Trả về str hoặc None
    if keyword == "python":
        return "Found"
    return None

def toa_do() -> Tuple[float, float]:
    return (10.5, 20.3)
```

### Type hints với Python 3.10+

```python
# Python 3.10+ có thể dùng | thay cho Union
def xu_ly(data: str | int | None) -> list[str]:
    # list[str] thay cho List[str]
    if data is None:
        return []
    return [str(data)]
```

## 6. Trả về nhiều giá trị từ hàm

### Tuple - Đơn giản và gọn nhẹ

Phù hợp khi ít giá trị và không cần gán nhãn.

```python
def lay_thong_tin() -> Tuple[str, int, str]:
    return "Nam", 25, "Hà Nội"

ten, tuoi, dia_chi = lay_thong_tin()
print(f"{ten}, {tuoi} tuổi, sống tại {dia_chi}")
```

### Dictionary - Rõ ràng về tên giá trị

Phù hợp khi cần rõ ràng về tên các giá trị.

```python
def thong_tin_sinh_vien() -> Dict[str, Union[str, int]]:
    return {
        "ten": "Nam",
        "tuoi": 25,
        "dia_chi": "Hà Nội"
    }

info = thong_tin_sinh_vien()
print(info["ten"])  # Nam
```

### NamedTuple - Lightweight và có tên thuộc tính

```python
from typing import NamedTuple

class SinhVienInfo(NamedTuple):
    ten: str
    tuoi: int
    dia_chi: str

def lay_sinh_vien() -> SinhVienInfo:
    return SinhVienInfo(ten="Nam", tuoi=25, dia_chi="Hà Nội")

sv = lay_sinh_vien()
print(sv.ten)    # Nam
print(sv.tuoi)   # 25
```

### DataClass - Cấu trúc rõ ràng và linh hoạt

Kết hợp tính gọn gàng và có cấu trúc rõ ràng, sử dụng khi có nhiều thuộc tính.

```python
from dataclasses import dataclass

@dataclass
class SinhVien:
    ten: str
    tuoi: int
    dia_chi: str
    
    def gioi_thieu(self):
        return f"{self.ten}, {self.tuoi} tuổi"

def thong_tin_sinh_vien() -> SinhVien:
    return SinhVien(ten="Nam", tuoi=25, dia_chi="Hà Nội")

sv = thong_tin_sinh_vien()
print(sv.ten)              # Nam
print(sv.gioi_thieu())     # Nam, 25 tuổi
```

## 7. Lambda Functions (Anonymous Functions)

Hàm lambda là hàm ngắn gọn, thường dùng cho các thao tác đơn giản.

```python
# Hàm thông thường
def binh_phuong(x):
    return x ** 2

# Lambda function
binh_phuong_lambda = lambda x: x ** 2

print(binh_phuong(5))         # 25
print(binh_phuong_lambda(5))  # 25

# Sử dụng với map, filter, sorted
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# Sorting với lambda
students = [
    {"name": "Nam", "age": 25},
    {"name": "Lan", "age": 22},
]
sorted_students = sorted(students, key=lambda s: s["age"])
```

## 8. Decorators - Trang trí hàm

Decorators cho phép thêm chức năng cho hàm mà không sửa đổi code gốc.

```python
import time
from functools import wraps

def timer_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer_decorator
def slow_function():
    time.sleep(1)
    return "Done"

slow_function()
# Output: slow_function took 1.0012 seconds
```

## 9. Best Practices

### ✅ DO - Nên làm

```python
# 1. Đặt tên hàm rõ ràng, động từ
def calculate_total_price(items: List[Item]) -> float:
    pass

# 2. Single Responsibility - Một hàm một việc
def validate_email(email: str) -> bool:
    pass

def send_email(to: str, subject: str, body: str) -> None:
    pass

# 3. Dùng type hints
def get_user_by_id(user_id: int) -> Optional[User]:
    pass

# 4. Docstring cho hàm phức tạp
def calculate_discount(price: float, discount_rate: float) -> float:
    """
    Tính giá sau khi giảm giá.
    
    Args:
        price: Giá gốc
        discount_rate: Tỷ lệ giảm giá (0.0 - 1.0)
    
    Returns:
        Giá sau giảm
    """
    return price * (1 - discount_rate)
```

### ❌ DON'T - Tránh làm

```python
# 1. Hàm quá dài (> 50 dòng)
# 2. Tên hàm không rõ nghĩa
def process_data():  # ❌ Quá chung chung
    pass

# 3. Side effects không cần thiết
def calculate_sum(numbers):
    print("Calculating...")  # ❌ Side effect
    return sum(numbers)

# 4. Modify arguments (mutating)
def add_item(items: List[str]):
    items.append("new")  # ❌ Thay đổi input
    return items
```

## 10. Tổng kết

| Khái niệm | Mục đích | Ví dụ |
|-----------|----------|-------|
| **def** | Định nghĩa hàm | `def hello(): pass` |
| **return** | Trả về giá trị | `return result` |
| **Default params** | Giá trị mặc định | `def func(x=10):` |
| ***args** | Tham số vị trí linh hoạt | `def func(*args):` |
| ****kwargs** | Tham số từ khóa linh hoạt | `def func(**kwargs):` |
| **Type hints** | Gợi ý kiểu dữ liệu | `def func(x: int) -> str:` |
| **Lambda** | Hàm ngắn gọn | `lambda x: x * 2` |
| **Decorator** | Thêm chức năng cho hàm | `@decorator` |

### Khi nào dùng phương thức nào để trả về nhiều giá trị?

- **Tuple**: 2-3 giá trị đơn giản, không cần tên
- **Dictionary**: Cần rõ ràng về key, dễ mở rộng
- **NamedTuple**: Lightweight, immutable, có type hints
- **DataClass**: Nhiều thuộc tính, cần methods, mutable

## Liên quan

- [[index|🐍 Python Programming]]
- [[../Go/index|🔷 Go Programming]] - So sánh functions trong Go
- [[../../index|💻 Languages]]
- [[../../../Software Design/SOLID|⚡ SOLID Principles]] - Single Responsibility cho functions
