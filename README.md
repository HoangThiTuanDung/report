## Sinh file PDF với Laravel

### 1. Giới thiệu

***a. Khái niệm về định dạng PDF***

- PDF (`Portable Document Format`) là định dạng tài liệu di động, tập tin văn bản khá phổ biến của hãng Adobe Systems. Cũng giống như định dạng Word, PDF hỗ trợ văn bản thô cùng với font chữ, hình ảnh, âm thanh và nhiều hiệu ứng khác. Một văn bản PDF sẽ được hiển thị giống nhau trên những môi trường làm việc khác nhau. Đây cũng chính là ưu điểm vượt trội mà PDF sở hữu khiến cho định dạng này trở nên phổ biến cho việc phát hành sách, báo hoặc tài liệu khác qua mạng Internet.

***b. Ưu điểm của định dạng PDF***

- Nội dung trình bày đa dạng cùng với khả năng bảo mật cực kỳ tốt.
- Bạn có thể in ra trên bất cứ thiết bị nào mà vẫn giữ nguyên được định dạng.
- Hỗ trợ trên hầu hết các loại thiết bị di động.
- Tài liệu dạng PDF thường có kích thước khá nhỏ khiến cho việc di chuyển, chia sẻ rất dễ dàng.

Chính nhờ những ưu điểm như trên mà việc xuất file PDF cũng được ứng dụng rất nhiều trong lập trình code, ví dụ như xuất các file hợp đồng, CV tuyển dụng, ...

Riêng Laravel có package hỗ trợ rất hữu ích cho việc xuất file PDF, đó là `Laravel dompdf`. Chúng ta cùng tìm hiểu về nó nhé.

### 2. Cài đặt với Laravel 5

```
composer require barryvdh/laravel-dompdf
```

- Thêm provider và alias trong file config/app:

```
'providers' => [
    ...
    Barryvdh\DomPDF\ServiceProvider::class,
],
```

```
'aliases' => [
    ...
    'PDF' => Barryvdh\DomPDF\Facade::class,
],
```

### 3. Các option có sẵn và giá trị default của chúng

- Một số option được định nghĩa trong file config (`vendor/barryvdh/laravel-dompdf/config/dompdf.php`)

```
<?php

return [
    // Throw an Exception on warnings from dompdf
    'show_warnings' => false,
    
    // The default orientation
    'orientation' => 'portrait',
    
    'defines' => [
        // The location of the DOMPDF font directory
        "DOMPDF_FONT_DIR" => storage_path('fonts/'),
        
        // The location of the DOMPDF font cache directory
        "DOMPDF_FONT_CACHE" => storage_path('fonts/'),
        
        // The location of a temporary directory
        "DOMPDF_TEMP_DIR" => sys_get_temp_dir(),
        
        // Prevents dompdf from accessing system files or other files on the webserver
        "DOMPDF_CHROOT" => realpath(base_path()),

        // Whether to use Unicode fonts or not
        "DOMPDF_UNICODE_ENABLED" => true,

        // Whether to enable font subsetting or not
        "DOMPDF_ENABLE_FONT_SUBSETTING" => false,

        // The PDF rendering backend to use
        "DOMPDF_PDF_BACKEND" => "CPDF",

        // Html target media view which should be rendered into pdf
        "DOMPDF_DEFAULT_MEDIA_TYPE" => "screen",

        // The default paper size
        "DOMPDF_DEFAULT_PAPER_SIZE" => "a4",

        // The default font family
        "DOMPDF_DEFAULT_FONT" => "serif",

        // Image DPI setting (In pdf, 1pt = 1/72 inch)
        "DOMPDF_DPI" => 96,

        // Enable inline PHP
        "DOMPDF_ENABLE_PHP" => false,

        // Enable inline Javascript
        "DOMPDF_ENABLE_JAVASCRIPT" => true,

        // Enable remote file access
        "DOMPDF_ENABLE_REMOTE" => true,

        // A ratio applied to the fonts height to be more like browsers' line height
        "DOMPDF_FONT_HEIGHT_RATIO" => 1.1,

        // Enable CSS float
        "DOMPDF_ENABLE_CSS_FLOAT" => false,

        // Use the more-than-experimental HTML5 Lib parser
        "DOMPDF_ENABLE_HTML5PARSER" => false,
    ],
];
```

