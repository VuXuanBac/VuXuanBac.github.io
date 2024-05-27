---
  title: Reusable Workflow and Job Matrix
  draft: false
  date: 2024-05-27
  tags:
    - github
    - ci-cd
  description: How to reuse jobs defined in one workflow in another? How to run one jobs many times with different inputs?
---

Tham khảo:

- [Ứng dụng giúp kiểm tra cú pháp của workflow](https://rhysd.github.io/actionlint/)

Reusable workflow cho phép tách một workflow ra các phần nhỏ giúp dễ bảo trì và có thể dùng lại. Một workflow có thể gọi một reused workflow (_called workflow_) từ cùng repository hoặc từ một repository có thể truy cập được.

Phạm vi truy cập các đối tượng ở hai workflows này như sau:

- **`github`** context tương ứng ánh xạ tới caller.
- called có thể truy cập vào `github.token` và `secrets.GITHUB_TOKEN` của caller.
- Các biến môi trường **`env`** là độc lập giữa hai workflows. Biến dùng chung giữa các workflows chỉ có thể định nghĩa ở mức _repository_, _organization_ hoặc _environment_.

## ♻ Tạo reusable workflow

Cú pháp của reusable workflow tương tự một workflow thông thường, chỉ khác là nó sử dụng một sự kiện đặc biệt **`workflow_call`**.

### 📥 Dữ liệu đầu vào

Reusable workflow có thể nhận vào các đối số, song cần định nghĩa đối số cũng như các giá trị `secrets` mong muốn nhận vào thông qua các trường **`on.workflow_call.inputs`** và **`on.workflow_call.secrets`**

Mỗi trường trong `inputs` tương ứng với tên của giá trị đầu vào, nội dung của trường này xác định các thông tin mô tả như `description`, `type`, `required` và `default`. Trong đó `type` chỉ có thể là **boolean, number, string**.

Tương tự với `secrets`

```YAML
on:
  workflow_call:
    inputs:
      config-path:
        required: true
        type: string
    secrets:
      envPAT:
        required: true
```

Sau khi đã định nghĩa các đối số cho called workflow trên trường `on`, ta có thể tham chiếu các đối số bên trong các jobs của nó thông qua [**`inputs`** context](https://docs.github.com/en/actions/learn-github-actions/contexts#inputs-context).

### 📤 Kết quả trả về

Reusable workflow có thể trả về giá trị cho workflow gọi nó. Tương tự, ta cũng cần khai báo các đối số trả về, với thông tin mô tả là `description` và `value`. Trong đó `value` lấy giá trị từ kết quả trả về của một job trong called workflow.

```YAML
name: Reusable workflow

on:
  workflow_call:
    # Map the workflow outputs to job outputs
    outputs:
      firstword:
        description: "The first output string"
        value: ${{ jobs.example_job.outputs.output1 }}
      secondword:
        description: "The second output string"
        value: ${{ jobs.example_job.outputs.output2 }}

jobs:
  example_job:
    name: Generate output
    runs-on: ubuntu-latest
    # Map the job outputs to step outputs
    outputs:
      output1: ${{ steps.step1.outputs.firstword }}
      output2: ${{ steps.step2.outputs.secondword }}
    steps:
      - id: step1
        run: echo "firstword=hello" >> $GITHUB_OUTPUT
      - id: step2
        run: echo "secondword=world" >> $GITHUB_OUTPUT

```

## 📞 Gọi reusable workflow

Bên trong caller workflow, ta gọi một reusable workflow qua trường **`jobs.<job-id>.uses`** và chỉ ở mức job. Giá trị của trường này là đường dẫn tham chiếu tới workflow đó (tương tự với [action](git/github-action.md#action)).

> [!note]
>
> Do called workflow được gọi ở mức _job_ nên các biến ở phạm vi _step_ như [biến môi trường mặc định](git/github-action.md#biến-môi-trường-mặc-định)

Để truyền đối số cho called workflow, ta sử dụng trường **`jobs.<job-id>.with`** và **`jobs.<job-id>.secrets`**.

> [!note]
>
> Giá trị cho **`jobs.<job-id>.secrets`** có thể là `inherit` để truyền toàn bộ các secrets từ caller workflow.

Về giá trị trả về, caller workflow có thể truy xuất từ các **jobs phía sau**

```YAML
jobs:
  job1:
    uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
    with:
      config-path: .github/labeler.yml
    secrets: inherit

  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - run: echo ${{ needs.job1.outputs.output1 }} ${{ needs.job1.outputs.output2 }}
```

## 🧮 Sử dụng job `matrix`

Job matrix giúp tạo các biến cho một job và kích hoạt job chạy lại nhiều lần tương ứng với mỗi tổ hợp giá trị của các biến.

Matrix cho một job được định nghĩa qua **`jobs.<job-id>.strategy.matrix.*`**. Mỗi trường trong `matrix` tương ứng một biến, tổ hợp tất cả giá trị giữa các biến sẽ tương ứng một lần chạy cho job.

> [!note]
>
> Giá trị của các biến có thể là mảng của các objects

Ta có thể thêm hoặc bớt tổ hợp trong matrix qua hai trường `include` và `exclude`

Với mỗi lần chạy, giá trị của các biến trong tổ hợp hiện tại được truy xuất qua [`matrix` context](https://docs.github.com/en/actions/learn-github-actions/contexts#matrix-context)

```YAML
jobs:
  example_matrix:
    strategy:
      matrix:
        version: [10, 12, 14]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

**🪝Tham khảo**:

- [Thêm tổ hợp](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs#expanding-or-adding-matrix-configurations)
- [Bớt tổ hợp](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs#excluding-matrix-configurations)

### 🪜 `matrix` cho reusable workflow

Ta có thể sử dụng job `matrix` để truyền các tổ hợp đối số cho việc gọi reusable workflow.

```YAML
jobs:
  ReuseableMatrixJobForDeployment:
    strategy:
      matrix:
        target: [dev, stage, prod]
    uses: octocat/octo-repo/.github/workflows/deployment.yml@main
    with:
      target: ${{ matrix.target }}
```

> [!important]
>
> Khi chạy job với `matrix`, giá trị đầu ra của job chỉ là kết quả **chạy thành công cuối cùng**.
