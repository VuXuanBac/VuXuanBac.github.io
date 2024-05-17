---
  title:  API Best Practices [Part 1]
  draft: false
  date: 2024-05-17
  tags:
    - api
    - til
    - series-api-best-practices
  description: Take notes some best practices for writing API [P1 - Design]
---

### ⚡ Vấn đề

> 🐠 **Bạn gặp một số khó khăn và đang tìm kiếm một "công thức" cho việc thiết kế API cho ứng dụng của mình ?** 🥺

Dưới đây là một số lời khuyên và ghi chú mình học được từ cuốn sách _"Build APIs You Won't Hate" - Phil Sturgeon_ [2015]

### 🗃 Tổ chức dữ liệu

1️⃣ **Sử dụng dữ liệu riêng cho các môi trường khác nhau: Development, Staging, Production**

Dữ liệu có thể sinh ngẫu nhiên song vẫn cần phù hợp với định dạng của nó, ví dụ một email, số điện thoại,.... [Faker](https://github.com/fzaninotto/Faker) là một thư viện quá nổi tiếng cho mục đích này.

2️⃣ **Xóa dữ liệu của các bảng có thể thất bại vì các ràng buộc toàn vẹn, đặc biệt là FKs**

Khi đó có thể tắt tạm thời chức năng kiểm tra toàn vẹn FKs bằng lệnh SQL sau:

```sql
SET FOREIGN_KEY_CHECKS=0;
```

3️⃣ **Sử dụng `AutoIncrement` cho ID cũng có hạn chế**

Đánh số tăng tự động cho ID tiết lộ về số lượng bản ghi trong DB. Nếu có dữ liệu cần che dấu, hãy sử dụng UUID (_universal unique identifier_) cho trường ID

### 🔷 Thiết kế Endpoints

Endpoints là địa chỉ mà yêu cầu từ phía client gửi đến server, chính là HTTP Method + URL. Thông qua endpoint, ứng dụng phía client có thể giao tiếp với API.

1️⃣ **PUT nên được dùng cho các hành động có kết quả nhất quán (_idempotent_, tức là việc thực hiện lặp lại luôn cho cùng một kết quả - phía server)**

Ví dụ 1: Một bài viết có nội dung ảnh,

- Nếu bài viết có thể có nhiều ảnh, hãy sử dụng POST khi muốn đăng tải một ảnh mới.
- Nếu bài viết chỉ có một ảnh, hãy sử dụng PUT vì ta mong muốn việc đăng tải nhiều lần chỉ thực hiện ghi đè ảnh cũ.

Ví dụ 2: Người dùng cập nhật các cấu hình của họ

- Nếu muốn mỗi yêu cầu cập nhật từng cấu hình, hãy sử dụng POST.
- Nếu mỗi yêu cầu cần gửi toàn bộ các cấu hình, hãy sử dụng PUT.

2️⃣ **Không nên sử dụng động từ trong URL, bản thân HTTP Method đã là một động từ**

URL là một địa chỉ nên hãy biểu diễn nó giống như một địa chỉ, xác định tài nguyên mà server cung cấp.

3️⃣ **Sử dụng dạng số nhiều cho tài nguyên**

Số nhiều đại diện cho một tập hợp, nên phù hợp với các thao tác trên tập hợp. Với các thao tác trên một phần tử, có thể sử dụng ID để xác định phần tử đó.

4️⃣ **Nên giữ các Endpoints độc lập nhất có thể**

Việc tổ chức các thao tác cập nhật vào các Controllers có thể gặp nhiều khó khăn và cần cân nhắc. Ví dụ, khi đăng ảnh cho bài nên để PostController xử lý hay một ImageController xử lý (và tham chiếu ID của bài viết trên URL).

Song, dù thế nào thì các Endpoints khác nhau cần có logic xử lý khác nhau, không nên có nhiều Endpoints có xử lý giống nhau.

### ⚖ Dữ liệu gửi lên và trả về

1️⃣ **Nên sử dụng JSON cho định dạng dữ liệu**

Ví dụ dưới đây đang đóng gói dữ liệu theo định dạng Form và gửi lên server với **`Content-Type: application/x-www-form-urlencoded`**

```
POST /checkins HTTP/1.1
Host: api.example.com
Authorization: Bearer vr5HmMkzlxKE70W1y4MibiJUusZwZC25NOVBEx3BD1
Content-Type: application/x-www-form-urlencoded

checkin[place_id]=1&checkin[message]=This is a bunch of text&checkin[with_friends][]=\
1&checkin[with_friends][]=2&checkin[with_friends][]=3&checkin[with_friends][]=4&check\
in[with_friends][]=5
```

Có thể thấy việc tổ chức dữ liệu này rất khó theo dõi.

Còn với XML, vấn đề lớn nhất của nó là việc tổ chức các tags làm tăng kích thước dữ liệu.

Cả hai định dạng này cũng làm mất kiểu của các trường dữ liệu, ví dụ như `true`, `false` sẽ chuyển về số nguyên hay `null` và `""` được coi tương đương nhau.

> [!note]
>
> Cùng với JSON, YAML cũng là một lựa chọn tốt.

2️⃣ **Lựa chọn cho việc tổ chức dữ liệu cũng rất đa dạng**

1. [**JSON-API**](http://jsonapi.org/format/)
2. **Twitter-style**

```JS
// single resource
{
    "name": "Hulk Hogan",
    "id": "100002"
}

// multiple resources
[
    {
        "name": "Hulk Hogan",
        "id": "100002"
    },
    {
        "name": "Mick Foley",
        "id": "100003"
    }
]
```

3. **Facebook-style**

```JS
// single resource
{
    "name": "Hulk Hogan",
    "id": "100002"
}

// multiple resources
{
    "data":  [
        {
            "name": "Hulk Hogan",
            "id": "100002"
        },
        {
            "name": "Mick Foley",
            "id": "100003"
        }
    ]
}
```

Hạn chế của **2.** và **3.** là thiếu chỗ cho các metadata như pagination, sorting,... Nên giải pháp thứ 4 là:

4. Sử dụng Namespace (`data` như ví dụ trên) cho cả single và multiple resources.

### ⁉ Status Code và Message

Mã phản hồi (_status code_) giúp client xác định nhanh kết quả xử lý từ server, các mã phản hồi có ý nghĩa khác nhau, có thể sử dụng trong nhiều trường hợp, và có thể nhóm lại thành các nhóm:

| Nhóm | Mô tả                                                                                                                                    |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1xx  | Thông tin về giao thức kết nối                                                                                                           |
| 2xx  | Thao tác được xử lý thành công                                                                                                           |
| 3xx  | Yêu cầu được chuyển hướng đến một nơi khác để xử lý                                                                                      |
| 4xx  | Yêu cầu từ client không hợp lệ và không nên gửi lại yêu cầu mà không có sự thay đổi                                                      |
| 5xx  | Một số lỗi tại server khiến yêu cầu không được xử lý như lỗi kết nối tới DB hoặc quá tải. Client có thể thử lại sau một khoảng thời gian |

**Danh sách các mã phản hồi**: [Tham khảo](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

1️⃣ **Nên có thêm thông tin về lỗi cùng với mã phản hồi**

Có nhiều trường hợp lỗi cùng trả về một mã phản hồi, vì chúng cùng thuộc một nhóm các lỗi có liên quan đến nhau. Ví dụ như các lỗi về đăng nhập có thể do nhiều nguyên nhân như:

- Tài khoản hoặc mật khẩu không đúng
- Tài khoản bị khóa
- Thời điểm đăng nhập không được phép

Ví dụ khác là mã phản hồi `404: Not Found` thường bị lạm dụng cho một số lỗi như:

- Tài nguyên chưa từng được tạo
- Tài nguyên đã bị xóa
- Tài nguyên hiện chưa phù hợp để hiển thị
- Bạn không có quyền đọc với tài nguyên

Do đó, nếu ứng dụng muốn thông báo cho người dùng về nguyên nhân gây lỗi, cần sử dụng thêm một số trường thông tin:

- `errorType`: Kiểu lỗi, ví dụ: lỗi liên quan đến OAuth.
- `errorCode`: Mã lỗi, cụ thể hóa ý nghĩa của từng lỗi.
- `errorMessage`: Phản hồi chi tiết về lỗi, human-readable.

2️⃣ **Trường dữ liệu cần thiết cho lỗi**

Có một số cách để lựa chọn cách tổ chức dữ liệu cho các lỗi

1. [**JSON-API**](https://jsonapi.org/format/#errors)

JSON-API cũng cung cấp cách thức mô tả cho lỗi:

| Trường dữ liệu | Mô tả                                                  |
| -------------- | ------------------------------------------------------ |
| id             | Định danh cho vấn đề (cụ thể) hiện tại                 |
| href           | Liên kết đến địa chỉ chứa mô tả chi tiết về vấn đề này |
| status         | Chuỗi mô tả HTTP Status Code                           |
| code           | Chuỗi mô tả mã lỗi (đặc tả của ứng dụng)               |
| title          | Mô tả vắn tắt của lỗi                                  |
| details        | Mô tả chi tiết về vấn đề hiện tại                      |
| links          | Các tài nguyên liên quan                               |
| path           | Thuộc tính của các tài nguyên liên quan                |

2. [**API Problem**](https://datatracker.ietf.org/doc/html/draft-nottingham-http-problem)

Một chuẩn để mô tả các vấn đề xảy ra. [Bài viết của Mark Nottingham](https://www.mnot.net/blog/2013/05/15/http_problem) mô tả về chuẩn này.

### 🪝Tham khảo

Chuỗi bài viết về API Best Practices sẽ được cập nhật. Bạn có thể đọc tiếp trong [Series API Best Practice](tags/series-api-best-practices)
