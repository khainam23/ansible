# Ansible 

Điều khiển nhiều máy cùng lúc thực hiện các task chỉ định bằng 1 máy trung tâm

# Architecture

## Inventory
Chứa thông tin về các server làm việc

## Playbook
File định nghĩa các thông tin task thực hiện trên các server 

### Task
Các task thực thi trên server

### Module
Là các phần task sử dụng để thực hiện.

### Facts
Thông tin của server

# Note
- Idempotent: Thao tác thực hiện 1 / nhiều lần liên tiếp thì kết quả của hệ thống vẫn giữ nguyên như khi thực hiện 1 lần.
- Mỗi task sẽ chạy riêng một lần tạo connection, muốn không như vậy thì sử dụng SSH multiplexing, bằng cách đặt 
`[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s`
- Thông thường Ansible thường để `gather_facts: true` -> Mặc định lấy thông tin, chạy lớn sẽ có vấn đề

# Case
- Nếu cần lưu trữ một lượng lớn facts của các server thì nên cấu hình cho ansible dủng redis làm BE vì cái facts này sẽ không thay đổi nhiều trong một thời gian ngắn