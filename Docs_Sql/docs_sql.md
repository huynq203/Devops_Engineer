# MySQL - Quản lý User và Phân quyền

## I. Quản lý User

### 1. Tạo User

Cú pháp:

```sql
CREATE USER 'username'@'ip_server' IDENTIFIED BY 'password';
```

Ví dụ:

```sql
CREATE USER 'shoeshop'@'%' IDENTIFIED BY '123456';
```

> `'%`' cho phép đăng nhập từ mọi địa chỉ IP. Có thể thay bằng IP cụ thể để tăng tính bảo mật.

---

### 2. Xóa User

Cú pháp:

```sql
DROP USER 'username'@'ip_server';
```

Ví dụ:

```sql
DROP USER 'shoeshop'@'%';
```

---

## II. Phân quyền User

### 1. Cấp toàn bộ quyền trên Database

Cú pháp:

```sql
GRANT ALL PRIVILEGES ON db_name.* TO 'username'@'ip_server';
```

Ví dụ:

```sql
GRANT ALL PRIVILEGES ON shoeshop.* TO 'shoeshop'@'%';
```

---

### 2. Cấp toàn bộ quyền trên một bảng

Cú pháp:

```sql
GRANT ALL PRIVILEGES ON db_name.table_name TO 'username'@'ip_server';
```

Ví dụ:

```sql
GRANT ALL PRIVILEGES ON shoeshop.products TO 'shoeshop'@'%';
```

---

### 3. Cấp một quyền cụ thể

Cú pháp:

```sql
GRANT <PRIVILEGE> ON db_name.* TO 'username'@'ip_server';
```

Ví dụ:

Chỉ cho phép đọc dữ liệu:

```sql
GRANT SELECT ON shoeshop.* TO 'shoeshop'@'%';
```

Chỉ cho phép thêm dữ liệu:

```sql
GRANT INSERT ON shoeshop.* TO 'shoeshop'@'%';
```

Chỉ cho phép cập nhật dữ liệu:

```sql
GRANT UPDATE ON shoeshop.* TO 'shoeshop'@'%';
```

Chỉ cho phép xóa dữ liệu:

```sql
GRANT DELETE ON shoeshop.* TO 'shoeshop'@'%';
```

Các quyền thường dùng:

| Quyền | Ý nghĩa |
|--------|----------|
| `SELECT` | Đọc dữ liệu |
| `INSERT` | Thêm dữ liệu |
| `UPDATE` | Cập nhật dữ liệu |
| `DELETE` | Xóa dữ liệu |
| `CREATE` | Tạo Database/Table |
| `DROP` | Xóa Database/Table |
| `ALTER` | Thay đổi cấu trúc bảng |
| `INDEX` | Tạo/Xóa Index |
| `EXECUTE` | Thực thi Stored Procedure |
| `ALL PRIVILEGES` | Cấp toàn bộ quyền |

---

### 4. Thu hồi toàn bộ quyền

Cú pháp:

```sql
REVOKE ALL PRIVILEGES ON db_name.* FROM 'username'@'ip_server';
```

Ví dụ:

```sql
REVOKE ALL PRIVILEGES ON shoeshop.* FROM 'shoeshop'@'%';
```

---

### 5. Thu hồi một quyền cụ thể

Cú pháp:

```sql
REVOKE SELECT ON db_name.* FROM 'username'@'ip_server';
```

Ví dụ:

```sql
REVOKE UPDATE ON shoeshop.* FROM 'shoeshop'@'%';
```

---

## III. Áp dụng thay đổi quyền

Sau khi thay đổi quyền, thực hiện:

```sql
FLUSH PRIVILEGES;
```

---

## IV. Kiểm tra quyền của User

Hiển thị quyền của User:

```sql
SHOW GRANTS FOR 'shoeshop'@'%';
```

---

## V. Ví dụ hoàn chỉnh

```sql
CREATE USER 'shoeshop'@'%' IDENTIFIED BY '123456';

GRANT ALL PRIVILEGES ON shoeshop.* TO 'shoeshop'@'%';

FLUSH PRIVILEGES;

SHOW GRANTS FOR 'shoeshop'@'%';
```

---

## VI. Luồng quản lý User

```text
Tạo User
     │
     ▼
Cấp quyền
     │
     ▼
FLUSH PRIVILEGES
     │
     ▼
Kiểm tra quyền
     │
     ▼
Sử dụng Database
     │
     ▼
Thu hồi quyền (nếu cần)
     │
     ▼
Xóa User
```