# Multiprocessing Pool vs ProcessPoolExecutor in Python
Python hỗ trợ 2 kiểu pool đó là Pool trong **Python multiprocessing Pool** và **concurrent.futures.ProcessPoolExecutor** class.

Trong bài hướng dẫn này, chúng ta sẽ tìm hiểu sự giống nhau và khác nhau của 2 loại này. Điều này sẽ giúp chúng ta đưa ra được những quyết định đúng đắn khi sử dụng.

## 1. Python multiprocessing Pool
Chúng ta có thể gọi pool thôn qua multiprocessing.Pool.
```python
from multiprocessing import Pool
```
Python multiprocessing Pool cho phép các task được submit như là các functions để các process pool thực thi. Một process pool là một programming pattern giúp cho việc quản lý các nhóm worker processes một các hiệu quả. 

Định nghĩa:
> A process pool object which controls a pool of worker processes to which jobs can be submitted. It supports asynchronous results with timeouts and callbacks and has a parallel map implementation.

Chúng ta có thể chỉ định số lượng worker cần khởi tạo thông qua một argument trong constructor.

```python
from multiprocessing import Pool
# Tạo 5 worker
pool = Pool(processes=5)
```

Chúng ta có thể sử dụng __map()__ để khiến việc sử dụng pool trở lên dễ dàng hơn.

```python
from multiprocessing import Pool

import time

work = (["A", 5], ["B", 2], ["C", 1], ["D", 3])


def work_log(work_data):
    print(" Process %s waiting %s seconds" % (work_data[0], work_data[1]))
    time.sleep(int(work_data[1]))
    print(" Process %s Finished." % work_data[0])


def pool_handler():
    p = Pool(2)
    p.map(work_log, work)


if __name__ == '__main__':
    pool_handler()


# Kết quả:
# Process A waiting 5 seconds
# Process B waiting 2 seconds
# Process B Finished.
# Process C waiting 1 seconds
# Process C Finished.
# Process D waiting 3 seconds
# Process A Finished.
# Process D Finished.
```
Ngoài ra chúng ta còn có thể sử dụng __apply_async()__, __map_async()__ và __starmap_async()__ cũng cho phép các process thực hiện task một cách bất đồng bộ. Tức là cho phép thực hiện song song nhiều method trên các workers.

```python
import multiprocessing as mp
import time

def _square(x):
  return x*x

def log_result(result):
  # Hàm được gọi bất kỳ khi nào _square(i) trả ra kết quả.
  # result_list được thực hiện trên main process, khong phải pool workers.
  result_list.append(result)

def apply_async_with_callback():
  pool = mp.Pool(processes=5)
  for i in range(20):
    pool.apply_async(_square, args = (i, ), callback = log_result)
  pool.close()
  pool.join()
  print(result_list)

if __name__ == '__main__':
  result_list = []
  apply_async_with_callback()


# Kết quả:
# [0, 1, 9, 25, 16, 36, 49, 64, 81, 100, 144, 4, 169, 225, 256, 289, 324, 361, 196, 121]
```

## 2. ProcessPoolExecutor
Chúng ta có thể gọi ProcessPoolExecutor theo cách sau:
```python
from concurrent.futures import ProcessPoolExecutor
```

Một process là một instance của 1 chương trình máy tính. Mỗi process đều có 1 main thread và có thể có thêm các threads bổ xung. Một tiến trình có thể rẽ nhánh ra thành nhiều tiến trình con. 

- Khởi tạo ProcessPoolExecutor:

```python
from concurrent.futures import ProcessPoolExecutor

executor = ProcessPoolExecutor(max_workers=10)
```
Cũng giống như ThreadPoolExecutor thì chúng ta cũng có thể chỉ định số worker lớn nhất có thể tạo.

Chúng ta cũng có thể __map()__ hoặc __submit()__ giống như cơ chế bên ThreadPoolExecutor.

> :exclamation: Đọc thêm ở link bên dưới. Đại loại khá giống với ThreadPoolExecutor

## 3. Comparison of Pool vs ProcessPoolExecutor
**Giống nhau:**
    - Cả 2 đều khởi tạo bởi hệ điều hành và có thể rẽ nhánh thành các tiến trình con. Điều này có nghĩa là các tác vụ được cấp cho mỗi pool process sẽ thực thi đồng thời và tận dụng tối đa các lõi CPU có sẵn.
    - Cả 2 đều cung cấp các hàm cho phép chạy bất đồng bộ.
    - Cả 2 đều cung cấp cơ chế chờ bằng cách sử dụng hàm __wait()__ tương ứng với 2 loại pool trên.

**Khác nhau:**
    - **ProcessPoolExecutor** có thể huỷ bỏ các tác vụ đã đc cấp, trong khi **multiprocessing.Pool** thì không thể.
    - **ProcessPoolExecutor** cung cấp các công cụ để làm việc với các nhóm tác vụ không đồng bộ, trong khi **multiprocessing.Pool** thì không._(:exclamation: đọc thêm ở link bên dưới. Chưa hiểu lắm)_
    - **multiprocessing.Pool** cung cấp khả năng kết thúc mạnh mẽ tất cả các tác vụ, trong khi **ProcessPoolExecutor** thì không. Lớp **multiprocessing.Pool** cung cấp các hàm **close()** và **term()** sẽ gửi tín hiệu **SIGTERM** và **SIGKILL** tới các quy trình worker con.
    - **multiprocessing.Pool** tập trung vào **map()** đồng thời(Có đến 6 biến thể của map), trong khi **ProcessPoolExecutor** thì không(chỉ có 1 hàm map duy nhất).
    - **ProcessPoolExecutor** cung cấp một cách để truy cập trực tiếp vào các exception được đưa ra trong một tác vụ không đồng bộ, trong khi **multiprocessing.Pool** thì không.

## :facepunch: Nói tóm lại 
- **ProcessPoolExecutor** có thể làm được hầu hết các công việc của **multiprocessing.Pool**. Còn **multiprocessing.Pool** chỉ hơn **ProcessPoolExecutor** vì tập trung nhiều vào thiết kế các biến thể của hàm map.

### Reference
[Multiprocessing Pool vs ProcessPoolExecutor in Python](https://superfastpython.com/multiprocessing-pool-vs-processpoolexecutor/)