## II. Threads trong python.
### 1. Khởi tạo 
#### Khởi tạo từ hàm
(Không dùng cách này)

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