---
  title:  Project Automation with GitHub Actions - Part 3
  draft: false
  date: 2024-07-19
  tags:
    - github
    - example
    - project-management
    - series-github-automation
  description: A detail example for GitHub Action workflow
---

### ⚡ Vấn đề

> 🐘 **_Trong quá trình quản lý dự án, các issues (tasks) thường phải chia nhỏ để dễ quản lý tiến độ tương ứng với mỗi pha phát triển như triển khai hay kiểm thử. Với một công việc lặp đi lặp lại như vậy, có cách nào để giảm công sức?_** 🤑

Nếu phần 2 giải quyết được các tác vụ chính của vấn đề được đề cập thì phần 3 này sẽ giải quyết phần chức năng nhỏ nhưng khó nhất. Đó là **thêm các tasks con vào GitHub Project** để quản lý.

Thực tế, [`github issue create` và `github issue edit`](https://cli.github.com/manual/gh_issue_create) đề cập trong phần 2 đều có thể thêm task vào Project, với flag `--project`. Tuy nhiên, chức năng của các lệnh này chỉ dừng lại ở đó 🍽, trong khi ta cũng muốn cập nhật các trường dữ liệu cho Project 🍝, ví dụ như `Iteration`, `Start date`,... Cho nên phần 3 sẽ xử lý yêu cầu này với GraphQL 🌀 Mutation.

### 📝 Truy vấn thông tin về Project

Coi như Project có một số trường dữ liệu mô tả cho các items là

| Trường          | Kiểu dữ liệu  | Mô tả                                              |
| --------------- | ------------- | -------------------------------------------------- |
| `Status`        | Single Select | Trạng thái của item: `Todo`, `In Progress`, `Done` |
| `Start Date`    | Date          | Ngày bắt đầu thực hiện                             |
| `Estimate Hour` | Number        | Số giờ ước tính để hoàn thành                      |
| `Group`         | Text          | Tên nhóm items liên quan                           |

trong đó, chỉ có `Status`, `Start Date` và `Group` là cần gán giá trị tự động:

- Giá trị cho `Status` là `Todo`
- Giá trị cho `Start Date` là ngày hôm nay.
- Giá trị cho `Group` là tiêu đề của issue cha.

> [!info]
>
> Thông tin các trường trong Project có thể kiểm tra tại mục `Settings`.

> [!info]
>
> Các kiểu dữ liệu và giá trị dùng để gán cho chúng có thể kiểm tra [tại đây](https://docs.github.com/en/graphql/reference/input-objects#projectv2fieldvalue).
>
> Ví dụ, để gán trường `Status` với giá trị `Todo` ta cần xác định ID của `Status` và của `Todo`.

Để thêm task vào Project và gán các trường dữ liệu, trước hết cần xác định ID của các nodes, bao gồm của Project, của Issue và của các trường dữ liệu.

1️⃣ _Tạo truy vấn_

Đoạn code sau đây được biến tấu lại từ [ví dụ về GitHub Automation](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/automating-projects-using-actions)

```bat
gh api graphql -f query='
  query($name: String!, $number: Int!) {
    user(login: $name){
      projectV2(number: $number) {
        id
        fields(first:20) {
          nodes {
            ... on ProjectV2Field {
              id
              name
            }
            ... on ProjectV2SingleSelectField {
              id
              name
              options {
                id
                name
              }
            }
          }
        }
      }
    }
  }' -f name=$ACCOUNT_NAME -F number=${{ env.PROJECT_NUMBER }} > project_data.json
```

Đoạn code này thực hiện truy vấn Project ID và ID của 20 trường đầu tiên trong Project cùng với các Option ID của các trường dạng Single Select. Đối số đầu vào là `username` của tài khoản sở hữu Project và Project Number (giá trị hiển thị trên URL của Project). Dữ liệu kết quả được đẩy vào tệp `project_data.json` trong môi trường thực thi của GitHub Action runner.

2️⃣ _Với dữ liệu có được, trích xuất các thông tin cần thiết_

Thông tin cần thiết ở đây bao gồm Project ID, ID của các trường `Start Date`, `Group` và `Status` và ID của option `Todo` tương ứng với giá trị mặc định cho `Status`.

Việc trích xuất dữ liệu có thể sử dụng lệnh **`jq`** và truy vấn trên tệp json vừa có.

```bat
echo 'PROJECT_ID='$(jq '.data.user.projectV2.id' project_data.json) >> $GITHUB_OUTPUT
echo 'START_DATE_FIELD_ID='$(jq '.data.user.projectV2.fields.nodes[] | select(.name== "${{ env.START_DATE_FIELD_NAME }}") | .id' project_data.json) >> $GITHUB_OUTPUT
echo 'GROUP_FIELD_ID='$(jq '.data.user.projectV2.fields.nodes[] | select(.name== "${{ env.GROUP_FIELD_NAME }}") | .id' project_data.json) >> $GITHUB_OUTPUT
echo 'STATUS_FIELD_ID='$(jq '.data.user.projectV2.fields.nodes[] | select(.name== "${{ env.STATUS_FIELD_NAME }}") | .id' project_data.json) >> $GITHUB_OUTPUT
echo 'STATUS_VALUE_ID='$(jq '.data.user.projectV2.fields.nodes[] | select(.name== "${{ env.STATUS_FIELD_NAME }}") | .options[] | select(.name== "${{ env.DEFAULT_STATUS_NAME }}") |.id' project_data.json) >> $GITHUB_OUTPUT
```

3️⃣ _Lấy thông tin ngày hôm nay_

Giá trị mặc định cho `Start Date` là ngày hôm nay, ta có thể sử dụng lệnh có sẵn trên cmd

```bat
echo "START_DATE_VALUE=$(date +"%Y-%m-%d")" >> $GITHUB_OUTPUT
```

> [!info]
>
> Giá trị cho kiểu Date trong GraphQL là chuỗi có định dạng ISO-8601.

### ➕ Thêm các issues vào project

Khi đã có dữ liệu cần thiết, cuối cùng chỉ cần gọi lệnh thêm issue vào project! 🎗

Ở đây, ta có nhiều issues cần thêm, chúng lại dùng chung dữ liệu. Thế nhưng GitHub không cung cấp lệnh nào để thêm cùng lúc. Nên ở đây, ta sẽ dùng **`matrix`**. Các bạn có thể xem thêm về Job `matrix` mà mình viết [tại đây](til/reuse-workflow-and-matrix.md).

Dữ liệu cho matrix chỉ là mảng một chiều các issue numbers.

1️⃣ Trước hết, tương tự trên, cần lấy được Node ID ứng với từng issue number

```bat
node_id="$(gh api graphql -f query='
            query($owner: String!, $repo: String!, $issue: Int!){
              user(login: $owner) {
                repository(name: $repo) {
                  issue(number: $issue) {
                    id
                  }
                }
              }
            }' -f owner="${{ env.ACCOUNT_NAME }}" -f repo="${{ env.REPOSITORY_NAME }}" -F issue=${{ matrix.issue-number }} --jq '.data.user.repository.issue.id')"
echo "Get node id of issue #${{ matrix.issue-number }}: $node_id"
echo 'ISSUE_ID='$node_id >> $GITHUB_ENV
```

Lệnh trên đây thực hiện đọc một phần tử `issue-number` trong `matrix` và sử dụng GraphQL để truy vấn Node ID của nó, kết quả sẽ được gán vào biến môi trường để dùng cho step sau.

2️⃣ Tiếp theo là thêm issue vào project

Bước này chỉ thực hiện thêm vào project mà không gán dữ liệu cho các trường trong project.

```bat
item_id="$( gh api graphql -f query='
            mutation($project:ID!, $item:ID!) {
              addProjectV2ItemById(input: {projectId: $project, contentId: $item}) {
                item {
                  id
                }
              }
            }' -f project=$PROJECT_ID -f item=$ISSUE_ID --jq '.data.addProjectV2ItemById.item.id')"
echo "Add issue #${{ matrix.issue-number }} to project: $item_id"
echo 'ITEM_ID='$item_id >> $GITHUB_ENV
```

Sau khi thêm vào, ta được một `item_id` là ID của issue bên trong project.

3️⃣ Đến đây, ta có thể gọi các lệnh GraphQL để thiết lập giá trị cho các trường trong project ứng với từng issue.

```bat
gh api graphql -f query='
    mutation (
              $project: ID!,
              $item: ID!,
              $start_date_field: ID!,
              $start_date_value: Date!,
              $status_field: ID!,
              $status_value: String!,
              $group_field: ID!,
              $group_value: String!) {
      set_start_date: updateProjectV2ItemFieldValue(
        input: {
          projectId: $project,
          itemId: $item,
          fieldId: $start_date_field,
          value: {date: $start_date_value}}
      ) {
        projectV2Item {
          id
        }
      }
      set_status: updateProjectV2ItemFieldValue(
        input: {
          projectId: $project,
          itemId: $item,
          fieldId: $status_field,
          value: {singleSelectOptionId: $status_value}}
      ) {
        projectV2Item {
          id
        }
      }
      set_group: updateProjectV2ItemFieldValue(
        input: {
          projectId: $project,
          itemId: $item,
          fieldId: $group_field,
          value: {text: $group_value}}
      ) {
        projectV2Item {
          id
        }
      }
    }' -f project=${{ env.PROJECT_ID }} -f item=$ITEM_ID -f start_date_field=${{ env.START_DATE_FIELD_ID }} -f start_date_value="${{ env.START_DATE_VALUE }}" -f status_field=${{ env.STATUS_FIELD_ID }} -f status_value=${{ env.STATUS_VALUE_ID }} -f group_field=${{ env.GROUP_FIELD_ID }} -f group_value="${{ env.GROUP_VALUE }}" --silent

```

> [!note]
>
> - Các biến môi trường trong lệnh ở step này được gán giá trị từ `outputs` từ jobs trước.
> - Riêng `GROUP_VALUE: "${{ github.event.issue.title }} #${{ github.event.issue.number }}"` lấy thông tin từ issue cha.

### 🎫 Về Permissions

Để thực hiện các lệnh GraphQL, ta cần truyền một Personal Access Token cho workflow với hai scopes sau:

- `project`: `read:project`
- `repo`: Full control of private repository (nếu repository là private)

### 📬 Tổng kết

Phần này mình đã trình bày các lệnh để triển khai tạo các Tasklists mỗi khi một issue được tạo. Đồng thời, thực hiện thêm các tasks đó vào project với một số trường gắn giá trị tự động. 🎯

Ở đây, các thông tin khác cho steps và jobs không được thể hiện cụ thể, nên các bạn linh hoạt nhé. Lấy ý tưởng là chính ⛳.

Ngoài ra, lúc triển khai, mình có thiết kế để các jobs trong Phần 3 vào trong một [reusable workflow](til/reuse-workflow-and-matrix.md) và gọi nó trong workflow tạo tasklists.

Một điểm nhỏ nữa là sự khác biệt khi gọi GraphQL cho Account cá nhân và Account tổ chức. Các bạn có thể thay thế các từ `user` thành `organization` trong các lệnh query là ổn rồi. Hoặc có thể dynamic nó bằng biến môi trường sau:

```bat
QUERY_LEVEL: ${{ github.event.repository.owner.type == 'User' && 'user' || 'organization' }}
```

Nó lấy thông tin về Account có sẵn từ `github.event` object để thiết lập level cho các lệnh query.
