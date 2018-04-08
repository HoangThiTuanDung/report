## Các kỹ thuật eloquent search cơ bản

### 1. Giới thiệu

- Tìm kiếm (search) là một phần quan trọng của bất kỳ ứng dụng nào.

- Tìm kiếm là một hoạt động hoặc một kỹ thuật giúp tìm ra vị trí của một phần tử hoặc giá trị nhất định trong danh sách.

- Bất kỳ tìm kiếm nào được cho là thành công hay không thành công tùy thuộc vào việc tìm ra đối tượng đang được tìm kiếm hay không.

- Và việc tìm kiếm với Eloquent được thực hiện thông qua các mệnh đề `where`.

### 2. Các mệnh đề `where` đơn giản

- Tìm kiếm các post có title là `foo` hết sức đơn giản bằng cách sử dụng mệnh đề `where` để so sánh các cột với giá trị đã cho:

    ```
    <?php

    $results = Post::where('title', 'foo')->get();
    ```

- Phức tạp hơn 1 chút, chúng ta có thể truyền vào 1 mảng để check nhiều cột. Thao tác này giống với toán tử `&&`. Chúng ta nhận được các bản ghi với các điều kiện đều thỏa mãn:

    ```
    <?php

    $results = Post::where([
        ['title', '=', 'foo'],
        ['published', '=', true],
    ])->get();
    ```

- Ngoài ra, chúng ta có thể xây dựng các truy vấn hoạt động giống với toán tử `||`. Tất cả những gì chúng ta phải làm là sử dụng phương thức `orWhere()` trên query builder. Kết quả trả về chúng ta nhận được các `post` với `tilte` hoặc `description` có giá trị là `foo`:

    ```
    <?php

    $results = Post::where('title', 'foo')->orWhere('description', 'foo')->get();
    ```

### 3. Sử dụng keyword `like`

- Với mệnh đề where thông thường, chúng ta chỉ có thể tìm được những bản ghi có giá trị chính xác với giá trị đã cho.

- Việc sử dụng thêm keyword sẽ giúp chúng ta nhận được các kết quả linh hoạt hơn. Điển hình với keyword `like` và ký tự `%`:

    ```
    <?php

    // Giá trị đã cho
    $keyword = 'foo';

    // Bắt đầu với 'foo', kết thúc bằng gì cũng được.
    // Ví dụ: foo, food, foobar, ...
    $results = Post::where('title', 'like', "{$keyword}%")->get();

    // Kết thúc 'foo', bắt đầu bằng gì cũng được
    // Ví dụ: foo, mafoo, ...
    $results = Post::where('title', 'like', "%{$keyword}")->get();

    // Chứa 'foo'
    // Ví dụ: foo, flatfooting, ...
    $results = Post::where('title', 'like', "%{$keyword}%")->get();
    ```

- Lưu ý: Nếu bạn đang sử dụng 1 collation kết thúc bằng `_ci` (ví dụ `utf8_general_ci`). Điều đó có nghĩa là nó không phân biệt chữ hoa chữ thường. Cho dù bạn nhập FOO, Foo, fOO, ... thì kết quả nhận được đều giống nhau.

### 4. Tìm kiếm trong JSON column

- Json type mang lại sự linh hoạt rất lớn. Hơn nữa Eloquent hỗ trợ rất tốt Json type nên ta có thể dễ dàng truy vấn chúng.

- Lưu ý là các Json column có phân biệt chữ hoa, chữ thường.

- Thông thường thì ta có thể viết đơn giản như sau:

    ```
    <?php

    $results = Post::where('meta->description', 'like', '%foo%')->get();
    ```

- Tuy nhiên, vì phân biệt chữ hoa chữ thường nên phải viết phức tạp hơn 1 chút:

    ```
    <?php

    $keyword = 'foo';
    $results = Post::whereRaw('lower(meta->"$.description") like lower(?)', ["%{$foo}%"]);
    ```

- Giải thích đơn giản cho cách viết trên là chuyển hết về định dạng chữ thường rồi so sánh.

### 5. Sử dụng keyword `sounds like`

- Khi bạn muốn search 1 từ khóa nhưng lại không nhớ chính xác từ xóa đó là gì thì `sounds like` là sự lựa chọn hoàn hảo.

    ```
    <?php

    $results = Post::where('title', 'sounds like', 'mistpyed')->get();
    ```

- Kết quả nhận được có sự tương đồng với từ khóa đã cho, chứ không chỉ bao gồm những kết quả có hoặc chứa chính xác từ khóa đó.


### Tài liệu tham khảo

1. [Basic Eloquent Search Techniques](https://pineco.de/basic-eloquent-search-techniques/)
