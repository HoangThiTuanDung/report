## Giới thiệu về 1 Javascript framework mới - Stimulus

### 1. Giới thiệu

### _a. Định nghĩa_

- Nhóm nghiên cứu tại Basecamp đã phát hành 1 Javascript framework mới (version 1.0.0) gọi là Stimulus, có thể sử dụng từ 30/01/2018.

- Stimulus là 1 Javascript framework với "tham vọng khiêm tốn" (modest ambition). Không giống các framework khác, Stimulus không chiếm toàn bộ giao diện ứng dụng của bạn. Thay vào đó, nó được thiết kế để tăng cường HTML bằng cách kết nối các phần từ đến các đối tượng Javascript tự động.

### _b. Kết nối HTML đến Javascript_

- Stimulus hoạt động bằng cách liên tục theo dõi trang web, chờ thuộc tính magic `data-controller` xuất hiện. Giống như thuộc tính class, bạn có thể put nhiều hơn 1 giá trị bên trong nó. Nhưng thay vì áp dụng hoặc loại bỏ các tên class CSS, các giá trị `data-controller` kết nối hoặc hủy kết nối các controller của Stimulus.

- Hãy nghĩ về nó như thế này: Giống như cách mà class là 1 cây cầu kết nối HTML với CSS, `data-controller` là 1 cây cầu từ HTML đến Javascript.

- Trên cơ sở đó, Stimulus thêm các thuộc tính magic `data-action` để mô tả làm thế nào mà các event trên page nên kích hoạt các controller method, và các thuộc tính `data-target` để cho phép bạn tìm ra các phần tử trong controller scope.

### _c. Tách các mối quan tâm_

- Các thuộc tính magic của Stimulus cho phép bạn tách riêng nội dung khỏi hành vi theo cách giống như cách bạn tách nội dung khỏi CSS. Ngoài ra, các Stimulus convention khuyến khích bạn nhóm code theo tên.

- Sự sắp xếp này giúp bạn xây dựng các controller tái sử dụng, cho bạn cấu trúc đủ để giữ cho code không bị phân chia thành “1 bát súp Javascript”.

### _d. Tài liệu dễ đọc_

- Hành vi Javascript của bạn được ánh xạ ra trong các thuộc tính magic, do đó bạn có thể đọc 1 đoạn HTML và biết điều gì đang xảy ra.

- Đánh dấu dễ đọc cũng cho phép những người khác trong team có thể dễ dàng theo dõi và chẩn đoán vấn đề 1 cách nhanh chóng.

### 2. Cài đặt với Laravel

- Cài đặt package cần thiết:

```
yarn add stimulus turbolinks
yarn add --dev babel-plugin-transform-class-properties
```

- Thêm đoạn code sau vào file `.babelrc` trong thư mục root của framework:

```
{
   "plugins": ["transform-class-properties"],
     "presets": [
       ["env", { "modules": false }]
   ]
}
```

- Tạo thư mục `resources/assets/js/controllers` và replace đoạn code sau trong file `resources/assets/js/app.js`:

```
var Turbolinks = require("turbolinks")
Turbolinks.start()

import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
const context = require.context("./controllers", true, /\.js$/)
application.load(definitionsFromContext(context))
```

- Lưu ý: Tubolink cần nằm trong phần `<head>` của tài liệu HTML. Phần `<body>` sẽ được thay thế khi được điều hướng sau khi nhận document thông qua XHR, do đó script cần phải được để bên ngoài thẻ `<body>`. Vì vậy, chỉ cần thêm file `app.js` trong phần head như sau:

```
<head>
    <!-- ... -->
    <link href="{{ mix('css/app.css') }}" rel="stylesheet" type="text/css" />
    <script src="{{ mix('js/app.js') }}"></script>
</head>
```

### 3. Tìm hiểu cách thức hoạt động thông qua ví dụ

### _a. Bắt đầu với HTML_

- Hãy bắt đầu với 1 bài tập đơn giản: Tạo 1 text field với 1 button. Khi click vào button, sẽ hiển thị giá trị của text field trong console.

- Mở file `public/index.html` và thêm nội dung sau vào thẻ body:

```
<div>
    <input type="text">
    <button>Greet</button>
</div>
```

- Reload lại trình duyệt bạn sẽ thấy có text field và button.

### _b. Các controller mang HTML vào cuộc sống_

- Ở cốt lõi của nó, mục đích của Stimulus là tự động kết nối các phần tử DOM với các đối tượng Javascript. Các đối tượng này gọi là các controller.

- Hãy tạo controller đầu tiên bằng cách mở rộng controller class được xây dựng trong framework. Tạo 1 file với tên gọi là `hello_controller.js` trong thư mục `src/controllers/`. Sau đó đặt đoạn code sau trong nó:

