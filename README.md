## Laravel soft delete

### I. Soft delete

#### 1. Giới thiệu

- Như ta đã biết, Laravel có 2 kiểu xóa dữ liệu cho model – đó là `Force delete` và `Soft delete`.

- `Force delete` thì quá quen thuộc rồi, hôm nay chúng ta sẽ tìm hiểu chi tiết hơn 1 chút về `Soft delete` nhé.

#### 2. Định nghĩa

- Khi model bị soft delete, nó không thực sự bị xóa khỏi database, chỉ có giá trị trường `deleted_at` được update.

- Nếu model có trường `deleted_at` khác null thì nó đã bị soft delete.

#### 3. Cách sử dụng

- Để enable tính năng soft delete cho model, bạn hãy sử dụng `Illuminate\Database\Eloquent\SoftDeletes` trait cho model và thêm trường `deleted_at` vào thuộc tính `$date`:

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}

```

- Tất nhiên, bạn nên thêm trường `deleted_at` vào database sử dụng Laravel helper:

```
Schema::table('flights', function ($table) {
    $table->softDeletes();
});
```

- Bây giờ, nếu bạn gọi phương thức `delete` model thì trường `deleted_at` sẽ được cập nhật thành thời gian hiện tại. Và khi bạn thực hiện truy vấn model sử dụng soft delete, các model đã bị soft delete sẽ bị loại bỏ khỏi tất cả các kết quả truy vấn.

- Để xác định xem model đã bị soft delete hay chưa thì bạn có thể sử dụng phương thức `trashed`:

```
if ($flight->trashed()) {
    //
}
```

#### 4. Truy vấn các model đã bị soft delete

- Truy vấn toàn bộ các bản ghi, bao gồm cả các model đã bị soft delete với phương thức `withTrashed()`:

```
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```

- Truy vấn chỉ những model đã bị soft delete với phương thức `onlyTrashed()`:

```
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

- Ta có thể khôi phục được các model đã bị soft delete với phương thức `restore()`:

```
$flight->restore();
```

- Ta cũng có thể kết hợp với force delete để xóa vĩnh viễn model với phương thức `forceDelete()`:

```
// Force deleting a single model instance...
$flight->forceDelete();

// Force deleting all related models...
$flight->history()->forceDelete();
```

### II. Soft delete model con khi model cha bị xóa

- Giả sử ta có relationship như sau:

```
User (has many) Posts (has many) Comments
```

- Việc soft delete 1 model là rất đơn giản trong Laravel, tuy nhiên làm sao để soft delete 1 model mà có relationship với các model khác?

#### 1. Sử dụng funtion boot

- Nếu bạn setup cascade ‘onDelete’ đến khóa ngoại, khi user bị xóa thì tất cả các post của user đó cũng bị xóa. 

- Tuy nhiên, soft delete không bị ảnh hưởng bởi cascade nữa, vì vậy, chúng ta phải định nghĩa 1 sự kiện để xóa model con nếu model cha bị xóa sử dụng funtion `boot` của model.

- Với User model:

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes;

    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
    */
    
    protected $dates = ['deleted_at'];
  
    /**
    * Override parent boot and Call deleting event
    *
    * @return void
    */
    
    protected static function boot() 
    {
        parent::boot();

        static::deleting(function($users) {
            foreach ($users->posts()->get() as $post) {
                $post->delete();
            }
        });
    }
   
    /**
    * Define relationship with post model
    *
    * @return \Illuminate\Database\Eloquent\Relations\HasMany
    */

    public function posts()
    {
        $this->hasMany('App\Post');
    }
}
```

- Với Post model:

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;

    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
    */
    
    protected $dates = ['deleted_at'];
   
    /**
    * Override parent boot and Call deleting event
    *
    * @return void
    */
    
    protected static function boot() 
    {
        parent::boot();

        static::deleting(function($posts) {
            foreach ($posts->comments()->get() as $comment) {
                $comment->delete();
            }
        });
    }
   
    /**
    * Define relationship with Comment model
    *
    * @return \Illuminate\Database\Eloquent\Relations\HasMany
    */
    
    public function comments()
    {
        $this->hasMany('App\Comment');
    }
}
```

#### 2. Sử dụng package

- Có 1 cách đơn giản và dễ dàng hơn, đó là sử dụng package [michaeldyrynda/laravel-cascade-soft-deletes](https://github.com/michaeldyrynda/laravel-cascade-soft-deletes):

a. Cài đặt:

```
composer require iatstuti/laravel-cascade-soft-deletes
``` 

b. Cách sử dụng:

- User model:

```
<?php

namespace App;

use App\Post;
use Iatstuti\Database\Support\CascadeSoftDeletes;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes, CascadeSoftDeletes;

    protected $cascadeDeletes = [posts];

    protected $dates = ['deleted_at'];

    public function posts()
    {
        return $this->hasMany(Comment::class);
    }
}    

```

- Post model:

```
<?php

namespace App;

use App\Comment;
use Iatstuti\Database\Support\CascadeSoftDeletes;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes, CascadeSoftDeletes;

    protected $cascadeDeletes = ['comments'];

    protected $dates = ['deleted_at'];

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}    
```

- Bây giờ khi bạn soft delele User model thì Post model, Comment model có liên quan sẽ bị soft delete theo.

- Lưu ý: Khi khôi phục model cha, các model con có liên quan sẽ không được khôi phục lại.

### III. Tài liệu tham khảo

1. [Soft Deleting](https://laravel.com/docs/5.5/eloquent#soft-deleting)
2. [Soft Deleting Parent & Child Models in Eloquent Laravel 5](https://medium.com/teknomuslim/soft-deleting-parent-child-models-in-eloquent-laravel-5-dc05a29133bf)
3. [Cascading soft deletes for the Laravel PHP Framework](https://github.com/michaeldyrynda/laravel-cascade-soft-deletes)
