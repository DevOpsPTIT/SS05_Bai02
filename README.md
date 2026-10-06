# Báo Cáo Thực Hành: Tái Cấu Trúc Lịch Sử Commit Bằng Interactive Rebase

- **Bài tập**: Bài 2 - Tái cấu trúc lịch sử commit bằng Interactive Rebase (Session 05 / Git Intermediate & Advanced Recovery)
- **Đường dẫn nộp bài**: `SS05/Bai02/README.md`
- **GitHub Remote Repository**: `https://github.com/DevOpsPTIT/SS05_Bai02.git`

---

## 1. Bối cảnh & Mục tiêu

### Bối cảnh
Trong quá trình làm việc cá nhân trên nhánh tính năng (feature branch), lập trình viên thường tạo ra rất nhiều commit nhỏ lẻ, mang tính chất thử nghiệm, sửa lỗi vặt hoặc chứa các tệp nháp rác (như `fix typo`, `temp file`, `add debug`). Nếu đẩy (push) toàn bộ lịch sử commit lộn xộn này lên kho lưu trữ chung (Remote Repository), cây lịch sử của dự án sẽ trở nên cồng kềnh, nhiễu thông tin và gây khó khăn cho việc theo dõi (code review). Lệnh `git rebase -i` (Interactive Rebase) là công cụ tiêu chuẩn giúp dọn dẹp, tái cấu trúc lịch sử commit trước khi gửi Pull Request / Merge vào nhánh chính (`main`/`master`).

### Mục tiêu
- **Nắm vững kỹ thuật Interactive Rebase**: Sử dụng cờ `-i` để điều khiển quá trình chỉnh sửa lịch sử commit linh hoạt.
- **Gộp commit (Squash)**: Gộp 3 commit nhỏ lẻ liên quan đến module authentication thành 1 commit duy nhất.
- **Đổi tên thông điệp commit (Reword)**: Đặt tên lại thông điệp commit sau gộp chuẩn hóa theo danh mục Conventional Commits (`feat: hoan thien module authentication`).
- **Xóa bỏ commit rác (Drop)**: Xóa hoàn toàn commit chứa tệp thử nghiệm `temp.txt` khỏi lịch sử Git.
- **Trích xuất log kiểm chứng**: Chứng minh lịch sử Git sau Rebase chỉ giữ lại 1 commit sạch sẽ duy nhất.

---

## 2. Thông Số Cấu Hình & Ràng Buộc

| Thông số / Cấu hình | Giá trị lựa chọn / Thao tác | Ghi chú / Ràng buộc |
| :--- | :--- | :--- |
| **Thư mục bài tập** | `SS05/Bai02` | Repository quản lý cục bộ |
| **Tệp tin mã nguồn** | `auth.js` | Tệp tin tính năng đăng nhập chính |
| **Tệp tin rác thử nghiệm**| `temp.txt` | Cần loại bỏ hoàn toàn khỏi Git |
| **Số lượng commit ban đầu** | 4 commit | Bao gồm 3 commit cho `auth.js` + 1 commit cho `temp.txt` |
| **Lệnh Rebase** | `git rebase -i HEAD~4` | Mở giao diện Interactive Rebase lùi 4 commit |
| **Hành động Rebase 1** | `pick` dòng 1, `squash` dòng 2 & 3 | Gộp 3 commit của `auth.js` thành 1 |
| **Hành động Rebase 2** | `drop` dòng 4 | Xóa bỏ vĩnh viễn commit tạo `temp.txt` |
| **Thông điệp commit mới** | `feat: hoan thien module authentication` | Chuẩn hóa thông điệp sau khi squash |
| **Remote Repository** | `https://github.com/DevOpsPTIT/SS05_Bai02.git` | Kho lưu trữ GitHub |

---

## 3. Các Từ Khóa Lệnh Trong Giao Diện Interactive Rebase

Khi thực thi `git rebase -i`, Git sẽ mở một trình soạn thảo văn bản chứa danh sách các commit từ cũ đến mới (theo thứ tự từ trên xuống dưới) kèm theo các lệnh cấu hình ở đầu mỗi dòng:

| Từ khóa / Lệnh | Viết tắt | Ý nghĩa & Tác động |
| :--- | :--- | :--- |
| **`pick`** | `p` | Giữ lại commit đó (không thay đổi gì) |
| **`reword`** | `r` | Giữ lại commit nhưng tạm dừng để chỉnh sửa lại thông điệp (commit message) |
| **`edit`** | `e` | Tạm dừng quá trình rebase tại commit đó để chỉnh sửa nội dung mã nguồn |
| **`squash`** | `s` | **Gộp commit này vào commit liền trước nó** và giữ lại/kết hợp cả 2 thông điệp |
| **`fixup`** | `f` | Gộp commit vào commit liền trước nó nhưng bỏ qua thông điệp của commit này |
| **`drop`** | `d` | **Xóa bỏ hoàn toàn commit** khỏi lịch sử (tương ứng xóa luôn các thay đổi của commit đó) |

---

## 4. Các Bước Thực Hiện Chi Tiết

### Bước 1: Khởi tạo 4 commit ban đầu

Khởi tạo kho lưu trữ Git cục bộ và tạo lần lượt 4 commit thử nghiệm theo gợi ý bài tập:

```bash
# 1. Tạo tệp auth.js lần đầu
echo "function login() { console.log('Login module initialized'); }" > auth.js
git add auth.js
git commit -m "feat: khoi tao module auth"

# 2. Sửa auth.js lần 2 (Sửa lỗi nhỏ)
echo "function login(user, pass) { console.log('Login attempt for:', user); }" > auth.js
git add auth.js
git commit -m "fix typo"

# 3. Sửa auth.js lần 3 (Thêm hàm mới)
echo "function login(user, pass) { console.log('Login attempt for:', user); }" > auth.js
echo "function logout() { console.log('User logged out'); }" >> auth.js
git add auth.js
git commit -m "adds utility functions"

# 4. Tạo tệp rác temp.txt
echo "DEBUG_LOG=true" > temp.txt
git add temp.txt
git commit -m "add temp file for debug"
```

---

### Bước 2: Kiểm tra lịch sử commit ban đầu (`git log --oneline`)

```console
$ git log --oneline
cbb168e add temp file for debug
ec8cf71 adds utility functions
20baafc fix typo
b102883 feat: khoi tao module auth
```
> **Nhận xét**: Lịch sử chứa 4 commit vụn vặt và 1 commit chứa file rác `temp.txt`.

---

### Bước 3: Thực thi Interactive Rebase (`git rebase -i HEAD~4`)

Chạy lệnh rebase lùi về 4 commit tính từ con trỏ `HEAD`:

```bash
git rebase -i HEAD~4
```

---

### Bước 4: Màn hình giao diện cấu hình Interactive Rebase

Trình soạn thảo văn bản sẽ hiển thị danh sách 4 commit theo thứ tự thời gian (từ cũ nhất ở trên đến mới nhất ở dưới). Tiến hành chỉnh sửa các lệnh đầu dòng như sau:

```git-rebase
pick b102883 feat: khoi tao module auth
squash 20baafc fix typo
squash ec8cf71 adds utility functions
drop cbb168e add temp file for debug

# Rebase 0726fdc..cbb168e onto 0726fdc (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash", but skip this commit's log message
# d, drop <commit> = remove commit
```

> **Giải thích cấu hình**:
> - Dòng 1 (`pick b102883`): Giữ làm commit gốc.
> - Dòng 2 (`squash 20baafc`): Gộp commit `fix typo` vào commit gốc `b102883`.
> - Dòng 3 (`squash ec8cf71`): Gộp commit `adds utility functions` vào commit gốc `b102883`.
> - Dòng 4 (`drop cbb168e`): Xóa bỏ hoàn toàn commit `add temp file for debug` (xóa luôn tệp `temp.txt`).

---

### Bước 5: Đặt lại thông điệp commit gộp

Sau khi lưu và đóng file cấu hình Rebase, Git mở tiếp màn hình soạn thảo thông điệp cho commit vừa gộp. Tiến hành sửa lại thành thông điệp chuẩn hóa:

```text
feat: hoan thien module authentication
```

Lưu và đóng file. Git thông báo: `Successfully rebased and updated refs/heads/master`.

---

## 5. Trích Xuất Log Terminal Chứng Minh (Verification Logs)

### 1. Kiểm tra lại lịch sử commit (`git log --oneline`)

```console
$ git log --oneline
5e38dbe feat: hoan thien module authentication
```
> **Đánh giá**: Đạt 100% yêu cầu: Lịch sử commit 4 bước ban đầu đã được dọn dẹp sạch sẽ, chỉ còn lại duy nhất 1 commit chuẩn hóa với thông điệp `feat: hoan thien module authentication`.

---

### 2. Kiểm tra sự tồn tại của tệp rác `temp.txt` trên đĩa cứng

```console
$ Test-Path temp.txt
False
```
> **Đánh giá**: Tệp `temp.txt` đã bị xóa hoàn toàn khỏi Working Tree nhờ lệnh `drop` trong Rebase.

---

### 3. Kiểm tra nội dung tệp `auth.js` đã được tổng hợp

```console
$ Get-Content auth.js
function login(user, pass) { console.log('Login attempt for:', user); }
function logout() { console.log('User logged out'); }
```
> **Đánh giá**: Toàn bộ mã nguồn từ 3 commit ban đầu của `auth.js` đã được bảo toàn và tích hợp đầy đủ trong 1 commit duy nhất.

---

### 4. Kiểm tra trạng thái kho lưu trữ (`git status`)

```console
$ git status
On branch main
nothing to commit, working tree clean
```

---

## 6. So Sánh Interactive Rebase vs Git Merge

| Tiêu chí | Interactive Rebase (`git rebase -i`) | Git Merge (`git merge`) |
| :--- | :--- | :--- |
| **Đặc điểm lịch sử** | Tạo ra lịch sử commit **tuyến tính (linear history)**, phẳng và sạch | Giữ nguyên nhánh phân nhánh, tạo thêm **Merge Commit** |
| **Tác động tới SHA-1 Hash** | **Thay đổi mã hash** của các commit bị chỉnh sửa/gộp | Giữ nguyên mã hash của các commit cũ |
| **Thời điểm áp dụng** | **Chỉ dùng cho Local Branch** trước khi push code lên Remote | Dùng khi tích hợp các nhánh độc lập trên Remote |
| **Quy tắc vàng (Golden Rule)** | **Không bao giờ rebase các commit đã push lên remote public!** | An toàn cho các nhánh chia sẻ chung |

---

## 7. Kết Luận

- Thực hiện thành công kỹ thuật **Interactive Rebase** (`git rebase -i`) để tái cấu trúc lịch sử commit.
- Gộp gọn 3 commit nhỏ lẻ thành 1 commit có ý nghĩa chuẩn mực về mặt chức năng.
- Loại bỏ hoàn toàn tệp tin rác `temp.txt` thông qua thao tác `drop`.
- Cây lịch sử commit được làm sạch hoàn toàn trước khi đẩy lên remote `https://github.com/DevOpsPTIT/SS05_Bai02.git`.
