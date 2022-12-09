# II. Threads trong Python.
## 3. Cơ chế bất đồng bộ - asynchronous
Đây là cơ chế cho phép 1 thread chạy xong rồi thì các thread khác mới đến lượt.

### a. Race Conditions

Cho ví dụ sau và đoán xem __value__ sẽ cho kết quả là bao nhiêu ^^:

```python
import logging
from concurrent.futures import ThreadPoolExecutor
import time

class FakeDatabase:
    def __init__(self):
        self.value = 0

    def update(self, name):
        logging.info("Thread %s: starting update", name)
        local_copy = self.value
        local_copy += 1
        time.sleep(0.1)
        self.value = local_copy
        logging.info("Thread %s: finishing update", name)

if __name__ == "__main__":
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO,
                        datefmt="%H:%M:%S")

    database = FakeDatabase()
    logging.info("Testing update. Starting value is %d.", database.value)
    with ThreadPoolExecutor(max_workers=2) as executor:
        for index in range(2):
            executor.submit(database.update, index)
    logging.info("Testing update. Ending value is %d.", database.value)

# Kết quả:
# 14:20:07: Testing update. Starting value is 0.
# 14:20:07: Thread 0: starting update
# 14:20:07: Thread 1: starting update
# 14:20:07: Thread 1: finishing update
# 14:20:07: Thread 0: finishing update
# 14:20:07: Testing update. Ending value is 1.
```

**Ngạc nhiên chưa? :D**

Để giải thích cho việc này thì ta sẽ tìm hiểu xem trong trường hợp 1 thread sẽ chạy ntn. 2 thread(đại diện cho nhiều thread) sẽ chạy ntn

#### One thread
![Workflow của 1 thread](/threads/images/intro-threading-single-thread.webp)

- Khi __Thread 1__ chạy, __FakeDatabase.value__ = 0. Dòng đầu tiên của method update là __local_copy = self.value__, sẽ gán giá trị của __local_copy__ bằng giá trị của __self.value__ và bằng 0.
- Tiếp theo nó tăng giá trị của __local_copy__ lên 1. Tiếp theo hàm __time.sleep()__ được gọi. _Lúc này thread hiện tại sẽ được tạm dừng và cho phép các thread khác(nếu có) chạy_. Do chỉ có 1 luồng nên trong trường hợp này hàm time.sleep() không có tác dụng. 
- Khi __Thread 1__ tỉnh dậy, có tiếp tục copy ngược lại giá trị của __local_copy__ vào __self.value__. Bạn có thể thấy giá trị của __self.value__ trong __Thread 1__ đã được đặt giá trị 1.

#### Two thread
Bây giờ ta sẽ phân tích nếu chạy 2 thread thì chương trình trên sẽ chạy như thế nào.
- Đầu tiên chương trình sẽ chạy với __Thread 1__ đầu tiên với hàm __.update()__
![Bắt đầu chương trình với Thread 1](/threads/images/intro-threading-two-threads-part1.webp)

- Khi __Thread 1__ gọi __time.sleep()__. Nó sẽ cho phép Thread khác bắt đầu được chạy. Thread 2 lúc này sẽ được khởi động lên. Nó cũng copy giá trị của __database.value__ vào trong __local_copy__ và bằng 0. Lúc này __local_copy__ ở cả 2 thread lại chuyển giá trị sang 0. Sau đó __local_copy__ được __Thread 2__ cộng thêm 1 và bằng 1(*). Sau đó __Thread 2__ lại chạy __time.slee()__, nhường chỗ cho __Thread 1__ tỉnh dậy và chạy tiếp. Lúc này giá trị __database.value__ vẫn bằng 0.
![Bắt đầu chương trình với Thread 1](/threads/images/intro-threading-two-threads-part2.webp)
- __Thread 1__ tỉnh dậy và tiếp tục gán __self.value = local_copy__ và bằng 1 do (*) và kết thúc __Thread 1__. Sau đó __Thread 2__ chưa biết là __Thread 1__ đã update xong rồi và tiếp tục lại gán __self.value = local_copy__ và bằng 1 rồi cũng kết thúc. Kết quả là __database.value__ bằng 1.
![Bắt đầu chương trình với Thread 1](/threads/images/intro-threading-two-threads-part3.webp)
- Tóm lại hai luồng có quyền truy cập xen kẽ vào một đối tượng được chia sẻ duy nhất, ghi đè lên kết quả của nhau.

### b. Đồng bộ hoá cơ bản với Lock.
Để giải quyết vấn đề trên, chúng ta cần tìm ra một cách cho phép chỉ một thread đọc và thay đổi dữ liệu trong 1 thời điểm. Cách phổ biến nhất đó là gọi __Lock__ trong Python. __Lock__ ở đây là một đối tượng.

Nhớ lại cơ chế GIL trong Python thì chỉ có thread nào cầm khoá thì mới được hoạt động. Thì ở đây cũng vậy chỉ khác là trong cơ chế GIL thì các khoá sau 1 khoảng thời gian sẽ bắt buộc phải đưa cho thread khác cầm thì ở đây nó được giữ cho đến khi được người dùng giải phóng trong code.

Ở đây ta cần chú ý đến 2 hàm cơ bản đó là __.acquire()__ và __.release()__. 1 Thread sẽ gọi __my_look.acquire()__. **_Nếu Lock đã được giữ bởi 1 Thread khác thì thì thread đó phải đợi cho đến khi Lock được giải phóng_**. 

**_Một điều quan trọng nữa là nếu 1 thread giữ Lock nhưng không bao giờ giải phóng thì chương trình sẽ bị mắc kẹt_**. Điều này sẽ được làm rõ ở phần dưới.

