# Cơ chế đồng bộ - synchronous
## 1. Đồng bộ hoá cơ bản với Lock và Rlock.
Tương tự với Thread thì Multiprocessing cũng có Lock và RLock. Và các cơ chế đều giống threads. Để xem chi tiết xem [Đồng bộ trong Thread](/threads/4-synchronous.md)

- Khai báo:
```python
import multiprocessing
lock = multiprocessing.Lock()
rlock = multiprocessing.RLock()
```
## 2. Đồng bộ thông qua Queue
Queue là một định dạng stack an toàn khi làm việc với multi thread và process. Chúng ta có thể tạo ra một queue và cho phép các thread, process truy cập dữ liệu mà không bị hiện tượng concurrency vì dữ liệu được truy suất và sử dụng một lần bởi một thread hoặc process.

Bên dưới chúng ta sẽ lấy ví dụ về việc sử dụng 2 process để đọc các dữ liệu trong một queue. Hai process này tới phiên của mình sẽ lấy ra các phần từ nằm trong queue theo kiểu FIFO (First Come First Out).

```python
from multiprocessing import Process, Queue
import time

def _counter_queue(queue, process_name, max_count):
  # lock.acquire()
  while max_count:
    time.sleep(0.01)
    value = queue.get()
    print("{}: {}".format(process_name, value))
    max_count -= 1
  # lock.release()

q = Queue()
for i in range(10):
  q.put(i)
max_count = 5
# lock = Lock()
exec1 = Process(target=_counter_queue, args=(q, "khanh process", 5)) # pass counter and thread_name into method _counter
exec2 = Process(target=_counter_queue, args=(q, "ai process", 5))
execs = [exec1, exec2]

exec1.start()
exec2.start()

for exec in execs:
  exec.join()

# Kết quả:
# khanh process: 0
# ai process: 1
# khanh process: 2
# ai process: 3
# khanh process: 4
# ai process: 5
# khanh process: 6
# ai process: 7
# khanh process: 8
# ai process: 9
```
Như vậy không có bất kỳ một data nào được sử dụng chung giữa 2 processes nên tránh được concurrency.

## 3. Deadlock
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