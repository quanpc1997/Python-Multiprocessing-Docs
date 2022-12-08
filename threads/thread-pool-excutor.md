## II. Threads trong python.
### 2. ThreadPoolExcutor
**ThreadPoolExecutor** cho phép bạn khởi tạo và quản lý các nhóm thread một cách tự động.
**ThreadPoolExecutor** class kế thừa từ abstract class **Executor**.

#### a. What Are Futures
A future is an object that represents a delayed result for an asynchronous task.

It is also sometimes called a promise or a delay. It provides a context for the result of a task that may or may not be executing and a way of getting a result once it is available.

In Python, the Future object is returned from an Executor, such as a ThreadPoolExecutor when calling the submit() function to dispatch a task to be executed asynchronously.

In general, we do not create Future objects; we only receive them and we may need to call functions on them.

There is always one Future object for each task sent into the ThreadPoolExecutor via a call to submit().

The Future object provides a number of helpful functions for inspecting the status of the task such as: cancelled(), running(), and done() to determine if the task was cancelled, is currently running, or has finished execution.

cancelled(): Returns True if the task was cancelled before being executed.
running(): Returns True if the task is currently running.
done(): Returns True if the task has completed or was cancelled.
A running task cannot be cancelled and a done task could have been cancelled.

A Future object also provides access to the result of the task via the result() function. If an exception was raised while executing the task, it will be re-raised when calling the result() function or can be accessed via the exception() function.

result(): Access the result from running the task.
exception(): Access any exception raised while running the task.
Both the result() and exception() functions allow a timeout to be specified as an argument, which is the number of seconds to wait for a return value if the task is not yet complete. If the timeout expires, then a TimeoutError will be raised.

Finally, we may want to have the thread pool automatically call a function once the task is completed.

This can be achieved by attaching a callback to the Future object for the task via the add_done_callback() function.

add_done_callback(): Add a callback function to the task to be executed by the thread pool once the task is completed.
We can add more than one callback to each task and they will be executed in the order they were added. If the task has already completed before we add the callback, then the callback is executed immediately.

Any exceptions raised in the callback function will not impact the task or thread pool.

We will take a closer look at the Future object in a later section.

Now that we are familiar with the functionality of a ThreadPoolExecutor provided by the Executor class and of Future objects returned by calling submit(), let’s take a closer look at the lifecycle of the ThreadPoolExecutor class.

#### b. LifeCycle of the ThreadPoolExecutor.
![LifeCycle of the ThreadPoolExecutor](/threads/images/ThreadPoolExecutor-Life-Cycle.webp)

Vòng đời của ThreadPoolExecutor bao gồm 4 bước:


**Bước 1: Khởi tạo thread pool bằng cách gọi contructor** **ThreadPoolExecutor**.
- **ThreadPoolExecutor** instance phải được khởi tạo trước tiên. Sau đó ta phải khai báo số threads trong pool. 
- Pool được khởi tạo với mỗi luồng cho mỗi CPU trong hệ thống. _Thực tế cho thấy số luồng tối ưu của hệ thống sẽ bằng tổng số luồng + 4_.

**=> Default Total Threads = (Total CPUs) + 4**

Ví dụ: Nếu chúng ta có 4 CPU trong 1 hệ thống, Mỗi CPU sẽ tương ứng với 2 luồng, khi đó Python sẽ coi như là hệ thống có 8 luồng + 4 = 12 luồng cho pool theo mặc định. 

Việc chúng ta phải khai báo số lượng luồng nhằm tránh trường hợp có quá nhiều luồng sẽ gây lên những ảnh hưởng đế dung lượng RAM còn trống, gây khó khăn trong việc trao đổi dữ liệu giữa các luồng. Điều này có thể dẫn đến hiệu xuất thấp hơn.

VD:
```python
import concurrent.futures

with concurrent.futures.ThreadPoolExecutor() as executor:
    pass
```

**Bước 2: Submit Tasks to the Thread Pool.**
Có 2 cách tiếp cận chính để submit task lên trên class cha Executor. Đó là __map()__ và __submit()__.

a) Submit tasks với map(). 
```python
# perform all tasks in parallel
results = pool.map(my_task, my_items) # does not block
```
Trong đó _my_task_ là tên của function và _my_items_ là iterable của objects. 

Hàm map() trả về một iterator. Chúng ta có thể thêm timeout (số giây) khi gọi map() thông qua tham số truyền vào _timeout_ để đặt số time chờ cho một task. Nếu vượt qua khoảng thời gian đó thì hệ thống sẽ báo lỗi. Còn nếu không thêm thì task sẽ chạy đến xong thì thôi.

```python
# iterate over results as they become available
for result in executor.map(my_task, my_items, timeout=5):
	print(result) 
```

b) Submit Task with submit().

```python
# submit a task with arguments and get a future object
future = executor.submit(my_task, arg1, arg2) # does not block
```
Trong đó _my_task_ là tên của hàm mà bạn muốn chạy. _arg1_ và _arg2_ là tham số đầu tiên (chỉ là optional) và tham số thứ 2 để truyền vào hàm _my_task_. 

Bạn có thể truy cập vào kết quả của task thông qua hàm __result()__. Kết quả trả về sẽ là 1 đối tượng __Future__. Ví dụ:
```python
# get the result from a future
future = executor.submit(my_task, arg1, arg2)
result = future.result() # blocks
```

Chúng ta có thể đặt timeout khi gọi hàm __result()__ với cách hoạt động giống timeout bên __map()__.
```python
# wait for task to complete or timeout expires
result = future.result(timeout=5) # blocks
```