```
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
}
```

### _c. Các identifier liên kết các controller với DOM_

- Tiếp theo, chúng ta cần phải nói với Stimulus cách controller này được kết nối với HTML. Chúng ta làm điều này bằng cách đặt 1 identifier trong thuộc tính `data-controller` của thẻ div:

```
<div data-controller="hello">
    <input type="text">
    <button>Greet</button>
</div>
```

- Các identifier được dùng làm liên kết giữa các phần tử và các controller. Trong trường hợp này, identifier hello nói Stimulus tạo 1 instance của controller class trong file `hello_controller.js`.


### _d. Làm sao để biết controller có hoạt động hay không?_

- Reload lại page và bạn thấy không có gì thay đổi. Làm sao biết được controller có hoạt động hay không?

- Có 1 cách là đặt 1 câu lệnh log trong phương thức `connect()` – đây là phương thức được Stimulus gọi mỗi lần 1 controller được kết nối với document.

- Thực hiện phương thức `connect()` trong `hello_controller.js` như sau:

```
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
    connect() {
        console.log("Hello, Stimulus!", this.element)
    }
}
```

### _e. Các action phản hồi đến các DOM event_

- Bây giờ chúng ta hãy xem làm thế nào để thay đổi code để message log xuất hiện khi chúng ta click vào button “Greet”.

- Bắt đầu bằng đổi tên method `connect()` thành `greet()`:

```
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
    greet() {
        console.log("Hello, Stimulus!", this.element)
    }
}
```

- Chúng ta muốn gọi method `greet()` khi sự kiện click button được kích hoạt. Trong Stimulus, các controller method mà xử lý các sự kiện được gọi là các action method.

- Để kết nối action method đến sự kiên click button, hãy mở file `public/index.html` và thêm 1 magic `data-action` vào button:

```
<div data-controller="hello">
    <input type="text">
    <button data-action="click->hello#greet">Greet</button>
</div>
```

- Trong đó,
    + `click` là tên event
    + `hello` là idenntifier của controller
    + `greet` là tên method được gọi trong controller

### _f. Các target map các yếu tố quan trọng đến các thuộc tính controller_

- Chúng ta sẽ kết thúc bài tập bằng cách thay đổi action say hello đến bất cứ tên nào được nhập vào text field.

- Để làm điều đó, đầu tiên chúng ta cần tham chiếu đến yếu tố input bên trong controller. Sau đó, chúng ta có thể đọc thuộc tính value để lấy nội dung của nó.

- Stimulus cho phép chúng ta đánh dấu các yếu tố quan trọng như target, vì vậy chúng ta có thể dễ dàng tham chiếu chúng trong controller thông qua các thuộc tính tương ứng. Mở file `public/index.html` và thêm 1 thuộc tính magic `data-target` đến phần tử input:

```
<div data-controller="hello">
    <input data-target="hello.name" type="text">
    <button data-action="click->hello#greet">Greet</button>
</div>
```

- Trong đó,
    + `hello` là controller identifier
    + `name` là tên target

- Khi chúng ta thêm name vào danh sách các target được định nghĩa của controller, Stimulus sẽ tự động tạo 1 thuộc tính là `this.nameTarget` mà trả về phần tử target phù hợp đầu tiên. Chúng ta có thể sử dụng thuộc tính này để đọc giá trị phần tử và xây dựng chuỗi lời chào.

- Mở file `hello_controller.js` và cập nhật nó như sau:

```
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
    static targets = [ "name" ]

    greet() {
        const element = this.nameTarget
        const name = element.value
        console.log(`Hello, ${name}!`)
    }
}
```

### _g. Các controller đơn giản hóa việc tái cấu trúc_

- Chúng ta có thể thấy rằng, các controller của Stimulus là các instance của các class Javascipt có các phương thức có thể hoạt động như các trình xử lý sự kiện.

- Điều đó có nghĩa là chúng ta có sẵn các kỹ thuật tái cấu trúc tiêu chuẩn. Ví dụ, chúng ta có thể dọn dẹp phương thức `greet()` bằng 1 getter name:

```
// src/controllers/hello_controller.js
import { Controller } from "stimulus"

export default class extends Controller {
    static targets = [ "name" ]

    greet() {
        console.log(`Hello, ${this.name}!`)
    }

    get name() {
        return this.nameTarget.value
    }
}
```

### 4. Tài liệu tham khảo

1. [A modest JavaScript framework for the HTML you already have.](https://stimulusjs.org/)
2. [Stimulus: A modest JavaScript framework for the HTML you already have](https://laravel-news.com/stimulus-javascript-framework)
