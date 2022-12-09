## Chia sẻ dữ liệu trong Process
Có 2 cách thông dụng để chia sẻ dữ liệu giữa các Process với nhau:
### Cách 1. Shared memory - Chia sẻ dữ liệu sử dụng Value và Array
Việc chia sẻ dữ liệu giữa các Process sẽ không giống với cách thông thường.
Ví dụ: 
```python
from multiprocessing import Process, Lock
import time

def _counter_arr(arrs, process_name):
  lock.acquire()
  for i, el in enumerate(arrs):
    time.sleep(0.01)
    arrs[i] = -arrs[i]
    print("{}: {}".format(process_name, arrs[i]))
  lock.release()

arrs = [1, 2, 3, 4]
lock = Lock()
exec1 = Process(target=_counter_arr, args=(arrs, "khanh process")) # pass counter and thread_name into method _counter
exec2 = Process(target=_counter_arr, args=(arrs, "ai process"))
execs = [exec1, exec2]

exec1.start()
exec2.start()

for exec in execs:
  exec.join()

# Kết quả:
# khanh process: -1
# khanh process: -2
# khanh process: -3
# khanh process: -4
# ai process: -1
# ai process: -2
# ai process: -3
# ai process: -4

```

Như ta đã thấy dữ liệu không bị thay đổi. Bây giờ ta chỉ cần thay đổi cách khai báo từ arrs = [1, 2, 3, 4] thành arrs = Array('i', range(1, 5, 1)).

```python
from multiprocessing import Process, Value, Array, Lock
import time

def _counter_arr(arrs, process_name):
  lock.acquire()
  for i, el in enumerate(arrs):
    time.sleep(0.01)
    arrs[i] = -arrs[i]
    print("{}: {}".format(process_name, arrs[i]))
  lock.release()

arrs = Array('i', range(1, 5, 1))
lock = Lock()
exec1 = Process(target=_counter_arr, args=(arrs, "khanh process")) # pass counter and thread_name into method _counter
exec2 = Process(target=_counter_arr, args=(arrs, "ai process"))
execs = [exec1, exec2]

exec1.start()
exec2.start()

for exec in execs:
  exec.join()

# Kết quả:
# khanh process: -1
# khanh process: -2
# khanh process: -3
# khanh process: -4
# ai process: 1
# ai process: 2
# ai process: 3
# ai process: 4
```
Dữ liệu đã được chia sẻ qua lại giữa hai processes.

### Cách 2. Server process
Khác với Share Memory chỉ hỗ trợ Array và Value thì ở đây đối tượng Manager sẽ hỗ trợ nhiều kiểu hơn như là __list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, và cả Value lẫn Array__.

```python
# Ví dụ này chỉ minh hoạ cách gọi ra các kiểu dữ liệu được hỗ trợ cho việc share dữ liệu
from multiprocessing import Process, Manager

def f(d, l):
    d[1] = '1'
    d['2'] = 2
    d[0.25] = None
    l.reverse()

if __name__ == '__main__':
    with Manager() as manager:
        d = manager.dict()
        l = manager.list(range(10))

        p = Process(target=f, args=(d, l))
        p.start()
        p.join()

        print(d)
        print(l)
```

#### Customized managers
Như chúng ta thấy với các kiểu dữ liệu không có sẵn thì nếu chỉ dùng Manager() mặc định thì sẽ không thể đáp ứng được bài toán. Do đó ta cần tuỳ chỉnh lại.

Để tạo Manager của riêng mình, người ta tạo một lớp con của BaseManager và sử dụng phương thức lớp register() để đăng ký các loại. Ví dụ:

```python
from multiprocessing.managers import BaseManager

class MathsClass:
    def add(self, x, y):
        return x + y
    def mul(self, x, y):
        return x * y

class MyManager(BaseManager):
    pass

MyManager.register('Maths', MathsClass)

if __name__ == '__main__':
    with MyManager() as manager:
        maths = manager.Maths()
        print(maths.add(4, 3))         # prints 7
        print(maths.mul(7, 8))         # prints 56
```


Ví dụ 2:
```python
# SuperFastPython.com
# example of sharing a python object using a manager
from time import sleep
from random import random
from multiprocessing import Process
from multiprocessing.managers import BaseManager
 
# custom manager to support custom classes
class CustomManager(BaseManager):
    # nothing
    pass
 
# custom function to be executed in a child process
def task(number, shared_set):
    # generate a number
    value = random()
    # block for a moment
    sleep(value)
    # store the result
    shared_set.add((number,value))
 
# protect the entry point
if __name__ == '__main__':
    # register the counter with the custom manager
    CustomManager.register('set', set)
    # create a new manager instance
    with CustomManager() as manager:
        # create a shared set instance
        shared_set = manager.set()
        # start some child processes
        processes = [Process(target=task, args=(i,shared_set)) for i in range(50)]
        # start processes
        for process in processes:
            process.start()
        # wait for processes to finish
        for process in processes:
            process.join()
        # all done
        print('Done')
        # report the results
        print(len(shared_set._getvalue()))
        print(shared_set)
```
## Reference
[Python Documents](https://python.readthedocs.io/en/latest/library/multiprocessing.html)

[Multiprocessing Manager to Share an Object with Processes](https://superfastpython.com/multiprocessing-share-object-with-processes/)