May mắn là, Lock trong Python sẽ vận hành như một Context Manager, Vì vậy bạn có thể sử dụng nó trong __with__ và nó sẽ tự động giải phóng nếu có bất kì điều gì xảy ra.

Bây giờ hãy nhìn vào __FakeDatabase__ cùng với 1 __Lock__ được thêm vào. 
```python
import logging
import time
import threading
from concurrent.futures import ThreadPoolExecutor

logging.getLogger().setLevel(logging.DEBUG)

class FakeDatabase:
    def __init__(self):
        self.value = 0
        self._lock = threading.Lock()

    def locked_update(self, name):
        logging.info("Thread %s: starting update", name)
        logging.debug("Thread %s about to lock", name)
        with self._lock:
            logging.debug("Thread %s has lock", name)
            local_copy = self.value
            local_copy += 1
            time.sleep(0.1)
            self.value = local_copy
            logging.debug("Thread %s about to release lock", name)
        logging.debug("Thread %s after release", name)
        logging.info("Thread %s: finishing update", name)

if __name__ == "__main__":
    format = "%(asctime)s: %(message)s"
    logging.basicConfig(format=format, level=logging.INFO,
                        datefmt="%H:%M:%S")

    database = FakeDatabase()
    logging.info("Testing update. Starting value is %d.", database.value)
    with ThreadPoolExecutor(max_workers=2) as executor:
        for index in range(2):
            executor.submit(database.locked_update, index)
    logging.info("Testing update. Ending value is %d.", database.value)

# Kết quả:
# 16:06:58: Testing update. Starting value is 0.
# 16:06:58: Thread 0: starting update
# 16:06:58: Thread 1: starting update
# 16:06:58: Thread 0 about to lock
# 16:06:58: Thread 1 about to lock
# 16:06:58: Thread 0 has lock
# 16:06:58: Thread 0 about to release lock
# 16:06:58: Thread 0 after release
# 16:06:58: Thread 0: finishing update
# 16:06:58: Thread 1 has lock
# 16:06:58: Thread 1 about to release lock
# 16:06:58: Thread 1 after release
# 16:06:58: Thread 1: finishing update
# 16:06:58: Testing update. Ending value is 2.
```
Ta dễ thấy rằng khi một instance **FakeDatabase** được tạo, thì instance đó sẽ giữ luôn Lock. Và Thread nào tương tác với instance đó thì sẽ giữ luôn Lock đó. Và chỉ khi có lỗi nào đó(với **with**) hoặc đã hoàn thành task, hoặc là được giải phóng thì Lock mới được chuyển cho thread khác.

### c. Deadlock
Trước khi đọc đến phần này, hãy chắc chắn rằng bạn đã đọc và hiểu hết những nội dung bên trên. Như đã thấy, nếu như **Lock** được gọi bởi hàm **.acquire()** chương trình sẽ đợi đến khi mà thread nào đang giữ lock gọi **.release()**. 

> _Deadlock là tình trạng tiến trình dừng lại vĩnh viễn do một Thread đang đợi để được truy nhập 1 tài nguyên được sử dụng bởi 1 Thread khác và Thread này lại đang chờ để truy cập 1 tài nguyên mà Thread đầu tiên đang nắm giữ_

Deadlock sẽ xảy ra trong các trường hợp sau:
- Lỗi triển khai trong đó Lock chưa được giải phóng đúng cách.
- Sự cố thiết kế trong đó một chức năng tiện ích cần được gọi bởi các chức năng có thể đã có hoặc chưa có Lock.

Cho ví dụ sau và đoán xem chuyện gì sẽ xảy ra:
```python
import threading

l = threading.Lock()
print("before first acquire")
l.acquire()
print("before second acquire")
l.acquire()
print("acquired lock twice")

# Kết quả:
# before first acquire
# before second acquire
```
Giải thích: Khi **l.acquire()** được gọi lần thứ 2. Nó chờ cho **Lock** được giải phóng thì mới cho chạy tiếp các dòng tiếp theo.

### d. RLock()
Tương tự như Lock và sử dụng khái niệm "owning thread" (tiến trình sở hữu) và "recursion level" (có thể gọi method acquire() nhiều lần).

- **Owning thread**: Rlock object lưu giữ định danh (ID) của thread sở hữu nó. Do đó, khi một thread sở hữu Rlock, thread đó có thể gọi method acquire() nhiều lần mà không bị block (Khác với Lock, khi một thread đã dành được khóa, các tiến trình gọi acquire() sẽ bị block cho đến khi khóa được release). Hơn nữa, đối với Rlock, khóa chỉ được unlocked khi thread sở hữu gọi method release() (các lời gọi acquire() và release() có thể lồng nhau) và số lần gọi release() phải bằng với số lần gọi method acquire() trước đó (khác với Lock, khóa có thể được unlocked khi một thread bất kỳ gọi method release()).
- **Recursion level**: Một khi thread dành được khóa, thread đó có thể gọi acquire() nhiều lần mà không bị block.

Bởi vì đặc điểm trên, Rlock có thể hữu ích cho các hàm đệ quy:

Ví dụ:
```python
import threading

l = threading.RLock()
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

### f. Threading Objects
Có một vài kiểu Threading. Bao gồm:
- Semaphore
- Timer
- Barrier

(Search GG thêm khi cần)

#### Reference 
[Realpython](https://realpython.com/intro-to-python-threading/)
[Viblo](https://viblo.asia/p/lap-trinh-da-luong-cac-co-che-dong-bo-trong-python-OEqGj6bQG9bL)
[lock-vs-semaphore-in-python](https://superfastpython.com/lock-vs-semaphore-in-python/)
[thread-reentrant-lock](https://superfastpython.com/thread-reentrant-lock/)