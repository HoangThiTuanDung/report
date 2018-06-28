## Convert 1 trang web thành 1 ảnh hoặc file pdf sử dụng chế độ Headless của Chrome

### 1. Giới thiệu

***a. Headless browser là gì?***

- Headless browser là một trình duyệt web không có giao diện đồ họa người  dùng. 
- Các headless browser cung cấp tương tác tự động một trang web trong một môi trường giống như các trình duyệt web phổ biến khác, nhưng nó được thực hiện thông qua giao diện dòng lệnh hoặc qua một mạng truyền thông. Chúng đặc biệt hữu ích cho việc kiểm thử các trang web vì chúng có thể hiển thị và hiểu HTML giống như các trình duyệt thông thường, bao gồm các cả styling như bố cục trang, màu sắc, font chữ và thực thi Javascript và AJAX mà thường không có sẵn khi sử dụng các phương pháp kiểm thử khác.

***b. Headless Chrome là gì?***

- Google Chrome – kể từ phiên bản 59 Chrome hỗ trợ chế độ Headless ở Linux và macOS.
- Chế độ Headless cho phép các chuyên viên kiểm thử phần mềm có thể mô tả các tương tác của Chrome, chỉ load các phần đang hoạt động trong Chrome, engine của Chrome mà không cần khởi động UI. Dễ hiểu hơn chế độ Headless là dùng Chrome mà không cần giao diện người dùng. Các tập lệnh tự động (automated scripts) thực hiện hành động trong chế độ Headless, ghi lại các hành động và báo cáo các vấn đề nó tìm thấy. Kết quả sau đó được sử dụng để đánh giá phần mềm hoặc kiểm tra đoạn code có vấn đề.

***c. Browsershot package***

