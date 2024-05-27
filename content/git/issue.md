---
  title:  Issue
  draft: false
  date: 2024-05-21
  tags:
    - github
    - project-management
  description: A solution for work tracking
---

**GitHub Issue** là một đối tượng giúp thể hiện các ý tưởng, yêu cầu hoặc một nơi để thảo luận một vấn đề liên quan đến các tính năng của dự án.

Issue cũng giúp tham chiếu lẫn nhau và tham chiếu tới các [Pull Request - PR](git/pull-request.md), tạo thành một chuỗi các đối tượng liên quan đến nhau, hỗ trợ trong [quản trị dự án](git/project-management.md).

- Issue có thể tổ chức theo cấp, một issue cha có thể bao gồm nhiều issues con, gọi là **task list**.
- Issue có thể được phân nhóm theo milestones, labels, assignees...

## 🖐 Tạo issue

GitHub hỗ trợ nhiều cách để tạo một issue:

- Tạo trên tab **Issue** bên trong Repository.
- Sử dụng GitHub CLI để tạo issue từ command line.
- Tạo từ GitHub Project
- Tạo từ Tasklist trong một issue khác
- Tạo từ một comment trong một issue khác, một đoạn mã, từ Dicussions,...

Sau đó, xác định các giá trị cho các trường:

- _Title_: Nội dung hiển thị trên danh sách Issues
- _Body/Description_: Mô tả vấn đề, sử dụng Markdown.
- _Label_: Gán các nhãn giúp lọc
- _Assignee_: Nếu là một thành viên trong dự án, có thể chỉ định người giải quyết issue này.
- _Project_: Một GitHub project quản lý issues này.
- _Milestone,..._

Sau khi tạo issue, các đóng góp và thảo luận có thể tiếp tục bằng các **comments**.

## 📃 Issue Template

Issue template giúp cung cấp sẵn bản mô tả các thông tin cần thiết cho issue, đặc biệt trong dự án có sự hợp tác của nhiều người.

> [!note]
>
> Các tệp mô tả issue template được đặt trong **.github/ISSUE_TEMPLATE**

> [!important]
>
> Issue template được lưu trong branch mặc định (thường là `main`) của repository, tức là

Có thể tạo issue template bằng hai cách

1️⃣ Sử dụng template builder, mô tả cấu trúc cho _title_ và _description_, sử dụng Markdown (**.md**)

VD:

```markdown
---
name: 🐞 Bug
about: File a bug/issue
title: "[BUG] <title>"
labels: Bug, Needs Triage
assignees: ""
---

<!--
Note: Please search to see if an issue already exists for the bug you encountered.
-->

### Current Behavior:

<!-- A concise description of what you're experiencing. -->

### Expected Behavior:

<!-- A concise description of what you expected to happen. -->

### Steps To Reproduce:

<!--
Example: steps to reproduce the behavior:
1. In this environment...
1. With this config...
1. Run '...'
1. See error...
-->

### Environment:

<!--
Example:
- OS: Ubuntu 20.04
- Node: 13.14.0
- npm: 7.6.3
-->

### Anything else:

<!--
Links? References? Anything that will give us more context about the issue that you are encountering!
-->
```

> [!note]
>
> Một template builder hợp lệ yêu cầu có hai trường `name` và `about` trên fronmatter

2️⃣ Sử dụng issue form, mô tả các trường dữ liệu và tập giá trị lựa chọn hoặc nhập liệu, sử dụng YAML (**.yml**)

Khi người tạo issue nhập liệu, dữ liệu đó sẽ được chuyển về dạng markdown để hiển thị.

VD:

```yaml
name: Bug Report
description: File a bug report.
title: "[Bug]: "
labels: ["bug", "triage"]
projects: ["octo-org/1", "octo-org/44"]
assignees:
  - octocat
body:
  - type: markdown
    attributes:
      value: |
        Thanks for taking the time to fill out this bug report!
  - type: input
    id: contact
    attributes:
      label: Contact Details
      description: How can we get in touch with you if we need more info?
      placeholder: ex. email@example.com
    validations:
      required: false
  - type: textarea
    id: what-happened
    attributes:
      label: What happened?
      description: Also tell us, what did you expect to happen?
      placeholder: Tell us what you see!
      value: "A bug happened!"
    validations:
      required: true
  - type: dropdown
    id: version
    attributes:
      label: Version
      description: What version of our software are you running?
      options:
        - 1.0.2 (Default)
        - 1.0.3 (Edge)
      default: 0
    validations:
      required: true
  - type: dropdown
    id: browsers
    attributes:
      label: What browsers are you seeing the problem on?
      multiple: true
      options:
        - Firefox
        - Chrome
        - Safari
        - Microsoft Edge
  - type: textarea
    id: logs
    attributes:
      label: Relevant log output
      description: Please copy and paste any relevant log output. This will be automatically formatted into code, so no need for backticks.
      render: shell
  - type: checkboxes
    id: terms
    attributes:
      label: Code of Conduct
      description: By submitting this issue, you agree to follow our [Code of Conduct](https://example.com).
      options:
        - label: I agree to follow this project's Code of Conduct
          required: true
```

> [!note]
>
> Một issue form hợp lệ yêu cầu có hai trường `name` và `description`

**🪝Tham khảo**:

- [Issue Form](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)
- [Syntax Issue Form](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-githubs-form-schema)
