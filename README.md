## Tìm hiểu về Apache JMeter

### 1. Apache Jmeter là gì?

- Ứng dụng `Apache JMeter` là một phần mềm mã nguồn mở, được viết 100% bằng Java, dùng để kiểm tra độ chịu tải (`load test`) và đo lường hiệu suất (`measure performance`).

- Ban đầu nó được thiết kế để kiểm thử các ứng dụng web nhưng sau đó được mở rộng để kiểm thử các chức năng khác.

### 2. Công dụng

- `Apache JMeter` có thể được sử dụng để kiểm tra hiệu suất cả trên các tài nguyên tĩnh, tài nguyên động và các ứng dụng web động.

- Nó có thể được được dùng để mô phỏng tải nặng (`heave load`) trên một server, của nhóm server, mạng hoặc đối tượng để kiểm tra cường độ, để phân tích hiệu suất tổng thể dưới nhiều loại tải khác nhau.

### 3. Tính năng

- Có khả năng tải và kiểm tra hiệu suất trên nhiều loại `ứng dụng`/`server`/`protocol` khác nhau:
    + Web - HTTP, HTTPS (Java, NodeJS, PHP, ASP.NET, …)
    + SOAP / REST Webservices
    + FTP
    + Database thông qua JDBC
    + LDAP
    + Middleware hướng thông điệp (MOM) thông qua JMS
    + Mail - SMTP(S), POP3(S) và IMAP(S)
    + Các native command hoặc shell script
    + TCP
    + Java Objects

- Đầy đủ tính năng `Test IDE` mà cho phép `Test Plan` ghi (`recording`), xây dựng (`building`), gỡ lỗi (`debugging`) nhanh chóng (từ các trình duyệt hoặc các ứng dụng native).

- Chế độ dòng lệnh (`command-line`) để kiểm tra tải từ bất cứ hệ điều hành tương thích Java nào (Linux, Windows, Max OSX, ...)

- Báo cáo HTML động được trình bày 1 cách hoàn chỉnh và sẵn sàng

- Dễ dàng tương tác thông qua khả năng trích xuất dữ liệu từ các định dạng phản hồi phổ biến nhất như HTML, JSON, XML hoặc bất cứ định dạng văn bản nào

- Linh động và 100% là Java

- Framework đa luồng (`multi-threading`) đầy đủ cho phép lấy mẫu đồng thời bởi nhiều luồng và lấy mẫu cùng lúc của các function khác nhau bởi các nhóm luồng riêng biệt.

- Caching và phân tích (`analysis`) / phát lại (`replaying`) kết quả kiểm tra ngoại tuyến.

- Lõi có khả năng mở rộng cao

### 4. Jmeter làm việc như thế nào?

