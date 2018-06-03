## Laravel-Translatable

### 1. Giới thiệu

- Nếu bạn muốn lưu trữ các bản dịch trong các model của mình ở database thì package này là dành cho bạn.

- Đây là 1 Laravel package dành cho các model có thể dịch được (translatable models). Mục tiêu của nó là loại bỏ sự phức tạp trong việc trích xuất và lưu trữ các instance model đa ngôn ngữ.

- Với package này, bạn sẽ viết ít code hơn vì các bản dịch là được lưu trữ khi bạn lưu trữ instance.

### 2. Demo

- Lấy các thuộc tính đã được dịch:

    ```
    $greece = Country::where('code', 'gr')->first();
    echo $greece->translate('en')->name; // Greece

    App::setLocale('en');
    echo $greece->name;     // Greece

    App::setLocale('de');
    echo $greece->name;     // Griechenland
    ```

- Lưu trữ các thuộc tính đã được dịch:

    ```
    $greece = Country::where('code', 'gr')->first();
    echo $greece->translate('en')->name; // Greece

    $greece->translate('en')->name = 'abc';
    $greece->save();

    $greece = Country::where('code', 'gr')->first();
    echo $greece->translate('en')->name; // abc
    ```

- Fill nhiều bản dịch:

```
    $data = [
        'code' => 'gr',
        'en'  => ['name' => 'Greece'],
        'fr'  => ['name' => 'Grèce'],
    ];

    $greece = Country::create($data);

    echo $greece->translate('fr')->name; // Grèce
```

### 3. Sự tương thích với Laravel

 Laravel  | Translatable
:---------|:----------
 5.6      | 9.*
 5.5      | 8.*
 5.4      | 7.*
 5.3      | 6.*
 5.2      | 5.5 - 6.*
 5.1      | 5.0 - 6.*
 5.0      | 5.0 - 5.4
 4.2.x    | 4.4.x
 4.1.x    | 4.4.x
 4.0.x    | 4.3.x

### 4. Cài đặt

#### a. Bước 1: Install package

- Thêm package vào file `composer.json` bằng việc execute command sau:

    ```
    composer require dimsav/laravel-translatable
    ```

- Sau đó, thêm dòng sau vào phần service provider trong file `app/config/app.php`:

    ```
    Dimsav\Translatable\TranslatableServiceProvider::class,
    ```

#### b. Bước 2: Migartion

    - Trong ví dụ này, mục tiêu là dịch model `Country`, chúng ta sẽ cần phải migration bảng `countries` và bảng mở rộng `country_translations`:

    ```
    Schema::create('countries', function(Blueprint $table)
    {
        $table->increments('id');
        $table->string('code');
        $table->timestamps();
    });

    Schema::create('country_translations', function(Blueprint $table)
    {
        $table->increments('id');
        $table->integer('country_id')->unsigned();
        $table->string('name');
        $table->string('locale')->index();

        $table->unique(['country_id','locale']);
        $table->foreign('country_id')->references('id')->on('countries')->onDelete('cascade');
    });
    ```

#### c. Bước 3: Model

- Model có thể dịch `Country` nên use trait `Dimsav\Translatable\Translatable`.

- Convention cho model dịch là `CountryTranslation`.

    ```
    // models/Country.php
    class Country extends Eloquent {

        use \Dimsav\Translatable\Translatable;

        public $translatedAttributes = ['name'];
        protected $fillable = ['code'];
    }

    // models/CountryTranslation.php
    class CountryTranslation extends Eloquent {

        public $timestamps = false;
        protected $fillable = ['name'];
    }
    ```

- Mảng `$translatedAttributes` chứa tên các trường mà được dịch trong `Translation` model.

#### d. Bước 4: Config

- Copy file config vào project:

    Laravel 5.*

    ```
    php artisan vendor:publish --tag=translatable
    ```

    Laravel 4.*

    ```
    php artisan config:publish dimsav/laravel-translatable
    ```

### 5. Cấu hình

#### a. File cấu hình `laravel-translatable/src/config/translatable.php`

