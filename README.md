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
### [1. Khởi tạo](/threads/khoi-tao-thread.md)
### [2. ThreadPoolExcutor](/threads/thread-pool-excutor.md)
### [3. Cơ chế bất đồng bộ - asynchronous](/threads/asynchronous.md)
### [4. Cơ chế đồng bộ - synchronous](/threads/synchronous.md)

## III. Processes trong Python.
## Reference:
[Realpython](https://realpython.com/intro-to-python-threading/)

[Digitalocean](https://www.digitalocean.com/community/tutorials/python-multiprocessing-example)

[phamdinhkhanh](https://phamdinhkhanh.github.io/2020/11/30/ParallelComputingPython.html)

[viblo.asia](https://viblo.asia/p/threads-and-processes-in-python-yMnKMzWEZ7P)
