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

# Note
- Idempotent: Thao tác thực hiện 1 / nhiều lần liên tiếp thì kết quả của hệ thống vẫn giữ nguyên như khi thực hiện 1 lần.
