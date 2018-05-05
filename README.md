## Đi sâu vào middleware trong Laravel

### I. Giới thiệu

#### 1. Laravel middleware là gì?

- Là 1 tính năng trong Laravel mà cung cấp 1 cơ chế để lọc các HTTP request vào ứng dụng của bạn.

- Nó cho phép bạn để nối vào luồng xử lý Laravel request để thực hiện 1 số logic quyết định cách ứng dụng của bạn thực hiện.

#### 2. Bạn sử dụng middleware để làm gì?

- Bảo vệ route.

- Thiết lập header trong các phản hồi HTTP.

- Logging các request đến ứng dụng của bạn.

- Bật chế độ bảo trì toàn bộ trang web.

- Điều khiển các phản hồi được sinh ra bởi ứng dụng của bạn.

### II. Làm sao để tạo 1 custom middleware?

#### 1. Tạo 1 custom middleware

- Tạo 1 middleware trong Laravel rất đơn giản bằng cách chạy dòng lệnh sau:

```
php artisan make:middleware <MiddlewareName>
```

- Nó sẽ tạo ra 1 middleware với tên được chỉ định nằm trong thư mục middleware của ứng dụng.

- Thật may mắn, Laravel cung cấp những thứ cơ bản cần thiết để bắt đầu custom middleware của bạn.

```

<?php

namespace App\Http\Middleware;

use Closure;

class RedirectIfSuperAdmin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return $next($request);
    }
}
```

- Lưu ý rằng function handle() chấp nhận 2 tham số là `$request` và `$next`.

- Tham số `$request` giữ incoming request URI trong ứng dụng của bạn, trong khi tham số `$next` được sử dụng để chuyển request sâu hơn vào ứng dụng.

- Logic cần thiết được viết trong function handle() và đưa chúng ta đến các loại middleware - `before middleware` và `after middleware`.

- `Before middleware` sẽ thực hiện 1 vài tác vụ trước khi chuyển tiếp request sâu hơn vào ứng dụng.

```
<?php

namespace App\Http\Middleware;

use Closure;

class RedirectIfSuperAdmin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        // Viết logic ở đây
        return $next($request);
    }
}
```

- Mặt khác, `after middleware` chạy sau khi request đã được ứng dụng xử lý và phản hồi được xây dựng.

```
<?php

namespace App\Http\Middleware;

use Closure;

class RedirectIfSuperAdmin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $response = $next($request);
        // Viết logic ở đây

        return $response;
    }
}

```

#### 2. Các danh mục middleware

- `Global middleware` chạy cho mọi single request mà truy cập ứng dụng. Laravel sẽ chạy với hầu hết các middleware trung gian như: ValidatePostSize, TrimStrings,CheckForMaintenanceMode, ...

- `Route middleware` chỉ chạy trên những route mà chúng được gắn vào. Ví dụ: redirectIfAuthenticated.

#### 3. Đăng ký 1 middleware

- Bất cứ middleware nào được tạo phải được đăng ký vì đó là cách duy nhất mà Laravel biết là nó tồn tại.

- Để đăng ký 1 middleware, chỉ cần mở file có tên là `Kernel.php` nằm trong thư mục Http.

- File này chứa tất cả các middleware đã được đăng ký mặc định với Laravel. Nó chứa 3 mảng chính bao gồm: `$middleware` , `$middlewareGroups` và `$routeMiddleware`.

```
<?php

namespace App\Http;

use Illuminate\Foundation\Http\Kernel as HttpKernel;

class Kernel extends HttpKernel
{
    /**
     * The application's global HTTP middleware stack.
     *
     * These middleware are run during every request to your application.
     *
     * @var array
     */
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
    ];

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
        'api' => [
            'throttle:60,1',
            'bindings',
        ],
    ];

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        // Middleware này mới được tạo
        'superadmin' => \App\Http\Middleware\RedirectIfSuperAdmin::class,
    ];
}
```

- Mảng `$middleware` giữ các `global middleware` chạy cho mọi single HTTP request đến ứng dụng, vì vậy nếu bạn muốn có 1 middleware để chạy với toàn bộ request thì đăng ký ở đây.

