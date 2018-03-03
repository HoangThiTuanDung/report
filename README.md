## 5 Laravel helpers hữu ích giúp việc phát triển hiệu quả hơn

### 1. data_get()

- `data_get()` helper cho phép bạn nhận giá trị từ 1 array hoặc object bằng dấu chấm.

- Cấu trúc: `data_get($target, $key, $default = null)`.

- Trong đó,
    + `$target`: Mảng hoặc object đầu vào.
    + `$key`: Key cần tìm sử dụng dấu chấm.
    + `$default`: Đối số tùy chọn, sử dụng để cung cấp giá trị mặc định nếu key không được tìm thấy.

    ```
    $array = ['albums' => ['rock' => ['count' => 75]]];

    $count = data_get($array, 'albums.rock.count'); // 75
    $avgCost = data_get($array, 'albums.rock.avg_cost', 0); // 0

    $object->albums->rock->count = 75;

    $count = data_get($object, 'albums.rock.count'); // 75
    $avgCost = data_get($object, 'albums.rock.avg_cost', 0); // 0
    ```

- Nếu bạn sử dụng `ký tự đại diện (*)` giữa 2 dấu chấm, Laravel sẽ trả về  1 mảng các kết quả.

    ```
    $array = ['albums' => ['rock' => ['count' => 75], 'punk' => ['count' => 12]]];
    $counts = data_get($array, 'albums.*.count'); // [75, 12]
    ```

### 2. str_plural()

- `str_plural()` helper giúp chuyển 1 chuỗi thành dạng số nhiều. Hiện tại chỉ có tiếng Anh được hỗ trợ.

- Cấu trúc: `function str_plural($value, $count = 2)`.

- Trong đó,
    + `$value`: Chuỗi đầu vào.
    + `$count`: Đối số tùy chọn để  helper trả về dạng số nhiều hoặc số ít. Mặc định 2 là số nhiều, 1 là số ít.


- Helper này cũng có thể chuyển đổi đối với những từ "không thể đếm được" hoặc những từ trong trường hợp đặc biệt.

    ```
    str_plural('dog'); // dogs
    str_plural('cat'); // cats

    str_plural('dog', 2); // dogs
    str_plural('cat', 1); // cat

    str_plural('child'); // children
    str_plural('person'); // people
    str_plural('fish'); // fish
    str_plural('deer', 2); // deer
    ```

### 3. route()

- `route()` helper tạo ra 1 URL từ 1 route name cụ thể.

- Cấu trúc: `route($name, $parameters = [], $absolute = true)`.

- Trong đó:
    + `$name`: route name.
    + `$parameters`: Đối số tùy chọn, chứa các tham số bổ sung.
    + `$absolute`: ối số tùy chọn, giúp helper xác định trả về URL tương đối hay URL tuyệt đối.

    ```
    Route::get('burgers', 'BurgersController@index')->name('burgers');
    route('burgers'); // http://example.com/burgers
    route('burgers', ['order_by' => 'price']); // http://example.com/burgers?order_by=price

    Route::get('burgers/{id}', 'BurgersController@show')->name('burgers.show');
    route('burgers.show', 1); // http://example.com/burgers/1
    route('burgers.show', ['id' => 1]); // http://example.com/burgers/1

    Route::get('employees/{id}/{name}', 'EmployeesController@show')->name('employees.show');
    route('employees.show', [5, 'chris']); // http://example.com/employees/5/chris
    route('employees.show', ['id' => 5, 'name' => 'chris']); // http://example.com/employees/5/chris
    route('employees.show', ['id' => 5, 'name' => 'chris', 'hide' => 'email']); // http://example.com/employees/5/chris?hide=email
    ```

- Nếu bạn truyền false cho đối số thứ 3, helper sẽ trả về 1 URL tương đối thay vì URL tuyệt đối.

    ```
    route('burgers.show', 1, false); // /burgers/1
    ```

- Route cho sub-domain làm việc đơn giản như thêm thuộc tính khác.

    ```
    Route::domain('{location}.example.com')->group(function () {
        Route::get('employees/{id}/{name}', 'EmployeesController@show')->name('employees.show');
    });

    route('employees.show', ['location' => 'raleigh', 'id' => 5, 'name' => 'chris']); // http://raleigh.example.com/employees/5/chris
    ```

- Bạn có thể pass Eloquent model trực tiếp vào route().

    ```
    route('burgers.show', Burger::find(1)); // http://example.com/burgers/1
    ```

- Sở dĩ có thể pass Eloquent model vào route() như vậy bởi vì class base Model implement UrlRoutable interface. Hành vi mặc định sử dụng khóa chính của model nhưng bạn có thể ghi đè nó bằng cách thêm method `getRouteKeyName()`.

    ```
    class Burger extends Model
    {
        public function getRouteKeyName()
        {
            return 'slug';
        }
    }
    ```

- Sau đó sử dụng như sau:

    ```
    Route::get('burgers/{slug}', 'BurgersController@show')->name('burgers.show');

    route('burgers.show', Burger::find(1)); // http://example.com/burgers/everyones-favorite-burger
    ```

### 4. abort_if()

- `abort_if()` helper throw 1 exception nếu biểu thức đã cho là đúng.

- Cấu trúc: `abort_if($boolean, $code, $message = '', array $headers = [])`.

- Trong đó,
    + `$boolean`: Biểu thức trả về kết quả true hoặc false.
    + `$code`: Mã exception.
    + `$message`: Đối số tùy chọn, cho phép hiển thị message tùy chỉnh cho exception.
    + `$headers`: Đối số tùy chọn, cho phép truyền 1 mảng data cho header.

    ```
    abort_if(! Auth::user()->isAdmin(), 403);
    abort_if(! Auth::user()->isAdmin(), 403, 'Sorry, you are not an admin');
    abort_if(Auth::user()->isCustomer(), 403);
    ```

- Nhiều người trong chúng ta làm như ví dụ dưới đây mà không biết `abort_if()` helper có thể rút ngắn lại còn 1 dòng.

    ```
    // In "admin" specific controller
    public function index()
    {
        if (! Auth::user()->isAdmin()) {
            abort(403, 'Sorry, you are not an admin');
        }
    }

    // better!
    public function index()
    {
        abort_if(! Auth::user()->isAdmin(), 403);
    }
    ```

### 5. optional()

- `optional()` helper cho phép bạn truy cập các thuộc tính hoặc gọi các method của 1 object. Nếu object chỉ định là null, các thuộc tính và method sẽ trả về null thay vì sinh ra lỗi.

- Cấu trúc: `optional($value = null)`

- Trong đó, `$value` là tên thuộc tính hoặc tên method của object.

    ```
    // User 1 exists, with account
    $user1 = User::find(1);
    $accountId = $user1->account->id; // 123

    // User 2 exists, without account
    $user2 = User::find(2);
    $accountId = $user2->account->id; // PHP Error: Trying to get property of non-object

    // Fix without optional()
    $accountId = $user2->account ? $user2->account->id : null; // null
    $accountId = $user2->account->id ?? null; // null

    // Fix with optional()
    $accountId = optional($user2->account)->id; // null
    ```

- `optional()` helper là lý tưởng khi sử dụng các object mà bạn có thể không sở hữu hoặc gọi dữ liệu lồng nhau trong Eloquent relationship mà có thể có, có thể không.

### Tài liệu tham khảo

1. [5 Laravel Helpers to Make Your Life Easier](https://laravel-news.com/5-laravel-helpers-make-life-easier)
