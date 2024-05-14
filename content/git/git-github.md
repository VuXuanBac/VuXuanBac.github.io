---
  title:  Git and Github
  draft: true
  date: 2024-05-14
  tags:
    - git
  description: Git Basic Tutorial
  aliases:
---

Git là một hệ thống quản lý phiên bản (_version control system - VCS_) - quản lý các thay đổi trên dự án. GitHub là nền tảng đám mây hỗ trợ làm việc nhóm trên cơ sở của Git.

## Một số khái niệm

📁 **repository** là một dự án được quản lý bởi Git, bao gồm toàn bộ các tệp và thư mục cùng với toàn bộ các phiên bản (lịch sử thay đổi) của chúng.

⛓ **commit** là một bản snapshot của dự án tại một thời điểm. Cụ thể, thay vì quản lý dữ liệu dưới dạng các cập nhật theo thời gian (như một số VCS khác - Xem hình bên trái), Git lưu lại toàn bộ các tệp trong mỗi phiên bản của dự án: những tệp không có sự thay đổi sẽ chỉ cần lưu lại tham chiếu từ phiên bản trước.

| ![Store data as changes of each file](assets/git/delta-base-data-management.png) | ![Store data as snapshots of the project overtime](assets/git/git-data-management.png) |
| :------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------: |
|        _Delta based data management: store data as changes of each file_         |                 _Git store data as snapshots of the project overtime_                  |

🔀 **file status** là trạng thái của tệp, trạng thái này có thể thay đổi qua một số thao tác. Cụ thể, mỗi tệp của dự án có thể trải qua các trạng thái sau:

- ⬜ **untracked**: Tệp chưa được quản lý bởi Git. VD: Tệp mới tạo, Tệp đã xóa,...
- 🟧 **modified**: Tệp có sự thay đổi so với snapshot gần nhất.
- 🟩 **unmodified**: Tệp chưa có sự thay đổi nào kể từ snapshot gần nhất.
- 🟥 **staged**: Tệp chuẩn bị được thêm vào commit tiếp theo.

Quá trình chuyển đổi trạng thái này diễn ra như sau:

![File status transition flow](assets/git/file-status-transition-flow.png)

🌵 **branch** là một khái niệm quan trọng trong các VCS, giúp cho việc tách biệt những thay đổi sắp tới thành một luồng (nhánh) riêng tránh việc làm xáo trộn luồng (nhánh) nội dung chính. Từ đó mà branching giúp cho nhiều người có thể độc lập thực hiện các thay đổi: mỗi người thực hiện thay đổi trên một branch riêng. Các thao tác với branch trong Git khá đơn giản và chi phí thấp.

Branch thực chất là một con trỏ trỏ tới một commit, giống như một cột mốc cho một chuỗi các commits. Việc di chuyển giữa các branch giúp chuyển đổi phiên bản (snapshot) của dự án.

Git có một con trỏ đặc biệt là **HEAD**, giúp xác định phiên bản của dự án hiện tại. Con trỏ này luôn trỏ tới một branch, gọi là **working branch**. Việc chuyển đổi các branches thực chất là thay đổi HEAD.

## Một số lệnh cơ bản

| Lệnh         | Mô tả                                                                                                                                                                                |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `git init`   | Khởi tạo một repository cho dự án. Thư mục **.git** được tạo chứa các tệp cần thiết cho việc quản lý                                                                                 |
| `git clone`  | Tạo một bản sao của một repository (remote) trên máy cục bộ, bao gồm nội dung dự án và **.git**                                                                                      |
| `git status` | Xem trạng thái của các tệp có trạng thái **untracked, modified (edit, delete), staged**                                                                                              |
| `git add`    | Chuyển trạng thái của các tệp chỉ định về **staged** (chuẩn bị cho commit tiếp theo)                                                                                                 |
| `git commit` | Tạo một commit với các tệp **staged**, lúc này các thay đổi sẽ được lưu trữ lâu dài trong dữ liệu của **.git** và trạng thái của các tệp **staged** sẽ được chuyển về **unmodified** |

## Remote Repository

Với mục đích hỗ trợ hợp tác trong quá trình phát triển, một repository thường được chia sẻ lên một nền tảng quản lý phiên bản trên mạng như GitHub. Khi đó, phiên bản repository này được gọi là **remote repository**.

Một repository cục bộ có thể gắn với nhiều remote repositories. Trường hợp này thường liên quan tới việc có nhiều người cùng đóng góp cho dự án và mỗi remote repository tương ứng với từng người.

Khi sử dụng `git clone`, liên kết đến remote repository sẽ tự động được thêm vào repository được tải về.

Danh sách các remote repositories được hiển thị qua lệnh **`git remote -v`**. Thông tin được hiển thị bao gồm tên và liên kết.

Để thêm một remote repository, sử dụng lệnh **`git remote add <name> <url>`**.

Tương tự, ta có thể đổi tên và xóa một remote repository:

```
git remote rename <old-name> <new-name>

git remote remove <name>
```

Những thay đổi trên repository cục bộ sẽ không tự động đồng bộ lên các remote repositories. Tức là thường sẽ có sự khác biệt giữa chúng, có thể là do _thay đổi tại repository cục bộ_ hoặc _thay đổi từ remote repository_ (khi được cập nhật từ một repository cục bộ khác, ví dụ, ở máy khác). Do đó, quá trình đồng bộ thường có 2 thao tác:
- **`git pull`** hoặc **`git fetch`**: Kéo dữ liệu từ remote repository về local repository.
- **`git push`**: Đẩy dữ liệu từ local repository lên remote repository.