# Ansible Project Structure

Dưới đây là một cấu trúc Ansible mẫu phù hợp cho dự án thực tế, rõ ràng, dễ mở rộng và dễ bảo trì. Mục tiêu là tách biệt rõ ràng giữa inventory, biến môi trường, playbook, role và tài nguyên được sử dụng của từng task.

## 1. Cấu trúc dự án khuyến nghị

```text
ansible-project/
├── ansible.cfg                  # Cấu hình chung cho Ansible
├── README.md                   # Hướng dẫn dự án
├── .gitignore                  # Bỏ qua file nhạy cảm / tạm thời
├── inventory/                  # Thư mục inventory theo môi trường
│   ├── production/
│   │   ├── hosts.ini           # Inventory cho production
│   │   └── group_vars/
│   │       └── all.yml
│   ├── staging/
│   │   ├── hosts.ini           # Inventory cho staging
│   │   └── group_vars/
│   │       └── all.yml
│   └── dev/
│       └── hosts.ini           # Inventory cho dev
├── group_vars/
│   ├── all.yml                  # Biến dùng chung cho tất cả nhóm
│   ├── webservers.yml           # Biến cho nhóm webservers
│   ├── dbservers.yml            # Biến cho nhóm dbservers
│   └── app.yml                 # Biến cho nhóm app
├── host_vars/
│   ├── web01.yml               # Biến riêng cho host web01
│   ├── db01.yml                # Biến riêng cho host db01
│   └── app01.yml               # Biến riêng cho host app01
├── files/                      # File tĩnh dùng trong module copy/template
│   ├── scripts/
│   │   └── deploy.sh
│   └── configs/
│       └── nginx.conf
├── templates/                  # Template Jinja2
│   ├── nginx.conf.j2
│   ├── app.env.j2
│   └── db.conf.j2
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── vars/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── ntp.conf.j2
│   │   ├── files/
│   │   │   └── motd.txt
│   │   └── meta/
│   │       └── main.yml
│   ├── webtier/
│   │   └── ...
│   ├── database/
│   │   └── ...
│   └── app/
│       └── ...
├── playbooks/
│   ├── site.yml                # Playbook chính
│   ├── webservers.yml          # Playbook riêng cho web tier
│   ├── dbservers.yml           # Playbook riêng cho db tier
│   └── deploy.yml              # Playbook triển khai ứng dụng
├── library/                    # Custom module (nếu có)
├── module_utils/               # Utilities cho custom module
├── filter_plugins/             # Filter plugin tùy chỉnh
├── lookup_plugins/             # Lookup plugin tùy chỉnh
├── vault/                      # File vault hoặc dữ liệu nhạy cảm
│   └── secrets.yml
├── docs/                       # Tài liệu bổ sung
│   └── deployment-guide.md
└── tests/                      # Kiểm thử / validation (nếu có)
    └── syntax-check.yml
```

## 2. Giải thích các thành phần chính

### 2.1. ansible.cfg
File này chứa cấu hình chung cho Ansible như:
- đường dẫn inventory mặc định
- số lượng task worker
- cài đặt callback / logging
- timeout và cách xử lý SSH

Mục đích: đảm bảo mọi máy chạy với cùng một cấu hình, tránh phải lặp lại các option trong từng playbook.

### 2.2. inventory/
Inventory là nơi khai báo danh sách host và nhóm máy.
- `production`, `staging`, `dev` tương ứng với từng môi trường
- Mỗi môi trường có thể có `hosts.ini` riêng
- Nên tách theo môi trường để không lẫn nhầm cấu hình giữa dev/staging/production

Ví dụ:

```ini
[webservers]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[dbservers]
db01 ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 2.3. group_vars/ và host_vars/
Đây là nơi lưu biến theo nhóm hoặc theo từng host:
- `group_vars/`: biến áp dụng cho một nhóm host
- `host_vars/`: biến áp dụng cho từng máy cụ thể

Tốt cho việc:
- quản lý biến môi trường theo từng nhóm
- tránh hardcode trong playbook
- dễ thay đổi cấu hình cho từng môi trường mà không sửa code task

Ví dụ:
- `group_vars/webservers.yml` chứa cổng HTTP, phiên bản NGINX, user deploy
- `host_vars/web01.yml` chứa địa chỉ IP, thông tin môi trường riêng hoặc secret nếu cần

### 2.4. roles/
Role là đơn vị cấu trúc chính của Ansible. Mỗi role tập trung vào một chức năng riêng như:
- `common`: cài đặt tác vụ chung cho mọi server
- `webtier`: cấu hình webserver
- `database`: cấu hình database
- `app`: deploy ứng dụng

Mỗi role nên có cấu trúc chuẩn:

```text
roles/
  myrole/
    tasks/
      main.yml
    handlers/
      main.yml
    defaults/
      main.yml
    vars/
      main.yml
    files/
      ...
    templates/
      ...
    meta/
      main.yml
