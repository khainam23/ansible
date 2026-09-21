- Sử dụng ansible-vault để lưu trữ mật khẩu 

- Khi tạo vault thì đặt nó trong group_vars, phần sub phía sau chính là group đã thiết lập trong inventory, nghĩa là
nếu ghi `[name]` thì sẽ để nó là `name`.
Bên trong là file được tạo bằng ansible-vault

- Mỗi lần chạy với ansible vault thì cần nhập pass, nếu không muốn dùng thì tạo thêm file .vault_pass để sử dụng nhanh, 
nhưng cần lưu ý là nó cần chmod 600 vì nếu không linux sẽ hiểu nó là file thực thi và chạy.