# Bài 2: Khởi tạo User thường và thiết lập đặc quyền quản trị (Sudoers Configuration)

## 1. Mục tiêu

Bài thực hành nhằm tạo một tài khoản người dùng thường trên Ubuntu Server để hạn chế việc thao tác trực tiếp bằng tài khoản `root`.

Tài khoản được sử dụng trong bài:

```text
devops
```

Các mục tiêu chính:

- Tạo user `devops`.
- Thêm user `devops` vào nhóm `sudo`.
- Sao chép SSH Key từ user `root` sang user `devops`.
- Thiết lập đúng quyền cho thư mục `.ssh`.
- Đăng nhập SSH bằng user `devops`.
- Kiểm tra quyền quản trị bằng lệnh `sudo whoami`.

---

## 2. Điều kiện ban đầu

Máy chủ Ubuntu đã được tạo và có thể SSH bằng tài khoản `root`.

Ví dụ:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" root@<IP_ADDRESS>
```

Sau khi đăng nhập thành công, terminal có dạng:

```text
root@server:~#
```

---

## 3. Tạo user devops

Đăng nhập vào máy chủ bằng tài khoản `root`.

Chạy:

```bash
adduser devops
```

Hệ thống sẽ yêu cầu tạo mật khẩu cho user `devops`.

Ví dụ:

```text
New password:
Retype new password:
```

Sau đó hệ thống có thể hỏi các thông tin như:

```text
Full Name
Room Number
Work Phone
Home Phone
Other
```

Có thể nhấn `Enter` để bỏ qua.

Cuối cùng nhập:

```text
Y
```

để xác nhận.

---

## 4. Kiểm tra user đã được tạo

Chạy:

```bash
id devops
```

Kết quả mong đợi có dạng:

```text
uid=1001(devops) gid=1001(devops) groups=1001(devops)
```

Có thể kiểm tra thư mục home:

```bash
ls -ld /home/devops
```

---

## 5. Thêm user devops vào nhóm sudo

Chạy:

```bash
usermod -aG sudo devops
```

Kiểm tra:

```bash
groups devops
```

Kết quả mong đợi có chứa:

```text
devops sudo
```

Hoặc:

```bash
id devops
```

và kiểm tra trong danh sách group có `sudo`.

---

## 6. Sao chép SSH Key từ root sang devops

Khi đang đăng nhập bằng `root`, thư mục SSH hiện tại thường nằm tại:

```text
/root/.ssh
```

Sử dụng lệnh:

```bash
rsync --archive --chown=devops:devops /root/.ssh /home/devops/
```

Nếu muốn dùng đúng dạng lệnh gợi ý của đề khi đang là root:

```bash
rsync --archive --chown=devops:devops ~/.ssh /home/devops/
```

Sau khi chạy, user `devops` sẽ có:

```text
/home/devops/.ssh/authorized_keys
```

---

## 7. Thiết lập quyền cho thư mục SSH

Để SSH hoạt động an toàn, cần đặt đúng owner và permission.

### 7.1. Đặt owner

```bash
chown -R devops:devops /home/devops/.ssh
```

### 7.2. Phân quyền thư mục `.ssh`

```bash
chmod 700 /home/devops/.ssh
```

### 7.3. Phân quyền file `authorized_keys`

```bash
chmod 600 /home/devops/.ssh/authorized_keys
```

---

## 8. Kiểm tra quyền SSH

Chạy:

```bash
ls -ld /home/devops/.ssh
```

Kết quả mong đợi tương tự:

```text
drwx------ 2 devops devops ...
```

Kiểm tra file:

```bash
ls -l /home/devops/.ssh/authorized_keys
```

Kết quả mong đợi tương tự:

```text
-rw------- 1 devops devops ...
```

---

## 9. Kiểm tra user thuộc nhóm sudo

Chạy:

```bash
groups devops
```

Kết quả mong đợi:

```text
devops : devops sudo
```

Có thể dùng thêm:

```bash
getent group sudo
```

và kiểm tra tên `devops` có xuất hiện trong danh sách.

---

## 10. Thoát khỏi tài khoản root

Chạy:

```bash
exit
```

Sau đó terminal trở về máy Windows cá nhân.

---

## 11. SSH bằng user devops

Trên Windows PowerShell, chạy:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devops@<IP_ADDRESS>
```

