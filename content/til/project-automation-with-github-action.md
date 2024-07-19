---
  title:  Project Automation with GitHub Actions
  draft: false
  date: 2024-05-28
  tags:
    - github
    - example
    - project-management
    - series-github-automation
  description: A detail example for GitHub Action workflow
---

### ⚡ Vấn đề

> 🐘 **Trong quá trình quản lý dự án, các issues (tasks) thường phải chia nhỏ để dễ quản lý tiến độ tương ứng với mỗi pha phát triển như triển khai hay kiểm thử. Với một công việc lặp đi lặp lại như vậy, có cách nào để giảm công sức?** 🤑

🎈Nếu bạn lựa chọn GitHub Project để quản lý dự án thì bạn có thể giải quyết nhiệm vụ trên với **GitHub Actions**, tuy nhiên, cũng không phải là đơn giản với lần đầu trải nghiệm... 🥹

Về nội dung liên quan đến lĩnh vực này, các chủ đề về [github](tags/github) của mình cũng đã trình bày nội dung cơ bản nhất. Ngoài ra, bạn có thể tham khảo trên [GitHub Docs](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project).

Trong bài này, mình chỉ show code và giải thích. Code cho GitHub Action workflows cũng kha khá command lines, siêu thú vị.

### 🔎 Phân tích vấn đề

Có một số điểm chú ý với vấn đề này

1️⃣ _Các loại issues khác nhau cần chia theo các tasks khác nhau_

Thông thường có hai loại issues được tạo cho repository:

- ✅ **New Feature Issue**, liên quan đến tính năng mới, thường chia ra thành các bước: _Đánh giá khả thi_, _Triển khai_, _Kiểm thử_, _UAT Feedback_,...
- ⭕ **Bug Issue**, liên quan đến các lỗi của ứng dụng, bên cạnh các bước ở trên (ngoại trừ _Đánh giá khả thi_), cần có thêm bước _Điều tra nguyên nhân/phạm vi ảnh hưởng_,...

2️⃣ _Không phải issue nào cũng cần chia tasks_

> Ví dụ như các issues thảo luận, các issues liên quan đến cập nhật tài liệu,...

3️⃣ _Khi tạo các tasks con, cần thêm lại nó vào issue ban đầu_

> Ví dụ liệt kê chúng thành [Tasklist](https://docs.github.com/en/issues/managing-your-tasks-with-tasklists/)

4️⃣ _Để phục vụ quản lý tập trung, cần tự động thêm issue từ repository vào project, đồng thời cấu hình sẵn một số giá trị cho một số trường trong project_

> [Project built-in workflow](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-built-in-automations) có thể tự động thêm một issue có label, assignee,... cho trước vào project, tuy nhiên, _để cấu hình giá trị cho các trường không phải "Status"_ cần sử dụng [GitHub CLI](https://docs.github.com/en/github-cli).

### 🕯Hướng giải quyết

#### 🧪 Đánh giá

> [!info]
>
> Các đánh giá này có được sau khi đã có code chạy thành công, nhưng đặt lên trước cho nó chuyên nghiệp, đúng trình tự 🤡.

Các yêu cầu 1️⃣ 2️⃣ 3️⃣ liên quan đến cập nhật repository, nên khá dễ dàng. Công việc gồm hai phần:

- Tạo các tasks con
- Cập nhật liên kết tới các tasks này vào issue cha

Cả hai công việc này đều có thể dùng [GitHub CLI](https://cli.github.com/manual/) với [GITHUB_TOKEN](git/github-action.md#🎫-github_token) cung cấp sẵn.

Yêu cầu 4️⃣ là khoai nhất, vì

- `GITHUB_TOKEN` chỉ có các quyền truy cập tài nguyên bên trong **repository**, nên để thao tác với project, cần có **Personal Access Token** (thật) hoặc **GitHub App ID** (khuyến khích đối với _Organization Project_, vì Token này gắn với tổ chức thay vì cá nhân) để cấp quyền cho GitHub Action.
- Để cập nhật issue trong project, cần lấy được ID của các trường, nên ta cần sử dụng thêm cú pháp của [GraphQL](https://docs.github.com/en/graphql).

> [!note]
>
> `GITHUB_TOKEN` tự động sinh bởi GitHub, nên các tác vụ sử dụng Token này sẽ hiển thị người thực hiện là **GitHub Bot**.
>
> Nếu muốn hiển thị bạn là người thực hiện, hãy sinh Personal Access Token và gán cho các tác vụ cần Token.

#### 💡 Ý tưởng triển khai

1️⃣ **_Sự kiện kích hoạt GitHub Action workflow là gì?_**

Có một số sự kiện liên quan đến [issue](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issues), cần chọn sự kiện ít gây "side-effects" nhất.

Mình lựa chọn **`opened`**

2️⃣ **_Làm sao để phân biệt issue cần tạo task con?_**

Trả lời cho câu hỏi này, ta có thể thêm điều kiện cho các _jobs_ với **`if`**

Có thể sử dụng một số trường giá trị để nhận biết

- _Sử dụng prefix cho title_. VD: "TASK: Bug on rendering Vietnamese"
- _Sử dụng milestone, assignee_. Chỉ những issue nào được gán milestone, assignee mới cần tạo tasks. Điều kiện này liên quan đến sự kiện kích hoạt nhiều hơn.
- _Sử dụng labels_

Mình lựa chọn nhận biết theo label **`Task`**, đồng nghĩa với việc cần tạo sẵn label này trên repository.

3️⃣ **_Làm sao để phân biệt các kiểu issues?_**

Ý tưởng tự nhiên là sử dụng **label**.

Các kiểu issues cũng thường có các [template](git/issue.md#📃-issue-template) khác nhau, ở các templates này, ta cũng có thể định nghĩa labels gán sẵn thông qua markdown frontmatter `labels`.

```markdown
---
name: Bug report
about: Something about Quartz isn't working the way you expect
labels: bug,task
---
```

4️⃣ **_Làm sao để biết được danh sách tasks ứng với từng kiểu issue?_**

Danh sách này khác nhau giữa các kiểu issues khác nhau (như đã nói), nên ý tưởng tự nhiên nhất là định nghĩa chúng thành tệp, mỗi kiểu một tệp. Phân biệt các issue bằng label nên việc tìm kiếm tệp tương ứng cũng sẽ thông qua label.

### 🥁 Tiếp nối mạch câu chuyện

Ái chà, bất ngờ chưa, phần 1 đến đây là hết rồi🪫. [Phần 2](#) ở đây nè [🔋](til/project-automation-with-github-action-p2.md).
