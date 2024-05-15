---
  title: Branch
  draft: false
  date: 2024-05-15
  tags:
    - git
  description: What is Branch and Remote Branch
  aliases:
---

**Branch** cung cấp một bối cảnh tách biệt và an toàn cho quá trình cập nhật dự án. Trong thực tế, branch thường được ứng dụng trong một số trường hợp:

- Thử nghiệm tính năng mới -> Dễ dàng khôi phục
- Sửa lỗi -> An toàn, kiểm tra tránh gây thêm lỗi khi sửa lỗi cũ
- Đánh dấu một giai đoạn -> develop, staging, production

## Cách hoạt động

Khi thực hiện staging, Git tính toán checksum cho mỗi tệp, lưu phiên bản mới của tệp vào trong **.git** (mỗi tệp được gọi là một blob) và thêm checksum vào vùng staging. Khi commit các tệp staged, Git tính toán checksum cho mỗi thư mục trong cây thư mục chứa các tệp đó và tạo một commit object trỏ tới gốc của cây cùng với các metadata của commit.

![Commit, Tree and Blob objects](assets/git/commit-tree-blob-object.png)

Một trong các dữ liệu được lưu trong commit object là các con trỏ trỏ tới các commit cha (snapshot ngay trước commit hiện tại). Số lượng commits cha có thể là 0 (initial commit), 1 (normal commit) hoặc nhiều (merge)

![Commits and Parents](assets/git/commit-parent.png)

Như vậy chỉ cần trỏ tới một commit, ta có thể theo dấu được các commits phía trước nó. Branch chính là con trỏ này, trỏ vào một commit và tự động di chuyển về sau khi thêm mới commits.

> Khi khởi tạo một repository, Git tự động tạo một branch với tên **master**

Thực tế, dữ liệu lưu trong một branch chỉ là **mã checksum của commit** nó trỏ tới.

> Danh sách các branches và dữ liệu của chúng được lưu trong các tệp trong **.git/refs/heads**
> Khi tên branch ở dạng đường dẫn, ví dụ: `feature/login`, thì tệp tương ứng sẽ được tổ chức theo thư mục.

Để xác định branch đang làm việc (phiên bản snapshot đang tạo các thay đổi), Git sử dụng con trỏ **HEAD**, một con trỏ trỏ vào một branch, branch này gọi là **working branch**.

> Giá trị con trỏ này được lưu trong tệp **.git/HEAD**: lưu theo đường dẫn

Khi di chuyển giữa các branches, **HEAD** sẽ thay đổi giá trị, trỏ tới branch tương ứng, đồng thời Git thay đổi nội dung dự án tương ứng với commit mà branch đó trỏ tới.

> **HEAD** có thể không trỏ tới một commit, chế độ này được gọi là **Detach HEAD**

## Một số lệnh cơ bản

> Tạm gọi hai branches là _**cùng luồng**_ nếu hai commits tương ứng có quan hệ cha-con.

| Lệnh                                                            | Mô tả                                                               |
| --------------------------------------------------------------- | ------------------------------------------------------------------- |
| `git branch <name> [<commit>]`                                  | Tạo một branch trỏ tới một commit (mặc định là commit hiện tại)     |
| `git switch <name>`<br />`git checkout <name>`                  | Di chuyển con trỏ HEAD tới một branch                               |
| `git switch --detach <checksum>`<br />`git checkout <checksum>` | Di chuyển con trỏ HEAD tới một commit (Detach HEAD)                 |
| `git branch -d <name>`                                          | Xóa branch (xóa con trỏ) nằm trên **cùng luồng** với working branch |
| `git branch -m <name> [<new-name>]`                             | Đổi tên cho branch hiện tại hoặc một branch khác                    |
| `git branch -v`                                                 | Xem danh sách branches và commit trỏ tới                            |
| `git branch --merged`                                           | Danh sách branches nằm trên **cùng luồng** với working branch       |