[![Diagram](https://gyazo.com/4b5aeb4c738207cb38dbe3e4bb9c11a0/raw)](https://gyazo.com/4b5aeb4c738207cb38dbe3e4bb9c11a0)

### 5. Cài đặt Jmeter cho Ubuntu 16.04

- Cập nhật hệ thống bởi command dưới đây:

    ```
    sudo apt-get update
    ```

- Cài đặt Jmeter

    ```
    sudo apt-get install jmeter
    ```

- Giao diện sau khi cài đặt thành công:

[![Start](https://gyazo.com/3de795edcff472123183127b9ce2da42/raw)](https://gyazo.com/3de795edcff472123183127b9ce2da42)

### 6. Một số khái niệm cơ bản

#### a. Test plan

- Một `test plan` có thể được xem như một container để chạy test. Nó xác định cần test cái gì và làm thế nào để test.

- Một `test plan` hoàn chỉnh bao gồm một hoặc nhiều thành phần như các nhóm luồng (`thread groups`), controller logic (`logic controllers`), controller tạo mẫu (`sample-generating controllers`), trình lắng nghe (`listeners`), bộ hẹn giờ (`timers`), các xác nhận (`assertions`), và các phần tử cấu hình (`configuration elements`).

- Một `test plan` phải có ít nhất một `thread group`.

- Xem lại ảnh giao diện khi cài đặt thành công Jmeter, chúng ta thấy có 2 node được tạo sẵn: `Test Plan` và `Workbench`:

[![Start](https://gyazo.com/3de795edcff472123183127b9ce2da42/raw)](https://gyazo.com/3de795edcff472123183127b9ce2da42)

- `Test Plan node`: Nơi thực sự chứa `test plan`.

- `Workbench node`: Nó chỉ đơn giản cung cấp một nơi để tạm thời để lưu trữ các phần tử test trong khi không sử dụng, dùng cho các mục đích copy/paste. Khi bạn lưu `test plan`, các `Workbench item` sẽ không được lưu cùng với nó.

#### b. Thread Group

- Các phần tử `Thread Group` là các điểm bắt đầu của `test plan`. Chúng điều khiển số lượng thread mà `JMeter` sẽ sử dụng trong suốt quá trình test.

- Ngoài ra, ta cũng có thể điều khiển những mục sau thông qua `Thread Group`:

    + `Number of Threads (users)`: Thiết lập số lượng thread
    + `Ramp-Up Period (in seconds)`: Cho biết thời gian để JMeter tạo ra tất cả những thread cần thiết.
    + `Loop Count`: Số lần thưc hiện chạy test.

#### c. Controllers

- `JMeter` có 2 loại Controller: `Samplers` và `Logic Controllers`.

- `Samplers` cho phép `JMeter` gửi các loại request cụ thể tới một server. Chúng mô phỏng một user request cho một page từ target server.

- Danh sách `Samplers` được JMeter cung cấp:

    + HTTP Request
    + FTP Request
    + JDBC Request
    + Java Request
    + SOAP/XML Request
    + RPC Requests

- `Logic Controllers` cho phép bạn kiểm soát thứ tự xử lý của `Samplers` trong một `Thread`. `Logic Controller` có thể thay đổi thứ tự của một request đến từ bất kỳ phần tử con nào của chúng.

- Danh sách `Logic Controllers` được JMeter cung cấp:

    + Simple Controller
    + Loop Controller
    + Once Only Controller
    + Interleave Controller
    + Random Controller
    + Random Order Controller
    + Throughput Controller
    + Runtime Controller
    + If Controller
    + While Controller
    + Switch Controller
    + ForEach Controller
    + Module Controller
    + Include Controller
    + Transaction Controller
    + Recording Controller

#### d. Listeners

- `Listeners` cho phép bạn xem kết quả của `Samplers` dưới dạng table, graph, tree hoặc text đơn giản trong một số file log. Chúng cung cấp khả năng truy cập trực quan đến dữ liệu được thu thập bởi `JMeter` về các test case như là một `Sampler component` của `JMeter` được thực thi.

- Danh sách `Listeners` được JMeter cung cấp:

    + Sample Result Save Configuration
    + Graph Full Results
    + Graph Results
    + Spline Visualizer
    + Assertion Results
    + View Results Tree
    + Aggregate Report
    + View Results in Table
    + Simple Data Writer
    + Monitor Results
    + Distribution Graph (alpha)
    + Aggregate Graph
    + Mailer Visualizer
    + BeanShell Listener
    + Summary Report

#### e. Timers

- Theo mặc định, một `JMeter thread` gửi các request mà không cần tạm dừng giữa mỗi sampler. Điều này có thể không phải là thứ bạn muốn. Bạn có thể thêm một `timer element` cho phép bạn xác định khoảng thời gian chờ giữa mỗi request.

- Danh sách `Timers` được JMeter cung cấp:

    + Constant Timer
    + Gaussian Random Timer
    + Uniform Random Timer
    + Constant Throughput Timer
    + Synchronizing Timer
    + JSR223 Time
    + BeanShell Time
    + BSF Time
    + Poisson Random Time

#### f. Assertions

- `Assertions` cho phép bạn bao gồm một số validation test trong response của request được thực hiện bằng cách sử dụng một `Sampler`. Sử dụng assertion giúp bạn có thể chứng minh rằng ứng dụng đang trả lại dữ liệu chính xác. `JMeter` sẽ highlight khi có một assertion thất bại.

- Danh sách `Assertions` được JMeter cung cấp:

    + Beanshell Assertion
    + BSF Assertion
    + Compare Assertion
    + JSR223 Assertion
    + Response Assertion
    + Duration Assertion
    + Size Assertion
    + XML Assertion
    + BeanShell Assertion
    + MD5Hex Assertion
    + HTML Assertion
    + XPath Assertion
    + XML Schema Assertion

#### g. Configuration Elements

-  `Configuration Elements` cho phép bạn tạo các giá trị mặc định và các biến được sử dụng bởi `Samplers`. Chúng được sử dụng để thêm hoặc sửa đổi các request được thực hiện bởi `Samplers`.

- Danh sách `Configuration Elements` được JMeter cung cấp:

    + Counter
    + CSV Data Set Config
    + FTP Request Defaults
    + HTTP Authorization Manager
    + HTTP Cache Manager
    + HTTP Cookie Manager
    + HTTP Proxy Server
    + HTTP Request Defaults
    + HTTP Header Manager
    + Java Request Defaults
    + Keystore Configuration
    + JDBC Connection Configuration
    + Login Config Element
    + LDAP Request Defaults
    + LDAP Extended Request Defaults
    + TCP Sampler Config
    + User Defined Variables
    + Simple Config Element
    + Random Variable


### 7. Ví dụ về web test plan

#### a. Bước 1: Tạo Thread Group

-  Click chuột phải vào `Test Plan` node ở bên trái màn hình

-  Chọn `Add` -> `Thread (Users)` -> `Thread Group`

- Thiết lập các thuộc tính của `Thread`: Giả sử:
    + `Number of Threads`: 1000
    + `Ramp-Up Period`: 0
    + `Loop Count`: 1

[![Thread group](https://gyazo.com/bbb4b041950da261f389d64777d165c8/raw)](https://gyazo.com/bbb4b041950da261f389d64777d165c8)

#### b. Bước 2: Tạo Sampler HTTP Request

- Click chuột phải vào `Thread Group` vừa tạo ở bước 1

- Chọn `Add` -> `Sampler` -> `HTTP Request`

- Có thể đổi tên nếu bạn muốn ở mục `Name`

- Nhập `Server Name`: laravel.com

- Nhập `Protocol`: https

- Chọn `Method`: GET

- Nhập `Path`: /

[![Http request](https://gyazo.com/8fa757e45bb82993ff90af08bf7fca8c/raw)](https://gyazo.com/8fa757e45bb82993ff90af08bf7fca8c)

#### c. Bước 3: Thêm các Listeners

- Click chuột phải vào `HTTP Request` vừa tạo ở bước 2

- Chọn `Add` -> `Listener` -> `Summary Report`

- Chọn `Add` -> `Listener` -> `View Results Tree`

- Chọn `Add` -> `Listener` -> `Graph Results`

- Click vào button `Start` ở thanh menu để chạy test

- Kết quả test:

[![Summary report](https://gyazo.com/32c0b13aad25956e81ff886db0debe13/raw)](https://gyazo.com/32c0b13aad25956e81ff886db0debe13)

[![View result tree](https://gyazo.com/e88b98b6087acc9ce2c8c5b1abb3ef1a/raw)](https://gyazo.com/e88b98b6087acc9ce2c8c5b1abb3ef1a)

[![Graph result](https://gyazo.com/4a640e98560be243d8e6669fa7a1f2ec/raw)](https://gyazo.com/4a640e98560be243d8e6669fa7a1f2ec)

### Tài liệu tham khảo

1. [Apache JMeter™](https://jmeter.apache.org/)
2. [jMeter - Quick Guide](http://www.tutorialspoint.com/jmeter/jmeter_quick_guide.htm)