```
<?php
return [
    /*
    |--------------------------------------------------------------------------
    | Các ngôn ngữ của ứng dụng
    |--------------------------------------------------------------------------
    |
    | Chứa một mảng với các ngôn ngữ có sẵn của ứng dụng.
    |
    */
    'locales' => [
        'en',
        'fr',
        'es' => [
            'MX', // mexican spanish
            'CO', // colombian spanish
        ],
    ],
    /*
    |--------------------------------------------------------------------------
    | Phân tách ngôn ngữ
    |--------------------------------------------------------------------------
    |
    | Đây là một chuỗi được sử dụng để gắn ngôn ngữ và quốc gia khi định nghĩa
    | các ngôn ngữ có sẵn. Ví dụ: nếu gía trị của nó là '-', thì ngôn ngữ cho
    | tiếng Colombian Spanish sẽ được lưu dưới dạng 'es-CO' vào database.
    |
    */
    'locale_separator' => '-',
    /*
    |--------------------------------------------------------------------------
    | Ngôn ngữ mặc định
    |--------------------------------------------------------------------------
    |
    | Nó là ngôn ngữ mặc định, Translatable lấy ngôn ngữ trình dịch của Laravel.
    | Nếu vì một số lý do bạn muốn ghi đè nó, bạn có thể chỉ định giá trị mặc
    | định nên được sử dụng ở đây.
    | Nếu bạn thiết lập giá trị ở đây, nó sẽ chỉ sử dụng giá trị cấu hình
    | hiện tại và không có dự phòng cho bản dịch.
    |
    */
    'locale' => null,
    /*
    |--------------------------------------------------------------------------
    | Sử dụng dự phòng
    |--------------------------------------------------------------------------
    |
    | Xác định xem các ngôn ngữ dự phòng có được trả về bởi mặc định hay không.
    | Giá trị này sẽ bị ghi đè bởi thuộc tính $useTranslationFallback nếu
    | được định nghĩa.
    |
    */
    'use_fallback' => false,
    /*
    |--------------------------------------------------------------------------
    | Sử dụng dự phòng cho mỗi thuộc tính
    |--------------------------------------------------------------------------
    |
    | Tính năng này của thuộc tính dự phòng sẽ trả về giá trị đã dịch của ngôn
    | ngữ dự phòng nếu thuộc tính là rỗng cho ngôn ngữ đã chọn.
    | Lưu ý rằng 'use_fallback' phải được enable.
    |
     */
    'use_property_fallback' => true,
    /*
    |--------------------------------------------------------------------------
    | Ngôn ngữ dự phòng
    |--------------------------------------------------------------------------
    |
    | Một ngôn ngữ dự phòng là ngôn ngữ được sử dụng để trả về bản dịch khi bản
    | dịch được yêu cầu không tồn tại. Để disable thì hãy thiết lập nó thành
    | false.
    |
    */
    'fallback_locale' => 'en',
    /*
    |--------------------------------------------------------------------------
    | Hậu tố dịch
    |--------------------------------------------------------------------------
    |
    | Định nghĩa hậu tố 'Translation' class mặc định. Ví dụ, nếu bạn muốn sử
    | dụng CountryTrans thay vì CountryTranslation application, hãy thiết lập
    | nó thành 'Trans'.
    |
    */
    'translation_suffix' => 'Translation',
    /*
    |--------------------------------------------------------------------------
    | Khóa bản dịch
    |--------------------------------------------------------------------------
    |
    | Định nghĩa tên field 'locale', được sử dụng bởi model dịch.
    |
    */
    'locale_key' => 'locale',
    /*
    |--------------------------------------------------------------------------
    | Luôn tải bản dịch khi chuyển đổi thành mảng
    |--------------------------------------------------------------------------
    | Thiết lập giá trị này là false sẽ cải thiện hiệu suất nhưng sẽ không trả
    | lại bản dịch khi sử dụng hàm toArray(), trừ khi translations relationship
    | đã được tải.
    |
     */
    'to_array_always_loads_translations' => true,
];
```

#### b. Model dịch

- Convention được sử dụng để định nghĩa class của model dịch là thêm từ khóa `Translation`.

