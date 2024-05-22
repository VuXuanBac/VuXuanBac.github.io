---
  title:  GitHub Action
  draft: false
  date: 2024-05-21
  tags:
    - git
    - github
    - ci-cd
  description: Using GitHub Action to automating jobs
  aliases:
---

GitHub Action là một nền tảng CI/CD hỗ trợ tự động hóa quá trình build, test và deployment bằng việc tự động thực thi một **workflow** người dùng định nghĩa khi một **sự kiện** xảy ra trên **repository**.

## Các khái niệm

### Workflow

Một workflow bao gồm một hay nhiều **jobs**, các _jobs_ này có thể thực thi tuần tự hoặc song song trong một **runner** riêng.

> [!note]
>
> Workflow được định nghĩa trong một tệp YAML nằm trong thư mục **.github/workflows** của repository.

### Sự kiện

Các workflows được thực thi tự động khi một sự kiện xảy ra, bao gồm:

- Sự kiện trên repository hiện tại
- Sự kiện bên ngoài GitHub nhưng kích hoạt sự kiện `repository_dispatch` trong GitHub.
- Thời gian lên lịch trước
- Tạo thủ công

Một sự kiện được định nghĩa bằng một hành động (activity) và cụ thể hóa qua một hay nhiều kiểu (types) và/hoặc tiêu chí (filters)

**🪝Tham khảo**:

