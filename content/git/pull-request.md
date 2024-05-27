---
  title:  Pull Request
  draft: false
  date: 2024-05-15
  tags:
    - github
    - project-management
  description: How to create and customize Pull Request on GitHub
  aliases: 
---

Pull Request (PR) là một bản đề xuất các thay đổi từ một nhánh (**head/topic branch**) và mong muốn được gộp vào một nhánh khác (**base branch**). Mở một PR tạo một môi trường để mọi người thảo luận và đánh giá những thay đổi trước khi chúng được gộp vào một nhánh quan trọng.

Bên trong PR hiển thị:

- Sự khác biệt giữa hai nhánh
- Mô tả nguồn gốc vấn đề, giải thích các sự thay đổi

Bên trong PR, ta hoàn toàn có thể tạo thêm các commits để cập nhật các thay đổi. Mọi người có thể cùng tham gia đánh giá và đưa ra các bình luận. Khi được chấp nhận, các thay đổi trong PR có thể được merge vào _base branch_.

## 📝 Tạo PR

Khi có sự thay đổi (ví dụ, giải quyết một [issue](git/issue.md)), trước hết cần tạo một topic branch rẽ nhánh từ _base branch_ trên local repository, tạo các commits và đẩy chúng lên remote repository. Cuối cùng là tạo PR cho hai branches này

Thông thường PR được tạo trên GitHub Web UI, khi có branch mới được đẩy lên remote repository, GitHub sẽ đưa ra gợi ý về việc tạo PR cho nó. Trong PR này chú ý lựa chọn đúng base branch.

Nếu có cập nhật về mã nguồn, ta có thể tiếp tục tạo các commits và đẩy lên remote branch, các commits này sẽ tự động được thêm vào danh sách commits của PR (theo thứ tự thời gian), và sự khác biệt giữa hai branches sẽ được tổng hợp từ các commits đó.

Đối với một remote repository mà không có quyền tạo branch, ta có thể

- Tạo một [fork](git/fork.md)
- Clone fork
- Thiết lập một `git remote add` trỏ đến **upstream repository**. [Tham khảo](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork)
- Tạo branch, commits và đẩy lên remote upstream đó.

## 📃 PR Template

PR template giúp cung cấp sẵn bản mô tả các thông tin cần thiết tổng hợp các thay đổi trong PR, đặc biệt trong dự án có sự hợp tác của nhiều người.

PR Template đơn giản là một tệp Markdown nằm trong

- **.github directory** (có thể đặt trong thư mục con _PULL_REQUEST_TEMPLATE_)
- **Root directory**
- **`docs` directory**

> [!note]
>
> Thông thường repository chỉ cần một PR Template, thường đặt tên là **pull_request_template.md**, đặt một trong các thư mục trên.

## ⛓️‍💥 PR Conflict

Khi tạo PR, các so sánh dựa trên snapshot hiện tại của _base branch_, do đó, nếu có các PRs được merge vào _base branch_ trước có thể gây các xung đột trên PR hiện tại.

Khi đó, ta cần sửa các xung đột trên local repository

- Kéo trạng thái mới nhất từ _base branch_
- Rebase/Merge từ _topic branch_ vào _base branch_
- Giải quyết xung đột
- Commit và đẩy lên _topic remote branch_
