## III. Processes trong Python.
### 1. Khởi tạo 
Dưới đây là cú pháp khởi tạo process:
```python
from multiprocessing import Process

process_1 = Process(target=my_task, args=(arg1, arg2), processes=5)

process_1.start()

process_1.join()
```
Trong đó: 
- **my_task**: tên function.
- **args**: các tham số truyền vào args.
- **processes**: số task thực thi song song.


Ngoài ra:
- Hàm _start()_ để chạy các process đó.
- Hàm _join()_ khi được gọi sẽ chấm dứt các process gọi đến nó thay vì để nó tiếp tục nhàn rỗi.



