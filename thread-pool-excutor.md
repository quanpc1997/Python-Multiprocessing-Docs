## II. Threads trong python.
### 2. ThreadPoolExcutor
**ThreadPoolExecutor** cho phép bạn khởi tạo và quản lý các nhóm thread một cách tự động.
**ThreadPoolExecutor** class kế thừa từ abstract class **Executor**.

#### a. LifeCycle of the ThreadPoolExecutor.
![LifeCycle of the ThreadPoolExecutor](/images/ThreadPoolExecutor-Life-Cycle.webp)

Vòng đời của ThreadPoolExecutor bao gồm 4 bước:
\------------------------------------
**Bước 1**: Khởi tạo thread pool bằng cách gọi contructor **ThreadPoolExecutor**.
- **ThreadPoolExecutor** instance phải được khởi tạo trước tiên. Sau đó ta phải khai báo số threads trong pool. 
- Pool được khởi tạo với mỗi luồng cho mỗi CPU trong hệ thống. _Thực tế cho thấy số luồng tối ưu của hệ thống sẽ bằng tổng số luồng + 4_.

**=> Default Total Threads = (Total CPUs) + 4**

Ví dụ: Nếu chúng ta có 4 CPU trong 1 hệ thống, Mỗi CPU sẽ tương ứng với 2 luồng, khi đó Python sẽ coi như là hệ thống có 8 luồng + 4 = 12 luồng cho pool theo mặc định. 

Việc chúng ta phải khai báo số lượng luồng nhằm tránh trường hợp có quá nhiều luồng sẽ gây lên những ảnh hưởng đế dung lượng RAM còn trống, gây khó khăn trong việc trao đổi dữ liệu giữa các luồng. Điều này có thể dẫn đến hiệu xuất thấp hơn.

VD:
```python
import threading
import concurrent.futures
```