## Remote Branch

Remote repository cũng có các branches. Phiên bản cục bộ của các branches này (được gọi là **remote-tracking branch**, tạm gọi **remote branch**) được đặt tên theo dạng **`<remote-name>/<branch-name>`**.

Remote branch dùng để **theo dấu các thay đổi diễn ra trên remote repository**, do đó chúng không được thay đổi thủ công như các branches tạo cục bộ mà được thay đổi qua các thao tác đồng bộ.

Khi muốn tiếp tục cập nhật thay đổi trên một remote branch, ta cần tạo một branch cục bộ ánh xạ tới nó (hỗ trợ quá trình đồng bộ giữa local và remote repository), các branches ánh xạ này gọi là **tracking branch**es.

> Khi thực hiện `git clone`, mặc định có tracking branch `master` cho remote branch `origin/master` > _Tên branch `master` và tên remote `origin` có thể khác biệt._

Lệnh để tạo tracking branch:

```
git branch <tracking-branch-name> <remote-branch-name>
```

> Lệnh này tương tự tạo các branches cục bộ khác, và `remote-branch-name` giúp xác định commit cần trỏ tới.

Để xem danh sách các branches cục bộ cùng với remote branches tương ứng, sử dụng lệnh:

```
git branch -vv
```

### 👇 `git fetch`

`git fetch` giúp tải về các đối tượng dữ liệu từ remote repositories, đồng thời cập nhật các remote branches trên repository cục bộ.

```
git fetch <remote>
```

Lệnh trên thực hiện cập nhật dữ liệu cho tất cả các remote branches chưa đồng bộ với repository cục bộ.

Do chỉ thực hiện tải về nên kết quả của lệnh có thể làm cho tracking branch và remote branch phân nhánh. Khi so sánh giữa tracking branch và remote branch, có thể có khác biệt:

- **Ahead**: Tracking branch đang có một số commits mà remote branch chưa có.
  - Để đồng bộ, thực hiện lệnh [`git push`](#🫸-git-push)
- **Behind**: Tracking branch đang thiếu một số commits trong remote branch.
  - Để đồng bộ, thực hiện lệnh [`git merge`](./merge-rebase.md)
- Có thể có cả Ahead và Behind

```
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] Add forgotten brackets
  master    1ae2a45 [origin/master] Deploy index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] This should do it
  testing   5ea463a Try something new
```

### 🫷 `git pull`

Nếu sử dụng `git fetch` có thể dẫn đến sự phân nhánh giữa tracking và remote branch thì `git pull` sẽ thực hiện một số thao tác để xử lý sự phân nhánh này.

```
git pull <remote>
```

Lệnh này thực hiện tải dữ liệu cho toàn bộ các remote branches và `git merge` remote branch ứng với branch hiện tại vào chính nó. Tức là tương đương với hai lệnh

```
git fetch <remote>
git merge <remote>/HEAD
```

`git pull` khác với `git fetch` ở chỗ nó làm thay đổi cấu trúc lịch sử thay đổi (vì tự động thực hiện Merging)

### 🫸 `git push`

`git push` giúp đồng bộ dữ liệu từ repository cục bộ tới remote repository. Thông thường ta sử dụng lệnh

```
git push <remote> <local-branch>[:<remote-branch>]
```

Lệnh này thực hiện cập nhật dữ liệu từ `local-branch` tới `remote-branch` của remote repository. Nếu `local-branch` chưa theo dấu một remote branch nào thì một remote branch sẽ được tạo.

Ta có thể sử dụng `git push` để thực hiện các thao tác cập nhật remote branches

```
#### Delete remote branch ####
git push <remote> -d <remote-branch>

#### Rename remote branch ####
# Rename tracking branch
git branch -m <old-name> <new-name>
# Delete old remote branch and push new local branch
git push <remote> :<old-name> <new-name>
# Reset upstream branch for new local branch
git push origin -u <new-name>
```