- Nếu muốn thay đổi giá trị của các option mặc định thì bạn có thể dùng command 

```
php artisan vendor:publish 
```

rồi thay đổi các giá trị mình cần trong file `config/dompdf.php`. 

- Một số option khác:

```
+ rootDir: "{app_directory}/vendor/dompdf/dompdf"
+ logOutputFile: "/tmp/log.htm"
+ debugPng: false
+ debugLayout: false
+ debugKeepTemp: false
+ debugCss: false
+ debugLayoutLines: true
+ debugLayoutPaddingBox: true
+ debugLayoutBlocks: true
+ debugLayoutInline: true
+ pdflibLicense: ""
+ adminPassword: "password"
+ adminUsername: "user"
```

### 4. Một số function hỗ trợ

```
// Get the DomPDF instance
public function getDomPDF()

// Set the paper size (default A4)
public function setPaper($paper, $orientation = 'portrait')

// Show or hide warnings
public function setWarnings($warnings)

// Load a HTML string
public function loadHTML($string, $encoding = null)

// Load a HTML file
public function loadFile($file)

// Load a View and convert to HTML
public function loadView($view, $data = [], $mergeData = [], $encoding = null)

// Set/Change an option in DomPdf
public function setOptions(array $options)

// Output the PDF as a string
public function output()

// Save the PDF to a file
public function save($filename)

// Make the PDF downloadable by the user
public function download($filename = 'document.pdf')

// Return a response with the PDF to show in the browser
public function stream($filename = 'document.pdf')

// Render the PDF
protected function render()
```

### 5. Cách sử dụng

- Bạn có thể tạo 1 instance DOMPDF mới và load chuỗi HTML, file hoặc tên view. 
- Bạn có thể lưu 1 file hoặc  hiển thị trên trình duyệt (stream) hoặc download nó.

```
$pdf = App::make('dompdf.wrapper');
$pdf->loadHTML('<h1>Test</h1>');

return $pdf->stream();
```

- Hoặc sử dụng facade:

```
$pdf = PDF::loadView('pdf.invoice', $data);

return $pdf->download('invoice.pdf');
```

- Hỗ trợ UTF-8 (thiết lập trong template):

```
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
```

- Sang trang mới (sử dụng thuộc tính CSS page-break-before/page-break-after)

```
<style>
.page-break {
    page-break-after: always;
}
</style>

<h1>Page 1</h1>
<div class="page-break"></div>
<h1>Page 2</h1>
```

### 6. Demo

***Stream***
- Sample Code:

```
class DemoController extends Controller
{
    public function streamPDF()
    {
        $pdf = App::make('dompdf.wrapper');
        $pdf->loadHTML('<h1>Demo stream</h1>');

        return $pdf->stream();
    }
}
```

- Result:

[![Stream](https://gyazo.com/ffe4317b7e9c3d73a9acd6305467d38a/raw)](https://gyazo.com/ffe4317b7e9c3d73a9acd6305467d38a)

***Download***

- Sample code:
```
class DemoController extends Controller
{
    public function downloadPDF()
    {
        $pdf = PDF::loadView('demo');

        return $pdf->download('demo.pdf');
    }
}
```

```
// demo.blade.php
<html>
<body>

<form action="#">
    First name:<br><input type="text" name="firstname" value=""><br>
    Last name:<br><input type="text" name="lastname" value=""><br><br>
    <input type="submit" value="Submit">
</form> 

</body>
</html>
```

- Result: File demo.pdf sẽ được lưu trong thư mục Downloads

### 7. Tài liệu tham khảo

1. [DOMPDF Wrapper for Laravel 5]( https://github.com/barryvdh/laravel-dompdf )
2. [PDF]( https://vi.wikipedia.org/wiki/PDF )
3. [File PDF là gì? Đọc file PDF như thế nào?]( https://quantrimang.com/tim-hieu-ve-dinh-dang-pdf-118417 )