- [Kích hoạt một workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)
- [Danh sách các sự kiện](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

### Job

Job là một thành phần tạo nên một _workflow_, nó bao gồm nhiều **steps** và các _steps_ này được chạy trên cùng một **runner**.

Các _jobs_ mặc định là **độc lập với nhau**, khi đó chúng có thể chạy song song với nhau. Song ta có thể chỉ định sự phụ thuộc cho chúng, đảm bảo chúng được chạy tuần tự.

### Runner

Runner là một server dùng để chạy các _jobs_, tại một thời điểm, một _runner_ chỉ chạy một _job_.

GitHub cung cấp sẵn các máy ảo Ubuntu, Windows, macOS để làm _runners_.

Ngoài ra có thể chỉ định một runner bên ngoài (self-host runner).

**🪝Tham khảo**:

- [Lựa chọn runner](https://docs.github.com/en/actions/using-jobs/choosing-the-runner-for-a-job)

### Action

Action là một ứng dụng giúp gom nhóm (với mục đích dùng lại) các _steps_ khi định nghĩa _job_ cho _workflow_. Action cũng được định nghĩa trong tệp YAML.

> [!note]
>
> Action cũng nhận dữ liệu đầu vào và trả dữ liệu ra, các thông tin này định nghĩa ngay trong tệp YAML.

Các _actions_ có thể tham chiếu từ:

- Repository hiện tại, với đường dẫn tương đối theo thư mục của repository
- Từ một public repository, với tham chiếu dạng **`{owner}/{repository}@{ref}`**
  - Trong đó: `ref` có thể là Tag, SHA hoặc Branch
- Từ một container trên Docker Hub, với tham chiếu dạng **`docker://{image}:{tag}`**

> [!tip]
>
> Ta có thể tìm kiếm các **actions** được định nghĩa sẵn trên [Marketplace](https://github.com/marketplace?type=actions)

**🪝Tham khảo**:

- [Tạo một action](https://docs.github.com/en/actions/creating-actions/about-custom-actions)

## Định nghĩa workflow

### Các trường dữ liệu

| Trường     | Mô tả                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `name`     | Tên để hiển thị trong tab **Actions**, mặc định là đường dẫn                                                                                                                                                                                                                                                                                                                                     |
| `run-name` | Tên hiển thị trong danh sách kết quả chạy workflows, mặc định sử dụng thông tin liên quan đến event                                                                                                                                                                                                                                                                                              |
| `on`       | Một hay nhiều sự kiện kích hoạt, cùng với các tham số mô tả cụ thể hoặc thời gian lên lịch                                                                                                                                                                                                                                                                                                       |
| `env`      | Định nghĩa ENV dùng chung cho toàn bộ các **jobs**.<br>_Có thể định nghĩa trong `jobs.<job_id>.env` hoặc `jobs.<job_id>.steps.env` (với mức độ ưu tiên tăng dần)_                                                                                                                                                                                                                                |
| `defaults` | Các cấu hình mặc định cho các **jobs**. <br>_Có thể định nghĩa trong `jobs.<job_id>.defaults`_ <br><br>- [`defaults.run.shell`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#defaultsrunshell): Định nghĩa shell để thực thi các scripts.<br>- **`defaults.run.working-directory`**: Định nghĩa thư mục làm việc hiện tại cho việc thực thi các scripts |
| `jobs`     | [Xem thêm](#định-nghĩa-jobs)                                                                                                                                                                                                                                                                                                                                                                     |

**🪝Tham khảo**:

- [Cú pháp](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

### Định nghĩa Jobs

Danh sách Jobs được định nghĩa qua key: `jobs`, mỗi job được định nghĩa bằng định danh `jobs.<job_id>`.

| Trường         | Mô tả                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `name`         | Tên hiển thị                                                                                                                         |
| `needs`        | Xác định sự phụ thuộc vào jobs khác (cần hoàn thành trước khi tiếp tục)                                                              |
| `if`           | Điều kiện cần thỏa mãn để thực thi job                                                                                               |
| `run-on`       | Xác định runner. [Xem thêm](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on) |
| `steps`        | Xác định các steps. [Xem thêm](#định-nghĩa-steps)                                                                                    |
| `uses`, `with` | Xác định [Action](#action) cùng với inputs cho nó                                                                                    |

### Định nghĩa steps

Một step có thể được định nghĩa bằng shell commands, scripts hoặc actions. Các steps của một job, mặc dù được chạy trên cùng runner (có thể truy cập vào cùng thư mục làm việc và hệ thống tệp), nhưng chạy trên các tiến trình khác nhau.

| Trường                              | Mô tả                                                                              |
| ----------------------------------- | ---------------------------------------------------------------------------------- |
| `name`                              | Tên hiển thị                                                                       |
| `if`                                | Điều kiện cần thỏa mãn để thực thi step                                            |
| `uses`, `with`                      | Xác định [Action](#action) cùng với inputs cho nó                                  |
| `run`, `shell`, `working-directory` | Thực thi shell command và scripts, cùng với cấu hình cho shell và thư mục làm việc |
|                                     |                                                                                    |

### Context

Context để chỉ các đối tượng lưu thông tin như runner, jobs, steps,... có thể truy cập bên trong workflow.

Giá trị của một biến lưu trong một context có thể truy cập qua một Expression

```yaml
${{ <context>.<variable> }}
```

**Tham khảo**:

- [Sử dụng context](https://docs.github.com/en/actions/learn-github-actions/contexts)

## Bảo mật

Trong quá trình định nghĩa workflow có thể cần sử dụng các thông tin cần giữ bí mật và không thể để trực tiếp trong thư mục repository.

Secrets là các biến được định nghĩa bên trong organization và repository giúp lưu các giá trị cần bảo mật sử dụng bên trong GitHub Action workflow. GitHub Action chỉ có thể đọc giá trị của một biến khi nó được tham chiếu bên trong workflow.

Bên trong workflow, secret có thể sử dụng trong một biến môi trường (`env`) hoặc inputs cho các actions (`with`). Các biến trong secrets có thể truy cập qua **`secrets`** context.

### GITHUB_TOKEN

GITHUB_TOKEN là một access token đặc biệt được sinh tự động để GitHub Action có thể truy cập các dịch vụ của GitHub. Mỗi khi GitHub Action job chạy, GitHub thực hiện sinh một token mới và gán cho biến `secrets.GITHUB_TOKEN`. Khi job thực thi xong (hoặc sau 24 giờ), token này sẽ hết hạn.

> [!info]
>
> Có thể truy cập GITHUB_TOKEN qua biến **`github.token`**

Mặc định, GITHUB_TOKEN được cấp những quyền sau: [GITHUB_TOKEN permissions](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)

Ta nên giới hạn các quyền này xuống tối thiểu bằng việc khai báo `permissions` cho workflow hoặc job.

```yaml
permissions:
  actions: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: read|write|none
  issues: read|write|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none
  repository-projects: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```
