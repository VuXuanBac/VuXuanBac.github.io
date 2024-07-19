---
  title:  GitHub Action
  draft: false
  date: 2024-05-21
  tags:
    - github
    - ci-cd
  description: Using GitHub Action to automating jobs
  aliases:
---

GitHub Action là một nền tảng CI/CD hỗ trợ tự động hóa quá trình build, test và deployment và các tác vụ khác bằng việc tự động thực thi một **workflow** người dùng định nghĩa khi một **sự kiện** xảy ra trên **repository**.

## Các khái niệm

### 🚀 Workflow

Một workflow bao gồm một hay nhiều **jobs**, các _jobs_ này có thể thực thi tuần tự hoặc song song trong một **runner** riêng.

> [!note]
>
> Workflow được định nghĩa trong một tệp YAML nằm trong thư mục **.github/workflows** của repository.

### 🔥 Sự kiện

Các workflows được thực thi tự động khi một sự kiện xảy ra, bao gồm:

- Sự kiện trên repository hiện tại
- Sự kiện bên ngoài GitHub nhưng kích hoạt sự kiện `repository_dispatch` trong GitHub.
- Thời gian lên lịch trước
- Tạo thủ công

Một sự kiện được định nghĩa bằng một hành động (**`on`**) và cụ thể hóa qua một hay nhiều kiểu (**`types`**) cũng như các tiêu chí lọc (chỉ một số sự kiện như `push`, `pull-request` có thể xác định tiêu chí lọc - xem tại [cú pháp workflow](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)).

**🪝Tham khảo**:

