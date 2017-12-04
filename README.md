## Laravel tap()

### 1. Giới thiệu

- Phải đến phiên bản 5.5 mới có tài liệu cụ thể, tuy nhiên bắt đầu từ phiên bản 5.3, Laravel đã lặng lẽ thêm vào 1 helper function khá hữu ích với tên gọi là `tap`. Cái tên này được lấy cảm hứng từ Ruby, giúp nâng cao khả năng khai báo cho framework.

### 2. Cấu trúc

    <?php
    function tap($value, $callback)
    {
        $callback($value);

        return $value;
    }

- Function tap() chấp nhận 2 đối số: 1 đối số `$value` tùy ý và 1 `Closure`. Đối số `$value` sẽ được truyền vào `Closure` và sau đó được trả về bởi function `tap`.

### 3. Sử dụng như thế nào?

- Giờ hãy xem Laravel Framework sử dụng function `tap()` như thế nào, thông qua ví dụ từ Eloquent:

    ```
    <?php
    public function create(array $attributes = [])
    {
        return tap($this->newModelInstance($attributes), function ($instance) {
            $instance->save();
        });
    }
    ```

- Các bước thực hiện của function trên là:
    + Tạo 1 Eloquent model mới
    + Lưu model
    + Trả về model instance

- `tap()` helper cho phép chúng ta làm điều đó chỉ với 1 khối code, bởi vì nó luôn trả về model instance mà chúng ta cung cấp.

- Trong phiên bản gần đây của Laravel 5.4, function `tap()` đã có diện mạo thu gọn hơn, không bao gồm `Closure`:

    ```
    <?php
    return tap($user)->update([
        'name' => $name,
        'age' => $age,
    ]);
    ```

- Nếu `Closure` không được truyền vào function `tap`, bạn có thể gọi bất cứ method nào sẵn có của đối số `$value`.
- Giá trị trả lại của method với function `tap` được gọi sẽ luôn là $value, bất kể method đó thực tế được định nghĩa như thế nào. Như ở ví dụ trên, hàm update thông thường trả về dữ liệu kiểu boolean, tuy nhiên, khi sử dụng `tap()`, nó sẽ trả về User model.
- Tính năng này của `tap` giúp nó dễ dàng bắt buộc bất kỳ method nào của 1 object phải trả về chính object đó.

### 4. Chức năng

***_a. Thực hiện các hành động trung gian_***

- Code thông thường:

    ```
    $response = $next($request);
     
    $this->storePasswordHashInSession($request);
     
    return $response;
    ```

- Code sử dụng `tap()`:

    ```
    return tap($next($request), function () use ($request) {
        $this->storePasswordHashInSession($request);
    });
    ```

- Pattern:

    ```
    // Gán object
    // Sử dụng object (hoặc có thể không)
    // Trả về object
    ```

***_b. Khôi phục lại trạng thái_***

- Một số option được định nghĩa trong file config (`vendor/barryvdh/laravel-dompdf/config/dompdf.php`)

    ```
    // Hàm này được định nghĩa trong App\Model mà App\Message extends.
    public function keepingTimestamps(callable $callback)
    {
        try {
            $timestamps = $this->timestamps;
            $this->timestamps = false;
     
            return tap($this, $callback);
        } finally {
            $this->timestamps = $timestamps;
        }
    }
    ```

    ``` 
    request()->user()->latestMessage->keepingTimestamps(function ($message) {
        $message->markRead(); // thay vì update `read_at` timestamp
    }); // trả về latestMessage
     
    ```

- Pattern:

    ```
    // Thu thập trạng thái
    // Sử dụng object
    // Khôi phục trạng thái
    // Trả về object đó
    ```

***_c. Collection_***

    # Item
    # $items = [
    #    ['name' => 'David Charleston', 'member' => 1, 'active' => 1],
    #    ['name' => 'Blain Charleston', 'member' => 0, 'active' => 0],
    #    ['name' => 'Megan Tarash', 'member' => 1, 'active' => 1],
    #    ['name' => 'Jonathan Phaedrus', 'member' => 1, 'active' => 1],
    #    ['name' => 'Paul Jackson', 'member' => 0, 'active' => 1]
    # ];

    return collect($items)
        ->where('active', 1)
        ->tap(function($collection){
            return var_dump($collection->pluck('name'));
        })
        ->where('member', 1)
        ->tap(function($collection){
            return var_dump($collection->pluck('name'));
        });

    # Results
    # First tap: David Charleston, Megan Tarash, Jonathan Phaedrus, Paul Jackson
    # Second tap: David Charleston, Megan Tarash, Jonathan Phaedrus

***_d. Collection tap() với việc xử lý console command_***

    public function handle()
    {
        Club::findOrFail($this->option('club'))
            ->members()
            ->subscribed()
            ->get()
            ->tap(function ($members) {
                $this->output->progressStart($members->count());
            })
            ->each(function ($member) {
                Mail::to($member)->queue(new Newsletter($member, $this->matchReport());
     
                $this->output->progressAdvance();
            })
            ->tap(function () {
                $this->output->progressFinish();
            });
    }
     
    public function matchReport()
    {
         return once(function () {
              return MatchReport::ofRound($this->option('round'))->firstOrFail();
         });
    }

***_e. Laravel Query Builder tap() macro_***

    public function handle()
    {
        Club::findOrFail($this->option('club'))
            ->players()
            ->active()
            ->with([
                'awards',
                'seasons.matches',
                'teams',
            ])
            ->tap(function ($query) {
                $this->output->progressStart(with(clone $query)->reorder()->count());
            })
            ->chunk(100, function ($players) {
                $players->each(function ($player) {
                    with(new PerformanceReport($player))->save();
                });
     
                $this->output->progressAdvance();
            })
            ->tap(function () {
                $this->output->progressFinish();
            });
    }

### 5. Tài liệu tham khảo

1. [Tap, Tap, Tap]( https://medium.com/@taylorotwell/tap-tap-tap-1fc6fc1f93a6 )
2. [tap()]( https://laravel.com/docs/5.5/helpers#method-tap )
3. [Laravel tap()]( http://derekmd.com/2017/02/laravel-tap/ )
4. [Laravel Collection “tap” Method]( https://laravel-news.com/collection-tap )