Ví dụ:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devops@xxx.xxx.xxx.xxx
```

Nếu cấu hình đúng, sẽ đăng nhập thành công và terminal có dạng:

```text
devops@server:~$
```

Kiểm tra user hiện tại:

```bash
whoami
```

Kết quả:

```text
devops
```

---

## 12. Kiểm tra quyền sudo

Sau khi đang đăng nhập bằng user `devops`, chạy:

```bash
sudo whoami
```

Hệ thống có thể yêu cầu:

```text
[sudo] password for devops:
```

Nhập mật khẩu đã tạo khi chạy lệnh:

```bash
adduser devops
```

Kết quả mong đợi:

```text
root
```

Điều này chứng minh user `devops` có quyền thực thi lệnh quản trị thông qua `sudo`.

---

## 13. Các lệnh chính đã sử dụng

```bash
adduser devops

usermod -aG sudo devops

groups devops

rsync --archive --chown=devops:devops /root/.ssh /home/devops/

chown -R devops:devops /home/devops/.ssh

chmod 700 /home/devops/.ssh

chmod 600 /home/devops/.ssh/authorized_keys

ls -ld /home/devops/.ssh

ls -l /home/devops/.ssh/authorized_keys
```

Trên máy cá nhân:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devops@<IP_ADDRESS>
```

Sau khi đăng nhập:

```bash
whoami

sudo whoami
```

---

## 14. Kết quả mong đợi

Sau khi hoàn thành bài:

- User `devops` được tạo thành công.
- User `devops` thuộc nhóm `sudo`.
- SSH Key từ `root` được sao chép sang user `devops`.
- Thư mục `.ssh` có quyền `700`.
- File `authorized_keys` có quyền `600`.
- Owner của `.ssh` và `authorized_keys` là `devops`.
- Có thể SSH trực tiếp bằng tài khoản `devops`.
- Lệnh `sudo whoami` trả về `root`.

---

## 15. Minh chứng

### 15.1. Kiểm tra group sudo

Chụp hoặc chèn log thực tế của lệnh:

```bash
groups devops
```

### 15.2. Kiểm tra quyền SSH

Chụp hoặc chèn log:

```bash
ls -ld /home/devops/.ssh
ls -l /home/devops/.ssh/authorized_keys
```

### 15.3. SSH bằng tài khoản devops

Chụp PowerShell sau khi chạy:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" devops@<IP_ADDRESS>
```

và sau đó:

```bash
whoami
```

Kết quả thực tế cần hiển thị:

```text
devops
```

### 15.4. Kiểm tra sudo

Chụp log thực tế:

```bash
sudo whoami
```

Kết quả:

```text
root
```

> Không sử dụng log giả. Nếu giảng viên yêu cầu minh chứng, cần thay phần này bằng ảnh hoặc log thật từ máy chủ.

---

## 16. Cấu trúc thư mục nộp bài

```text
homework/
└── session_02/
    └── ex2/
        ├── README.md
        └── images/
            ├── ssh-devops.png
            └── sudo-whoami.png
```

Có thể chèn ảnh vào README:

```markdown
![SSH DevOps](./images/ssh-devops.png)

![Sudo Whoami](./images/sudo-whoami.png)
```

---

## 17. Kết luận

Bài thực hành đã mô tả quy trình tạo tài khoản người dùng thường `devops`, cấp quyền quản trị thông qua nhóm `sudo` và cấu hình SSH Key để đăng nhập an toàn.

Việc sử dụng user thường thay cho `root` giúp giảm rủi ro khi vận hành máy chủ và phù hợp với nguyên tắc Least Privilege.

User `devops` chỉ sử dụng quyền quản trị khi cần thông qua lệnh:

```bash
sudo <command>
```

Qua đó giúp việc quản trị Ubuntu Server an toàn và dễ kiểm soát hơn.