- Để convert html thành file pdf hoặc ảnh thì người ta hay sử  dụng công cụ [`wkhtmltopdf` và `wkhtmltoimage`](https://wkhtmltopdf.org/). Tuy nhiên những công cụ này lại làm việc với các trình duyệt lỗi thời, trong khi Google đã thêm chế độ headless vào Chrome và công bố 1 thư viện js gọi là [Puppeteer](https://github.com/GoogleChrome/puppeteer) cho phép bạn lập trình và kiểm soát tốt trên Chrome.
- Bạn có thể sử dụng Chrome và Puppeteer để convert html sang định dạng pdf và image. Browsershot chính là gói thư viện mà bạn cần.

### 2. Yêu cầu

***a. Thư viện***

- node >= v7.6.0
- Thư viện Puppeteer Node

```
// ubuntu 16.04
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ sudo apt-get install -y nodejs gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils wget
$ sudo npm install --global --unsafe-perm puppeteer
$ sudo chmod -R o+rx /usr/lib/node_modules/puppeteer/.local-chromium
```

***b. Custom node và npm binaries***

- Browsershot mặc định làm việc với thư mục public trong project. Nếu bạn setup node và npm trong thư mục khác thì khi chạy sẽ báo lỗi:

```
Error Output:
================
module.js:473
      throw err;
      ^

Error: Cannot find module '/home/{xxx}/blog/public/npm'
```

- Do đó, bạn cần phải link đến đường dẫn chính xác bằng cách dùng các method `setNodeBinary` và `setNpmBinary`:

```
Browsershot::html('Foo')
    ->setNodeBinary('/usr/local/bin/node')
    ->setNpmBinary('/usr/local/bin/npm');
```

***c. Custom include path***

- Việc thiết lập đường dẫn include có thể hữu ích trong trường hợp `node` và `npm` không thể được tìm thấy tự động:

```
Browsershot::html('Foo')
    ->setIncludePath('$PATH:/usr/local/bin')
```

### 3. Cài đặt

```
composer require spatie/browsershot
```

### 4. Cách sử dụng

- Thêm vào trên đầu file cần sử dụng:

```
use Spatie\Browsershot\Browsershot;
```

***a. Screenshots***

- Tạo ảnh từ trang web

```
Browsershot::url('https://example.com')->save($pathToImage);
```

***_a1. Thiết lập kích thước ảnh_***

- Mặc định screenshot sẽ có định dạng `png` và kích thước sẽ khớp với độ phân giải mà bạn sử dụng cho desktop. Tuy nhiên, bạn có thể chỉnh size cho screenshot như sau:

```
Browsershot::url('https://example.com')
    ->windowSize(640, 480)
    ->save($pathToImage);
```

- Bạn cũng có thể thiết lập kích thước của output image độc lập với kích thước của window:

```
Browsershot::url('https://example.com')
    ->windowSize(1920, 1080)
    ->fit(Manipulations::FIT_CONTAIN, 200, 200)
    ->save($pathToImage);
```

- Bạn có thể chỉ screenshot một phần của page bằng cách sử dụng clip:

```
Browsershot::url('https://example.com')
    ->clip($x, $y, $width, $height)
    ->save($pathToImage);
```

***_a2. Thao tác với ảnh_***

- Bạn có thể sử dụng tất cả các phương thức mà [spatie/image](https://docs.spatie.be/image/v1/image-manipulations/optimizing-images) cung cấp. Ví dụ như `greyscale()`:

```
Browsershot::url('https://example.com')
    ->windowSize(640, 480)
    ->greyscale()
    ->save($pathToImage);
```

***_a3. Chụp toàn bộ trang_***

```
Browsershot::url('https://example.com')
    ->fullPage()
    ->save($pathToImage);
```

***_a4. Thiết lập device scale_***

- Bạn cũng có thể chụp trang web với mật độ pixel cao hơn bằng cách truyền giá trị hệ số device scale là 2 hoặc 3:

```
Browsershot::url('https://example.com')
    ->deviceScaleFactor(2)
    ->save($pathToImage);
```

***b. PDFs***

- Browsershot sẽ lưu 1 file pdf nếu đường dẫn truyền vào phương thức `save()` có đuôi `.pdf`.

```
Browsershot::url('https://example.com')->save('example.pdf');
```

- Ngoài ra, nếu muốn rõ ràng hơn, bạn có thể sử dụng phương thức `savePdf`:

```
Browsershot::url('https://example.com')->savePdf('example.pdf');
```

- Bạn cũng có thể dễ dàng convert từ html sang pdf:

```
Browsershot::html($someHtml)->savePdf('example.pdf');
```

***_b1. Thiết lập kích thước file pdf_***

- Bạn có thể chỉ định `width` và `height` bằng đơn vị milimét:

```
Browsershot::html($someHtml)
   ->paperSize($width, $height)
   ->save('example.pdf');
```

***_b2. Thiết lập margin_***

- Margin có thể được thiết lập bằng đơn vị milimét:

```
Browsershot::html($someHtml)
   ->margins($top, $right, $bottom, $left)
   ->save('example.pdf');
```

***_b3 Header và footer_***

- Mặc định, file PDF sẽ không hiển thị header và footer được tạo bởi Chrome. Nếu bạn muốn hiển thị nó thì có thể dùng phương thức `showBrowserHeaderAndFooter`:

```
Browsershot::html($someHtml)
   ->showBrowserHeaderAndFooter()
   ->save('example.pdf');
```

***_b4. Background_***

- Mặc định file pdf không hiển thị background của html page. Nếu bạn muốn hiển thị nó thì có thể dùng phương thức `showBackground`:

```
Browsershot::html($someHtml)
   ->showBackground()
   ->save('example.pdf');
```

***_b5. Thiết lập khổ ngang_***

```
Browsershot::html($someHtml)
   ->landscape()
   ->save('example.pdf');
```

***_b6. Chỉ export những trang được chỉ định_***

```
Browsershot::html($someHtml)
   ->pages('1-5, 8, 11-13')
   ->save('example.pdf');
```

***c. HTML***

- Browsershot cũng có thể lấy body của một trang html sau khi JavaScript đã được thực hiện:

```
Browsershot::url('https://example.com')->bodyHtml();
```

***d. Misc***

- Thiết lập user argent:

```
Browsershot::url('https://example.com')
    ->userAgent('My Special Snowflake Browser 1.0')
    ->save($pathToImage);
```

- Giá trị timeout mặc định là 60 giây, bạn có thể thay đổi nó bằng phương thức `timeout()`:

```
Browsershot::url('https://example.com')
    ->timeout(120)
    ->save($pathToImage);
```

- Thiết lập network idle timeout:

```
Browsershot::url('https://example.com')
    ->setNetworkIdleTimeout(1000);
```

### 5. Tài liệu tham khảo

1. [Headless browsers là gì?]( http://hanhtranglaptrinh.vn6.vn/headless-browsers-la-gi/ )
2. [Google Chrome 59 với giao diện Material mới trong Settings và nhiều tính năng khác]( https://quantrimang.com/google-chrome-59-voi-giao-dien-material-moi-trong-settings-va-nhieu-tinh-nang-khac-135221 )
4. [Introducing Browsershot v3: the best way to convert html to PDFs and images]( https://murze.be/2017/10/introducing-browsershot-v3-best-way-convert-html-pdfs-images/ )
3. [Convert a webpage to an image or pdf using headless Chrome]( https://github.com/spatie/browsershot )