- `$middlewareGroup` giúp có thể đăng ký các middleware theo group, dễ dàng đính kèm nhiều middleware đến 1 route bằng cách sử dụng group name.

- Mảng `$routeMiddleware` giữ các middleware đã đăng ký riêng lẻ.

#### 4. Gán 1 middleware

Khi 1 middleware được đăng ký, nó có thể được gắn vào 1 route bằng 2 cách chính sau đây:

*_a. Thông qua hàm khởi tạo trong controller:_*

- Việc gán middleware thông qua hàm khởi tạo trong controller sẽ linh hoạt hơn vì nó cung cấp 2 hàm quan trọng là `except($parameters)` và `only($parameters)` để ngăn chặn hoặc cho phép middleware áp dụng cho 1 số helper function trong controller đó.

- Nếu không sử dụng các helper function, middleware sẽ áp dụng cho mọi single function trong controller đó.

```
<?php

use Illuminate\Http\Request;

class ForumController extends Controller
{

    public function __construct()
    {
        // Middleware được áp dụng cho mọi single function bên trong controller này
        $this->middleware('auth');
    }

    public function viewForum()
    {
        return view('index');
    }

    public function edit($id)
    {
    }

    public function delete($id)
    {
    }

}
```

- Với `except` và `only` function, chúng ta có thể chọn những hàm nào mà middleware sẽ áp dụng:

```
<?php
use Illuminate\Http\Request;

class ForumController extends Controller
{

    public function __construct()
    {
        /** Middleware này được áp dụng cho tất cả các function ngoại trừ viewForums() và viewForumDetails()
        * ngược lại nếu sử dụng only()
        */
        $this->middleware('auth')->except(['viewForums', 'viewForumDetails']);
    }

    public function viewForums()
    {
        return view('index');
    }

    public function edit($id)
    {
    }

    public function delete($id)
    {
    }

    public function viewForumDetails()
    {
    }
}

```

*_b. Middleware được gán thông qua route_*

- 1 middleware đã được đăng ký có thể được gắn trực tiếp vào route như dưới đây:

```
<?php

// Cách 1
Route::get('admin/profile', function () {
  //action
})->middleware('auth');

// Cách 2: Sử dụng tên class
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    // action
})->middleware(CheckAge::class);

// Cách 3
Route::group(['middleware' => ['web']], function () {
    //action
});
```

- Lưu ý: `Middleware group` có thể được gán vào route giống như `single middleware`.

#### 5. Các tham số middleware

- Các tham số bổ sung có thể được truyền vào 1 middleware. Một ví dụ điển hình là trường hợp mỗi user id được gán 1 role và middleware sẽ check role của 1 user để xác định xem họ có quyền truy cập vào URI được yêu cầu hay không. Các tham số có thể được truyền vào middleware theo cách sau đây:

```
<?php

// Cách 1 (Thông qua route)
Route::get('admin/profile', function () {
  //action
})->middleware('auth:<role>');

// Cách 2 (Thông qua controller)
use Illuminate\Http\Request;

class ForumController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:<role>');
    }
}
```

- Nhiều tham số có thể được truyền vào middleware bằng cách tách từng tham số bằng dấu phẩy.

```
<?php
Route::get('admin/profile', function () {
  //action
})->middleware('auth:<role>,<age>,<country>');

```

- Các tham số này được truyền vào `handle()` function của middleware sau biến `$next`.

```
<?php

class RedirectIfSuperAdmin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next, $role, $age, $country)
    {
        // Logic cho middleware sử dụng các tham số được truyền vào
        return $next($request);
    }
}

```

### III. Tổng kết

Để tạo 1 middleware, bạn có thể thông qua quá trình dưới đây:

- Tạo 1 middleware với artisan command:

```
php artisan make:middleware <Middleware Name>
```

- Đăng ký middleware trong file `Kernel.php` ở thư mục `app/Http`.

- Viết logic của bạn trong middleware được viết.

- Gán middleware vào route hoặc controller.

### IV. Tài liệu tham khảo

1. [Deep dive into middlewares in Laravel](https://dev.to/franko4don/deep-dive-into-middlewares-in-laravel-doo)