**Bước 3: Wait for Tasks to Complete(Optional).**
The concurrent.futures module provides two module utility functions for waiting for tasks via their Future objects.

Recall that Future objects are only created when we call submit() to push tasks into the thread pool.

These wait functions are optional to use, as you can wait for results directly after calling _map()_ or _submit()_ or wait for all tasks in the thread pool to finish.

These two module functions are __wait()__ for waiting for Future objects to complete and __as_completed()__ for getting Future objects as their tasks complete.

- __wait()__: Wait on one or more Future objects until they are completed.
- __as_completed()__: Returns Future objects from a collection as they complete their execution.

You can use both functions with Future objects created by one or more thread pools, they are not specific to any given thread pool in your application. This is helpful if you want to perform waiting operations across multiple thread pools that are executing different types of tasks.

Both functions are useful to use with an idiom of dispatching multiple tasks into the thread pool via submit in a list compression; for example:

```python
# dispatch tasks into the thread pool and create a list of futures
futures = [executor.submit(my_task, my_data) for my_data in my_datalist]
```

Here, my_task is our custom target task function, “my_data” is one element of data passed as an argument to “my_task“, and “my_datalist” is our source of my_data objects.

We can then pass the Future objects to wait() or as_completed().

Creating a list of Future objects in this way is not required, just a common pattern when converting for loops into tasks submitted to a thread pool.

__a) Wait for Futures to Complete__
The wait() function can take one or more Future objects and will return when a specified action occurs, such as all tasks completing, one task completing, or one task raising an exception.

The function will return one set of Future objects that match the condition set via the “return_when“. The second set will contain all of the futures for tasks that did not meet the condition. These are called the “done” and the “not_done” sets of futures.

It is useful for waiting on a large batch of work and to stop waiting when we get the first result.

This can be achieved via the FIRST_COMPLETED constant passed to the “return_when” argument.

```python
# wait until we get the first result
done, not_done = wait(futures, return_when=concurrent.futures.FIRST_COMPLETED)
```

Alternatively, we can wait for all tasks to complete via the ALL_COMPLETED constant.

This can be helpful if you are using submit() to dispatch tasks and are looking for an easy way to wait for all work to be completed.

```python wait for all tasks to complete
done, not_done = wait(futures, return_when=concurrent.futures.ALL_COMPLETED)
```

There is also an option to wait for the first exception via the FIRST_EXCEPTION constant.

```python
# wait for the first exception
done, not_done = wait(futures, return_when=concurrent.futures.FIRST_EXCEPTION)
```

__b) Wait for Futures as Completed__
The beauty of performing tasks concurrently is that we can get results as they become available, rather than waiting for all tasks to be completed.

The as_completed() function will return Future objects for tasks as they are completed in the thread pool.

We can call the function and provide it a list of Future objects created by calling submit() and it will return Future objects as they are completed in whatever order.

It is common to use the as_completed() function in a loop over the list of Future objects created when calling submit; for example:

```python
# iterate over all submitted tasks and get results as they are available
for future in as_completed(futures):
	# get the result for the next completed task
	result = future.result() # blocks
```
__Note__: this is different from iterating over the results from calling map() in two ways. Firstly, map() returns an iterator over objects, not over Future objects. Secondly, map() returns results in the order that the tasks were submitted, not in the order that they are completed.

__Bước 4. Shutdown the Thread Pool__
Once all tasks are completed, we can close down the thread pool, which will release each thread and any resources it may hold (e.g. the thread stack space).

```python
# shutdown the thread pool
executor.shutdown() # blocks
```
The __shutdown()__ function will wait for all tasks in the thread pool to complete before returning by default.

This behavior can be changed by setting the “wait” argument to False when calling shutdown(), in which case the function will return immediately. The resources used by thread pool will not be released until all current and queued tasks are completed.

```python
# shutdown the thread pool
executor.shutdown(wait=False) # does not blocks
```
We can also instruct the pool to cancel all queued tasks to prevent their execution. This can be achieved by setting the “cancel_futures” argument to True. By default queued tasks are not cancelled when calling shutdown().

```python
# cancel all queued tasks
executor.shutdown(cancel_futures=True) # blocks
```
If we forget to close the thread pool, the thread pool will be closed automatically when we exit the main thread. If we forget to close the pool and there are still tasks executing, the main thread will not exit until all tasks in the pool and all queued tasks have executed.

#### ThreadPoolExecutor Context Manager
A preferred way to work with the ThreadPoolExecutor class is to use a context manager.

This matches the preferred way to work with other resources, such as files and sockets.

Using the ThreadPoolExecutor with a context manager involves using the “with” keyword to create a block in which you can use the thread pool to execute tasks and get results.

Once the block has completed, the thread pool is automatically shut down. Internally, the context manager will call the shutdown() function with the default arguments, waiting for all queued and executing tasks to complete before returning and carrying on.

Below is a code snippet to demonstrate creating a thread pool using the context manager.

```python
# create a thread pool
with ThreadPoolExecutor(max_workers=10) as pool:
	# submit tasks and get results
	# ...
	# automatically shutdown the thread pool...
# the pool is shutdown at this point
```

This is a very handy idiom if you are converting a for loop to be multithreaded.

It is less useful if you want the thread pool to operate in the background while you perform other work in the main thread of your program, or if you wish to reuse the thread pool multiple times throughout your program.

Now that we are familiar with how to use the ThreadPoolExecutor, let’s look at some worked examples.

#### Reference
[ThreadPoolExecutor](https://superfastpython.com/threadpoolexecutor-in-python/)