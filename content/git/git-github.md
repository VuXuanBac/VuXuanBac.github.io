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

📁 **repository** là một dự án được quản lý bởi Git, bao gồm toàn bộ các tệp và thư mục cùng với toàn bộ các phiên bản của chúng.

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