- [Kích hoạt một workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)
- [Danh sách các sự kiện](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Dữ liệu của sự kiện](https://docs.github.com/en/webhooks/webhook-events-and-payloads)

### 🧨 Job

Job là một thành phần tạo nên một _workflow_, nó bao gồm nhiều **steps** và các _steps_ này được chạy trên cùng một **runner** (có thể truy cập vào cùng thư mục làm việc và hệ thống tệp), nhưng chạy trên các tiến trình khác nhau.

Một _step_ có thể được định nghĩa bằng shell commands, scripts hoặc [actions](#action).

Có thể chỉ định điều kiện để thực thi một _job_ hoặc một _step_ (**`if`**).

Các _jobs_ mặc định là **độc lập với nhau**, khi đó chúng có thể chạy song song với nhau. Song ta có thể chỉ định sự phụ thuộc cho chúng (**`needs`**), đảm bảo chúng được chạy tuần tự.

### 🏃 Runner

Runner là một server dùng để chạy các _jobs_, tại một thời điểm, một _runner_ chỉ chạy một _job_.

GitHub cung cấp sẵn các máy ảo Ubuntu, Windows, macOS để làm _runners_.

Ngoài ra có thể chỉ định một runner bên ngoài (self-host runner).

**🪝Tham khảo**:

- [Lựa chọn runner](https://docs.github.com/en/actions/using-jobs/choosing-the-runner-for-a-job)

### 🕯Action

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

## Biến, biểu thức và dữ liệu

### 🍪 Biến môi trường

Biến môi trường giúp chia sẻ dữ liệu bên trong steps, jobs, workflows hay giữa các workflows. Việc sử dụng biến môi trường lưu dữ liệu người dùng cung cấp cũng giúp tăng cường bảo vệ hệ thống khỏi các tấn công injections.

Các mức có thể định nghĩa biến môi trường:

- **Chia sẻ giữa các workflows**: Định nghĩa bên trong repository, organization hoặc environment
- **Chia sẻ bên trong một workflow**: Định nghĩa qua trường **`env`** (mức tệp)
- **Chia sẻ bên trong một job**: Định nghĩa qua trường **`env`** (mức **`jobs.<job-id>`**)
- **Chia sẻ bên trong một step**: Định nghĩa qua trường **`env`** (mức **`jobs.<job-id>.steps[*]`**)

Dữ liệu của biến môi trường có thể truy cập qua:

- [**`env`** Context](#context)
- Bên trong runner. VD: **`$ISSUE_NUMBER`** cho Linux và **`$env:ISSUE_NUMBER`** cho Windows

> [!important]
>
> - Chỉ có các lệnh **`run`** bên trong các steps, cũng như các [action](#action) là được xử lý bởi runner, nên mới có thể sử dụng cú pháp thứ hai.
> - Ở các phạm vi khác chỉ có thể sử dụng cú pháp thứ nhất, và chúng sẽ được xử lý bởi GitHub Action.
> - Vì GitHub Action xử lý trước runner nên có thể sử dụng Context bên trong **`run`**

### 🍩 Biến môi trường mặc định

Đây là các biến môi trường được tạo và gán giá trị bởi GitHub, **chỉ có phạm vi bên trong runner (tức là chia sẻ giữa các steps)** và **hầu hết không thể thay đổi giá trị**.

Trong suốt quá trình thực thi một workflow, runners có thể sinh các tệp tạm thời, đường dẫn tới các tệp này có thể truy xuất qua các biến môi trường mặc định.

Dưới đây là một số usecases

1️⃣ Khi một step sinh dữ liệu cho các steps phía sau, có thể gán dữ liệu đó cho một biến môi trường bằng việc thêm nội dung cho tệp định nghĩa bởi **`GITHUB_ENV`**

VD:

```cmd
echo 'DATA="Hello World"'$(echo 'World') >> $GITHUB_ENV
```

Trong các steps phía sau, có thể truy xuất dữ liệu qua `$DATA`

2️⃣ Một step có thể trả về output bằng việc ghi nội dung vào tệp trỏ bởi **`GITHUB_OUTPUT`** và bên trong job có thể tham chiếu output này qua **`id`** của step.

VD:

```YAML
- name: Set color
  id: color-selector
  run: echo "SELECTED_COLOR=green" >> "$GITHUB_OUTPUT"
- name: Get color
  env:
    SELECTED_COLOR: ${{ steps.color-selector.outputs.SELECTED_COLOR }}
  run: echo "The selected color is $SELECTED_COLOR"
```

**🪝Tham khảo**:

- [Các biến môi trường mặc định](https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables)
- [Các tệp môi trường](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#environment-files)

### 🔧 Biểu thức

GitHub Action có thể xử lý các biểu thức và tính toán trước giá trị cần sử dụng. Cú pháp của biểu thức:

```
${{ <expression> }}
```

Giá trị của biểu thức hỗ trợ các kiểu dữ liệu: boolean, null, number và string. Có thể sử dụng các toán tử như `&&, ||, [],...`

> [!note]
>
> string bên trong biểu thức phải được bao bằng kí tự `'`, nếu nội dung cũng chứa kí tự này, có thể escape thành `''`.

#### Hàm

Đặc biệt, GitHub cung cấp sẵn một số hàm tiện ích giúp xử lý dữ liệu:

| Hàm                                   | Mô tả                                                         |
| ------------------------------------- | ------------------------------------------------------------- |
| `contains, startsWith, endsWith`      | Kiểm tra chuỗi con, phần tử                                   |
| `format`                              | String interpolation với placeholder dạng `{N}`               |
| `join`                                | Nối các phần tử                                               |
| `fromJSON, toJSON`                    | Chuyển đổi dữ liệu về dạng JSON nếu dữ liệu xử lý phức tạp    |
| `success, always, cancelled, failure` | Trả về trạng thái của step trước, job trước hoặc của workflow |

#### Lọc

Với dữ liệu phức, có thể sử dụng toán tử `*` để lấy các trường dữ liệu bên trong một mảng mà không cần truy xuất từng phần tử cụ thể.

VD:

```JSON
vegetables = {
  "scallions":
  {
    "colors": ["green", "white", "red"],
    "ediblePortions": ["roots", "stalks"],
  },
  "beets":
  {
    "colors": ["purple", "red", "gold", "white", "pink"],
    "ediblePortions": ["roots", "stems", "leaves"],
  },
  "artichokes":
  {
    "colors": ["green", "purple", "red", "black"],
    "ediblePortions": ["hearts", "stems", "leaves"],
  },
}
```

Khi truy xuất **`vegetables.*.ediblePortions`**, ta được kết quả

```JSON
[
  ["roots", "stalks"],
  ["hearts", "stems", "leaves"],
  ["roots", "stems", "leaves"],
]
```

### 🧭 Context

Context để chỉ các đối tượng lưu thông tin về runner environment, jobs, steps,.... Context là các objects với các thuộc tính là **string** hoặc **objects**.

Giá trị của một biến lưu trong một context có thể truy cập qua một biểu thức với hai cú pháp

```
${{ <context>.<variable> }}

${{ <context>['<variable>'] }}
```

Tuy nhiên, các **context**s khác nhau chỉ giới hạn truy cập ở các trường khác nhau. [Xem thêm](https://docs.github.com/en/actions/learn-github-actions/contexts#context-availability)

**🪝Tham khảo**:

- [Danh sách contexts và thuộc tính](https://docs.github.com/en/actions/learn-github-actions/contexts)

## 🔐 Bảo mật

Trong quá trình định nghĩa workflow có thể cần sử dụng các thông tin cần giữ bí mật và không thể để trực tiếp trong thư mục repository.

Secrets là các biến được định nghĩa bên trong organization và repository giúp lưu các giá trị cần bảo mật sử dụng bên trong GitHub Action workflow. GitHub Action chỉ có thể đọc giá trị của một biến khi nó được tham chiếu bên trong workflow.

Bên trong workflow, secret có thể sử dụng trong một biến môi trường (`env`) hoặc inputs cho các actions (`with`). Các biến trong secrets có thể truy cập qua **`secrets`** context.

### 🎫 GITHUB_TOKEN

GITHUB_TOKEN là một access token đặc biệt được sinh tự động để GitHub Action có thể truy cập các dịch vụ của GitHub. Mỗi khi GitHub Action job chạy, GitHub thực hiện sinh một token mới và gán cho biến `secrets.GITHUB_TOKEN`. Khi job thực thi xong (hoặc sau 24 giờ), token này sẽ hết hạn.

> [!info]
>
> Có thể truy cập GITHUB_TOKEN qua biến **`github.token`**

> [!note]
>
> `GITHUB_ACTION` là tự động sinh bởi GitHub, nên các tác vụ sử dụng Token này sẽ hiển thị người thực hiện là **GitHub Bot**, nếu muốn hiển thị bạn là người thực hiện, hãy sinh Personal Access Token và gán cho các tác vụ cần Token.

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

### 🌱 Các cách xác thực khác

`GITHUB_TOKEN` chỉ có các quyền truy cập tài nguyên bên trong **repository**, nên để thao tác với các tài nguyên bên ngoài (như project hoặc repository khác), cần có **Personal Access Token** (thật) hoặc **GitHub App ID** (khuyến khích đối với _Organization Project_, vì Token này gắn với tổ chức thay vì cá nhân) để cấp quyền cho GitHub Action.

**🪝Tham khảo**:

- [Xác thực với GitHub App](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow)

## 🪝Tham khảo

- [Ứng dụng giúp kiểm tra cú pháp của workflow](https://rhysd.github.io/actionlint/)
- [VSCode Extension](https://marketplace.visualstudio.com/items?itemName=github.vscode-github-actions)