- Nếu model của bạn là `\MyApp\Models\Country`, bản dịch mặc định sẽ là `\MyApp\Models\CountryTranslation`.

-  Sử dụng custom class làm model dịch:

    ```
    <?php

    namespace MyApp\Models;

    use Dimsav\Translatable\Translatable;
    use Illuminate\Database\Eloquent\Model as Eloquent;

    class Country extends Eloquent
    {
        use Translatable;

        public $translationModel = 'MyApp\Models\CountryAwesomeTranslation';
    }
    ```

### 6. Feature list

#### a. Các method khả dụng

- Cách xác định ngôn ngữ mặc định:

    ```
    App::getLocale(); // 'fr'
    ```

- Ví dụ về model;

    ```
    $germany = Country::where('code', 'de')->first();
    ```

- Trả về một instance của `CountryTranslation` sử dụng ngôn ngữ mặc định. Trong trường hợp này là tiếng Pháp. Nếu không có bản dịch tiếng Pháp nào được tìm thấy, nó sẽ return null.

    ```
    $translation = $germany->translate();
    ```

- Có thể định nghĩa ngôn ngữ mặc định cho mỗi model bằng cách ghi đè hàm constructor của model.

    ```
    public function __construct(array $attributes = [])
    {
        parent::__construct($attributes);

        $this->defaultLocale = 'de';
    }

    ```

- Hoặc có thể viết như sau:

    ```
    $germany->setDefaultLocale('de');
    ```

- Nếu bản dịch tiếng Đức tồn tại, nó trả về một instance của `CountryTranslation`. Ngược lại trả về null.

    ```
    $translation = $germany->translate('de');
    ```

- Nếu bản dịch tiếng Đức không tồn tại, nó sẽ cố lấy bản dịch của ngôn ngữ dự phòng.

    ```
    $translation = $germany->translate('de', true);
    ```

- Alias của phần bên trên.

    ```
    $translation = $germany->translateOrDefault('de');
    ```

- Trả về instance của `CountryTranslation` sử dụng ngôn ngữ mặc định. Nếu không tìm thấy bản dịch nào, nó sẽ trả về bản dịch dự phòng nếu được enabled trong cấu hình.

    ```
    $translation = $germany->getTranslation();
    ```

- Nếu bản dịch tiếng Đức tồn tại, nó trả về một instance của `CountryTranslation`. Ngược lại trả về null.

    ```
    // Giống như $germany->translate('de');

    $translation = $germany->getTranslation('de', true);
    ```

- Để thiết lập bản dịch cho 1 field, bạn có thể cập nhật model dịch. Việc lưu model cũng sẽ lưu tất cả các bản dịch liên quan.

    ```
    $germany->translate('en')->name = 'Germany';
    $germany->save();
    ```

- Ngoài ra chúng ta có thể sử dụng shortcut:

    ```
    $germany->{'name:en'} = 'Germany';
    $germany->save();
    ```

- Có 2 cách để chèn nhiều bản dịch vào database

    ```
    // Cách 1: Sử dụng ngôn ngữ làm array key.
    $greece = $country->fill([
        'en'  => ['name' => 'Greece'],
        'fr'  => ['name' => 'Grèce'],
    ]);

    // Cách 2: Sử dụng cú pháp sau:
    $greece = $country->fill([
        'name:en' => 'Greece',
        'name:fr' => 'Grèce',
    ]);
    ```

- Trả về `true/false` nếu model có bản dịch của ngôn ngữ hiện tại.

    ```
    $germany->hasTranslation();
    ```

- Trả về `true/false` nếu model có bản dịch tiếng Pháp.

    ```
    $germany->hasTranslation('fr');
    ```

- Nếu bản dịch tiếng Đức không tồn tại, nó sẽ trả về một new instance của `CountryTranslation`.

    ```
    $translation = $germany->translateOrNew('de');
    ```

- Trả về một new `CountryTranslation` instance cho ngôn ngữ được chọn và liên kết nó với `$germany`.

    ```
    $translation = $germany->getNewTranslation('it');
    ```

- Eloquent model relationship:

    ```
    $germany->translations();
    ```

