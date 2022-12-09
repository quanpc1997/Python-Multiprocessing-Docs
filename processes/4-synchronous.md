# Cơ chế đồng bộ - synchronous
## 1. Đồng bộ hoá cơ bản với Lock và Rlock.
Tương tự với Thread thì Multiprocessing cũng có Lock và RLock. Và các cơ chế đều giống threads. Để xem chi tiết xem [Đồng bộ trong Thread](/threads/4-synchronous.md)

- Khai báo:
```python
import multiprocessing
lock = multiprocessing.Lock()
rlock = multiprocessing.RLock()
```

## 2. Deadlock
Tương tự như Thread. Để xem chi tiết xem [Đồng bộ trong Thread](/threads/4-synchronous.md)

Ví dụ 1: Chương trình không thể tự động thoát.
```python
import multiprocessing

l = multiprocessing.Lock()
print("before first acquire")
l.acquire()
print("before second acquire")
l.acquire()
print("acquired lock twice")

# Kết quả:
# before first acquire
# before second acquire
```

Ví dụ 2:
```python
import multiprocessing

l = multiprocessing.RLock()
print("before first acquire")
l.acquire()
print("before second acquire")
l.acquire()
print("acquired lock twice")

# Kết quả:
# before first acquire
# before second acquire
# acquired lock twice
```

## Reference
[Multiprocessing Lock in Python](https://superfastpython.com/multiprocessing-mutex-lock-in-python/)

[Synchronization and Pooling of processes in Python](https://www.tutorialspoint.com/synchronization-and-pooling-of-processes-in-python)