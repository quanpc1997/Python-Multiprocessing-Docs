# Parallel Computing on Python

## I. Process và thread
### 1. Process là gì?
Process là tiến trình để chạy một phần mềm. Khi bạn khởi động một program tức là đang khởi tạo 1 hay nhiều process. Hệ điều hành khi đó sẽ cung cấp các tài nguyên về memory, CPU, disk, bandwidth cho process để chạy ứng dụng đó.
Đặc điểm:
    - Được tạo bởi OS để chạy các chương trình.
    - Process có thể có multiple thread.
    - Chi phí của processes cao hơn threads bởi việc đóng và mở chương trình mất nhiều thời gian hơn.
    - 2 processes có thể thực thi code đồng thời trong cùng một chương trình python.
    - Dữ liệu của process thì được thiét kế private nên một process sẽ mất một cách khó hơn để chia sẻ dữ liệu với các process khác. Để chia sẻ dữ liệu giữa các process yêu cầu những cấu trúc giữ liệu đặc biệt đòi hỏi thời gian IO.

### 2. Thread là gì?
- Các threads giống như các Processes nhỏ bên trong một process. 
- Là một phiên bản nhẹ hơn của Process.
- Chúng được thiết kế để chia sẻ không gian bộ nhớ và đọc ghi dữ liệu hiệu quả vào các biến giống nhau.
- 2 Threads không thể thực thi code đồng thời trong cùng một chương trình python.

### 3. Multiple-threads vs Multiple-processes
Trong python thì process và thread cùng kế thừa chung một interface là một base thread. Chúng sẽ có những đặc tính chung, nhưng thread là một phiên bản nhẹ hơn so với process. Do đó việc khởi tạo thread sẽ nhanh hơn. Một điểm khác biệt nữa đó là thread được thiết kế để có thể hoạt động tương tác lẫn nhau. Các threads trong cùng một process sẽ chia sẻ được dữ liệu qua lại nên có lợi thế về I/O. Dữ liệu của process thì được thiết kế private nên một process không thể chia sẻ dữ liệu với các process khác. Đây là lý do chúng ta cần nhiều threads hoạt động trong một process.

## II. Threads trong Python.
### [1. Khởi tạo](/threads/1-khoi-tao-thread.md)
### [2. ThreadPoolExcutor](/threads/2-thread-pool-excutor.md)
### [3. Cơ chế bất đồng bộ - asynchronous](/threads/3-asynchronous.md)
### [4. Cơ chế đồng bộ - synchronous](/threads/4-synchronous.md)

## III. Processes trong Python.
### [1. Khởi tạo](/processes/1-khoi-tao-process.md)
### [2. Process Pool](/processes/2-Process-Pool.md)
### [3. Chia sẻ dữ liệu trong Process](/processes/3-share-object-with-process.md)
### [4. Cơ chế đồng bộ - synchronous](/processes/4-synchronous.md)

## IV. Queue trong Threading và Processing
Queue là một định dạng stack an toàn khi làm việc với multi thread và process. Chúng ta có thể tạo ra một queue và cho phép các thread, process truy cập dữ liệu mà không bị hiện tượng concurrency vì dữ liệu được truy suất và sử dụng một lần bởi một thread hoặc process.

Bên dưới chúng ta sẽ lấy ví dụ về việc sử dụng 2 process để đọc các dữ liệu trong một queue. Hai process này tới phiên của mình sẽ lấy ra các phần từ nằm trong queue theo kiểu FIFO (First Come First Out).

**(Multiple Thread tương tự)**
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

## V. Process Pool vs Thread Pool
Nhìn chung 2 cái này giống nhau. Chỉ khác nhau ở chỗ Process Pool chạy song song được còn Thread Pool thì không. Ngoài ra Thread Pool sẽ có lợi hơn về I/O  vì các threads được thiết kế để có thể chia sẻ data qua lại lẫn nhau còn giữa các thread. Đổi lại thì process sẽ có lợi hơn khi không bị giới hạn bởi CPU và Memory. **Nên lời khuyên là nếu task của bạn gặp phải giới hạn về I/O bound thì nên sử dụng thread pool và giới hạn về CPUs bound thì nên sử dụng process pool.**
## Reference:

[Python Docs](https://docs.python.org/3/library/multiprocessing.html)

[Digitalocean](https://www.digitalocean.com/community/tutorials/python-multiprocessing-example)

[phamdinhkhanh](https://phamdinhkhanh.github.io/2020/11/30/ParallelComputingPython.html)

[viblo.asia](https://viblo.asia/p/threads-and-processes-in-python-yMnKMzWEZ7P)
