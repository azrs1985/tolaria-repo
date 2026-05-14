# [PG] Set default schema

Owner: Nam Tran
Last edited time: October 22, 2025 2:07 PM

To check the default schema (i.e., the current search_path) in PostgreSQL, you can run this SQL command: *(Để kiểm tra schema mặc định (tức search_path hiện tại) trong PostgreSQL, hãy chạy lệnh SQL sau:)*

```sql
SHOW search_path;
```

This will return the list of schemas that PostgreSQL will search by default when resolving unqualified table names. The first schema in the list is considered the default. (*Lệnh này trả về danh sách các schema mà PostgreSQL sẽ tìm theo mặc định khi xử lý các tên bảng không đủ định danh. Schema đứng đầu danh sách được coi là mặc định.)*

---

**🛠 Option 1: Set default schema for a specific user** *(Thiết lập schema mặc định cho một người dùng cụ thể)*

This sets the default schema every time the user connects. (*Tuỳ chọn này đặt schema mặc định mỗi khi người dùng kết nối.)*

```sql
ALTER ROLE your_username SET search_path TO your_schema;
```

Replace your_username with the PostgreSQL role name, and your_schema with the schema you want to use by default. (*Thay your_username bằng tên role trong PostgreSQL, và your_schema bằng schema bạn muốn dùng mặc định.)*

---

**🛠 Option 2: Set default schema for the current session** *(Thiết lập schema mặc định cho phiên làm việc hiện tại)*

This only affects the current connection/session. (*Chỉ ảnh hưởng đến kết nối/phiên làm việc hiện tại.)*

```sql
SET search_path TO your_schema;
```

---

**🛠 Option 3: Set default schema at the database level** *(Thiết lập schema mặc định ở cấp độ cơ sở dữ liệu)*

This applies to all users unless overridden. (*Áp dụng cho mọi người dùng trừ khi bị ghi đè ở nơi khác.)*

```sql
ALTER DATABASE your_database SET search_path TO your_schema;
```

---

**🛠 You can specify multiple schemas in search_path, like:** *(Bạn có thể chỉ định nhiều schema trong search_path, ví dụ:)*

```sql
SET search_path TO schema1, schema2, public;
```

The public schema is included by default unless you override it. *(Schema public được bao gồm theo mặc định trừ khi bạn ghi đè.)*