```

Ưu điểm của role:
- tái sử dụng dễ dàng
- code rõ ràng, modular
- thay đổi một role không ảnh hưởng lớn đến toàn bộ project

### 2.5. playbooks/
Playbook là file mô tả mục tiêu triển khai. Nên giữ playbook ngắn, tập trung vào "cái gì cần chạy" thay vì "làm từng thao tác chi tiết".

Ví dụ:

```yaml
- hosts: webservers
  become: true
  roles:
    - common
    - webtier
```

Nên:
- có playbook chung `site.yml` để chạy toàn bộ hệ thống
- có playbook riêng cho từng tầng như `webservers.yml`, `dbservers.yml`
- không viết quá nhiều logic trực tiếp trong playbook; hãy đưa logic vào role

### 2.6. files/ và templates/
- `files/`: lưu file tĩnh cần sao chép trực tiếp đến host
- `templates/`: lưu template Jinja2, cho phép ghép biến vào nội dung file

Ví dụ:
- `templates/nginx.conf.j2` dùng để sinh ra file cấu hình NGINX từ biến khác nhau
- `files/scripts/deploy.sh` là script deploy có thể được copy sang máy

## 3. Cách tổ chức thông tin sao cho rõ ràng và dễ bảo trì

### 3.1. Tách biệt theo chức năng, không gom lẫn
Không nên đặt quá nhiều file cùng lúc vào một thư mục. Ví dụ:
- biến môi trường phải nằm trong `group_vars` hoặc `host_vars`
- task logic nên nằm trong `roles/*/tasks`
- cấu hình template nên nằm trong `templates/`
- file nhạy cảm nên lưu riêng và mã hóa bằng Vault

### 3.2. Tách môi trường theo inventory
Mỗi môi trường (`dev`, `staging`, `production`) nên có inventory riêng. Điều này giúp:
- tránh nhầm lẫn giữa môi trường
- dễ chạy playbook đúng môi trường
- thuận tiện cho triển khai theo từng giai đoạn

### 3.3. Sử dụng role để giảm trùng lặp
Nếu nhiều playbook cần cùng một tác vụ, hãy đưa vào role. Ví dụ:
- cài đặt package chung
- cấu hình SSH
- quản lý user/systemd
- deploy ứng dụng

Role giúp tái sử dụng, dễ test và dễ cập nhật.

### 3.4. Biến nên được đặt ở đúng mức ưu tiên
Ansible có quy tắc ưu tiên biến, nên nên tổ chức theo nguyên tắc:
- `defaults/`: giá trị mặc định, dùng cho role
- `vars/`: biến cục bộ của role
- `group_vars/`: biến theo nhóm
- `host_vars/`: biến theo từng host
- biến truyền từ command line hoặc vault: ưu tiên cao nhất

Nguyên tắc này giúp tránh hardcode và dễ kiểm soát override.

### 3.5. Giữ playbook ngắn gọn
Playbook nên đọc như một "kịch bản triển khai" chứ không phải là "địa chỉ tất cả việc làm". Một playbook tốt thường:
- chỉ định `hosts`
- chỉ định `become`
- gọi các role cần thiết
- không chứa quá nhiều task phức tạp

### 3.6. Bảo mật và quản lý secret
Các thông tin nhạy cảm như mật khẩu, token, khóa SSH nên:
- không để trực tiếp trong repo
- dùng `ansible-vault` để mã hóa
- đặt secret trong `group_vars` hoặc file `vault/` kèm khóa bảo mật

## 4. Nguyên tắc thiết kế để dễ bảo trì

1. Mỗi role chỉ làm một nhiệm vụ rõ ràng.
2. Đặt tên file và thư mục có ý nghĩa, dễ hiểu.
3. Giữ task ngắn, tách logic thành file nhỏ nếu quá dài.
4. Sử dụng biến và defaults để giảm hardcode.
5. Dùng inventory theo môi trường để tránh xung đột cấu hình.
6. Viết comment và README mô tả mục đích của role/playbook.
7. Kiểm tra syntax và chạy triển khai trên môi trường test trước khi production.
8. Duy trì cấu trúc thư mục ổn định để team dễ làm việc cùng nhau.

## 5. Ví dụ cấu trúc role đơn giản

```text
roles/
  webtier/
    tasks/
      main.yml
    handlers/
      main.yml
    defaults/
      main.yml
    templates/
      nginx.conf.j2
    files/
      app.conf
```

`main.yml` trong role webtier thường chứa logic như:
- cài đặt package
- sao chép template
- khởi động service
- gọi handler khi file cấu hình thay đổi

## 6. Kết luận

Một dự án Ansible tốt không chỉ cần chạy thành công, mà còn phải dễ đọc, dễ mở rộng và dễ bảo trì khi team lớn lên. Cách tổ chức hợp lý nhất là:
- tách môi trường bằng inventory
- gom biến theo mục đích bằng `group_vars` / `host_vars`
- chia logic thành role
- giữ playbook ngắn gọn
- lưu template và file tĩnh riêng biệt
- kiểm soát secret bằng Vault

Nếu làm đúng theo các nguyên tắc trên, dự án sẽ rất dễ quản lý trong thời gian dài và dễ phát triển thêm tính năng mới mà không làm rối cấu trúc hiện tại.
