---
  title:  Project Automation with GitHub Actions - Part 2
  draft: false
  date: 2024-05-29
  tags:
    - github
    - example
    - project-management
    - series-github-automation
  description: A detail example for GitHub Action workflow
---

### ⚡ Vấn đề

> 🐘 **_Trong quá trình quản lý dự án, các issues (tasks) thường phải chia nhỏ để dễ quản lý tiến độ tương ứng với mỗi pha phát triển như triển khai hay kiểm thử. Với một công việc lặp đi lặp lại như vậy, có cách nào để giảm công sức?_** 🤑

Tiếp nối kết quả phân tích ở trước, phần này mình sẽ ném ý tưởng vào code. Như đã nói, code toàn command lines nên siêu hay nhé 😂.

**Đầu tiên sẽ là các bước cập nhật trên 📋 repository.**

### 📝 Tạo các tasks con

Để tạo task con, sử dụng [lệnh của GitHub CLI](https://cli.github.com/manual/gh_issue_create):

```bat
gh issue create --title "<title>" --body "<body>" --label "<label>s" --assignee "<assignee>s" --repo "<repository>"
```

Nhiệm vụ hiện tại là lấy dữ liệu cho các trường này. Như đã phân tích ở trên, các issues khác nhau có danh sách tasks con khác nhau, nên thông tin về các tasks sẽ được đẩy vào cùng một tệp, vấn đề là dùng command line để đọc tệp thôi.

1️⃣ _Trước hết, dựa trên label của issue cha, mình tìm kiếm tệp định nghĩa các tasks tương ứng_

```bat
for type in $ISSUE_LABELS; do
  file="./.github/workflows/tasks-$(echo $type | sed 's/.*/\L&/').txt"
  if [ -f $file ]; then
    :: if file exists
    while read line; do
      :: each $line represents one task
    done < $file
    :: done with reading file
  fi
done
```

Ở đây, `ISSUE_LABELS` là biến môi trường được định nghĩa là danh sách các labels của issue cha nối với nhau bởi ký tự space (theo đúng format của cmd list). Đồng thời mình cũng định nghĩa luôn `ISSUE_NUMBER` cho issue cha, vì nó được dùng lại nhiều lần. Cả hai biến môi trường này đều lấy dữ liệu từ **`github.event.issue`**.

```yml
env:
  ISSUE_NUMBER: ${{ github.event.issue.number }}
  ISSUE_LABELS: ${{ join(github.event.issue.labels.*.name, ' ') }}
```

> [!info]
>
> Ở đây mình dùng [Object Filter](https://docs.github.com/en/actions/learn-github-actions/expressions#object-filters) để lấy các trường bên trong một mảng.
>
> Cụ thể mình chỉ lấy `name` cho toàn bộ các `labels` của issue cha.

2️⃣ _Lệnh đọc tệp trên có một khuyết điểm là nó tự động bỏ qua dòng cuối cùng_

Để giải quyết vấn đề này, với phương châm "Không bao giờ đặt niềm tin vào người dùng", mình sẽ yêu cầu tệp phải kết thúc bằng `====`, nếu sau nó còn nội dung cũng không đọc tiếp nữa.

```bat
:: each $line represents one task
if [[ $line == =* ]]; then
  break
fi
:: extract line that is not in pattern "=*"
```

3️⃣ _Mỗi dòng tương ứng với thông tin một task, có thể sử dụng `cut` để tách biệt từng phần, các phần phân tách nhau với ký tự `|`_

```bat
:: extract line that is not in pattern "=*"
line="$line|"

title="#$ISSUE_NUMBER $(echo $line | cut -d'|' -f1)"
body="$(echo $line | cut -d'|' -f2)"
extra_labels="$(echo $line | cut -d'|' -f3)"
if [ -n "$extra_labels" ]; then
  labels="$SUB_ISSUE_LABELS,$extra_labels"
else
  labels="$SUB_ISSUE_LABELS"
fi
assignees="$(echo $line | cut -d'|' -f4 )"
:: use extracted fields
```

`title` ở đây mình thêm issue number của issue cha vào phía trước. Task con cũng sẽ mặc định được gắn một label là **`Subtask`** (gán cho biến môi trường **`SUB_ISSUE_LABELS`**).

> [!note]
>
> - Mình thêm `|` vào sau `$line` để đảm bảo việc tách các phần sẽ trả về kết quả `""` nếu phần đó rỗng. Nếu không, với dữ liệu `line=ABC` thì tất cả các trường đều là `ABC`.
> - `labels` là tên các nhãn cần gán, đảm bảo không có khoảng trắng hay `,` thừa
> - `assignees` có thể rỗng.

> [!note]
>
> Do là tự động nên không nhất thiết quá quan trọng độ chính xác, vì sau này cũng sẽ cần cập nhật thủ công, nhiệm vụ chỉ là điền các giá trị quan trọng và chung nhất có thể.

4️⃣ _Có dữ liệu, ta có thể gọi GitHub CLI_

```bat
:: use extracted fields
url="$(gh issue create --title "$title" --body "$body" --label "$labels" --assignee "$assignees" --repo $GITHUB_REPOSITORY)"
number=$(echo $url | sed 's|.*/issues/\([0-9]\+\)$|\1|')
:: created subissue's url or number is available
```

Kết quả của lệnh tạo cần được lưu lại để nhúng vào Tasklist của issue cha. Ở đây có thể dùng URL (kết quả trả về từ CLI), nhưng mình sẽ tách phần số từ URL ra (đó cũng là issue number của task con), lý do là dùng để query GraphQL.

### 🔗 Thêm liên kết tới chúng vào issue cha

Sau khi tạo các tasks con, mình cần tham chiếu chúng vào issue cha, kiểu như

```markdown
<!-- Parent Issue's Body -->

## Planning

- [] #122 <!-- Feasibility -->
- [] #123 <!-- Implementation -->
- [] #124 <!-- Testing -->
- [] #125 <!-- UAT Feedback -->
```

1️⃣ _Mỗi lần tạo xong một task, mình có thể nối luôn number (hoặc url) của nó theo template như trên_

```bat
:: if file exist
tasklists=""
:::::::::::::::::::::::::::::
:: created subissue's url or number is available
tasklist="$tasklist\n- [ ] #$number"
```

2️⃣ _Có danh sách tasklists theo template, tiếp theo cần nhúng kết quả này cho `GITHUB_ENV` để dùng cho steps sau_

```bat
:: done with reading file
{
      echo 'TASKLIST<<EOF'
      echo -e $tasklist
      echo EOF
} >> "$GITHUB_ENV"
```

3️⃣ _Chú ý ở đây mình dùng `echo -e` vì muốn ký tự `\n` sẽ được hiển thị như ký tự xuống dòng_

> [!info]
>
> `echo -e` enable interpretation of backslash escapes
>
> If -e is in effect, the following sequences are recognized:
>
> - \\ backslash
> - \a alert (BEL)
> - \b backspace
> - \c produce no further output
> - \e escape
> - \f form feed
> - \n new line
> - \r carriage return
> - \t horizontal tab
> - \v vertical tab
> - \0NNN byte with octal value NNN (1 to 3 digits)
> - \xHH byte with hexadecimal value HH (1 to 2 digits)

Khi đó, nội dung của biến môi trường `TASKLIST` sẽ được trải dài trên nhiều dòng, nên mình dùng cú pháp của [Multiline String](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#multiline-strings)

4️⃣ _Có nội dung phần **Planning**, mình cần append nó vào body của issue cha. Tạo một biến môi trường cho toàn bộ phần body, cả cũ lẫn mới này_

```yaml
env:
  NEW_BODY: "${{ github.event.issue.body }}\n${{ env.TASKLIST_HEADING }}\n${{ env.TASKLIST }}"
```

Ở đây, `TASKLIST_HEADING` được định nghĩa là `## Planning` tương tự template đã đề cập ở đầu mục này.

> [!note]
>
> Ký tự `\n` trong biến môi trường định nghĩa bởi GitHub Action sẽ tự động được coi như newline, thay vì bị escape như trong cmd.

5️⃣ _Ok, đến đây chỉ cần gọi [GitHub CLI](https://cli.github.com/manual/gh_issue_edit)_

```bat
gh issue edit $ISSUE_NUMBER --body "$NEW_BODY" --repo $GITHUB_REPOSITORY
```

> [!info]
>
> GitHub CLI không có command nào để append nội dung cho issue nên mình phải thêm thủ công.

### 📬 Tổng kết

Với hai bước ở trên, về cơ bản, nghiệp vụ chính của vấn đề đã hoàn thành: tự động tạo các tasks con và thêm chúng vào issue cha. Tuy nhiên, một phần rất quan trọng khác để job hiện tại chạy được là **permission**.

🎫 Các lệnh GitHub CLI được sử dụng ở trên yêu cầu phải xác định biến môi trường **`GH_TOKEN`** để chạy được. Có thể truyền PAT của tài khoản thực để chỉ GitHub workflow hiện tại chạy thay cho mình (khi hiển thị, người tạo các issues sẽ là tài khoản thực). Tuy nhiên ở đây mình dùng `GITHUB_TOKEN`, token sinh tự động bởi GitHub Action (Bot)

```yml
env:
  GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Ngoài ra các lệnh GitHub CLI trên yêu cầu token phải được cấp các quyền

```yml
permissions:
  issues: write # for creating new tasks and update parent issue
  contents: read # for access repository's content
```

🥁 Phần 2 đến đây là kết thúc. Nếu vẫn còn bất ngờ vì kết thúc sớm quá 🪫, hãy đến với [Phần 3](#) nhé [🔋](til/project-automation-with-github-action-p3.md).
