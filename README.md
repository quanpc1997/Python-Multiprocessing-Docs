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

## II. Threads trong python.
### 1. Khởi tạo
#### Khởi tạo từ hàm - không nên dùng
#### Khởi tạo kế thừa
Kế thừa từ class threading

```python
import threading
import time

class FirstThread(threading.Thread):
  def __init__(self, thread_id, thread_name, counter):
    threading.Thread.__init__(self)
    self.thread_id = thread_id
    self.thread_name = thread_name
    self.counter = counter

  def run(self):
    print("Start thread {}!".format(self.thread_name))
    while (self.counter):
      time.sleep(0.01)
      print("{} : {}".format(self.thread_name, self.counter))
      self.counter -= 1
    print("End thread {}".format(self.thread_name))


thread1 = FirstThread(1, "khanh thread", 5)
thread2 = FirstThread(2, "ai thread", 5)

thread1.start()
thread2.start()

thread1.join()
thread2.join()
```

Để triển khai theo cách này ta bắt buộc phải kế thừa class threading và trong class vừa tạo bắt buộc phải khởi tạo 2 hàm là init và run.
- Hàm _start()_ để chạy các thread đó.
- Hàm _join()_ khi được gọi sẽ chấm dứt các thread gọi đến nó thay vì để nó tiếp tục nhàn rỗi.

### 2. Cơ chế bất đồng bộ - asynchronous
Trong python nếu để mặc định thì các thread sẽ theo cơ chế GIL nên tuy không chạy song song nhưng vẫn là không theo thứ tự. Đó là bất đồng bộ trong python

### 3. Cơ chế đồng bộ - synchronous - Cơ chế Thread Lock
Đây là cơ chế cho phép 1 thread chạy xong rồi thì các thread khác mới đến lượt.




## Reference:
[Realpython](https://realpython.com/intro-to-python-threading/)
[Digitalocean](https://www.digitalocean.com/community/tutorials/python-multiprocessing-example)
[phamdinhkhanh](https://phamdinhkhanh.github.io/2020/11/30/ParallelComputingPython.html)
[viblo.asia](https://viblo.asia/p/threads-and-processes-in-python-yMnKMzWEZ7P)