- Xóa tất cả các bản dịch được liên kết với một đối tượng.

    ```
    $germany->deleteTranslations();
    ```

- Xóa một hoặc nhiều bản dịch.

    ```
    $germany->deleteTranslations('de');
    $germany->deleteTranslations(['de', 'en']);
    ```

- Nhận tất cả các bản dịch dưới dạng mảng.

    ```
    $germany->getTranslationsArray();

    // Kết quả trả về
    [
        'en' => ['name' => 'Germany'],
        'de' => ['name' => 'Deutschland'],
        'fr' => ['name' => 'Allemagne'],
    ];
    ```

- Tạo bản sao và sao chép các bản dịch.

    ```
    $replicate = $germany->replicateWithTranslations();
    ```

#### b. Các scope khả dụng

- Trả về tất cả các country có bản dịch bằng tiếng Anh.

    ```
    Country::translatedIn('en')->get();
    ```

- Trả về tất cả các country không được dịch bằng tiếng Anh.

    ```
    Country::notTranslatedIn('en')->get();
    ```

- Trả về tất cả các country có bản dịch.

    ```
    Country::translated()->get();
    ```

- Eager load translation relationship chỉ cho ngôn ngữ mặc định và ngôn ngữ dự phòng (nếu được enable).

    ```
    Country::withTranslation()->get();
    ```

- Trả về một mảng chứa các cặp country ids và thuộc tính tên đã được dịch.

    ```
    // Ví dụ:
    // [
    //     ['id' => 1, 'name' => 'Greece'],
    //     ['id' => 2, 'name' => 'Belgium']
    // ]
    Country::listsTranslations('name')->get()->toArray();
    ```

- Lọc các country bằng cách check các bản dịch so với giá trị đã cho.

    ```
    Country::whereTranslation('name', 'Greece')->first();

    // orWhereTranslation
    Country::whereTranslation('name', 'Greece')->orWhereTranslation('name', 'France')->get();
    ```

- Lọc các country bằng cách kiểm tra các bản dịch dựa vào chuỗi đã cho với các ký tự đại diện.

    ```
    Country::whereTranslationLike('name', '%Gree%')->first();

    // orWhereTranslationLike
    Country::whereTranslationLike('name', '%eece%')->orWhereTranslationLike('name', '%ance%')->get();
    ```

#### c. Các thuộc tính magic

- Để sử dụng các thuộc tính magic, bạn phải định nghĩa thuộc tính `$translatedAttributes` ở model chính:

    ```
    class Country extends Eloquent {

        use \Dimsav\Translatable\Translatable;

        public $translatedAttributes = ['name'];
    }

    ```

    ```
    // Bắt đầu bằng 1 instance của country
    $germany = Country::where('code', 'de')->first();

    // Có thể tham chiếu các thuộc tính của đối tượng dịch trực tiếp từ model chính.
    // Nó sử dụng ngôn ngữ mặc định và tương đương với `$germany->translate()->name`
    $germany->name; // 'Germany'

    // Có thể truy cập nhanh bản dịch với ngôn ngữ tùy chỉnh
    $germany->{'name:de'} // 'Deutschland'
    ```

#### d. Ngôn ngữ dự phòng

- Nếu bạn muốn ngôn ngữ dự phòng trở thành ngôn ngữ mặc định khi không tìm thấy bản dịch, hãy enable nó trong cấu hình bằng cách sử dụng key `use_fallback`. Và để chọn ngôn ngữ mặc định, sử dụng key `use_fallback`.

- Cấu hình ví dụ:

    ```
    return [
        'use_fallback' => true,

        'fallback_locale' => 'en',
    ];
    ```

- Bạn cũng có thể định nghĩa việc sử dụng ngôn ngữ dự phòng cho mỗi model, bằng cách thiết lập thuộc tính `$useTranslationFallback`:

    ```
    class Country {

        public $useTranslationFallback = true;

    }
    ```

- Bạn có thể tham khảo thêm về ngôn ngữ dự phòng ở [tài liệu](https://github.com/dimsav/laravel-translatable)

### Tài liệu tham khảo

1. [Laravel-Translatable](https://github.com/dimsav/laravel-translatable)