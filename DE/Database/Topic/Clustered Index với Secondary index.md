### 1. Physical Position (Vị trí vật lý) không đổi

Trong InnoDB, bảng dữ liệu chính là một cái kho được sắp xếp theo Primary Key (gọi là Clustered Index).

- Khi bạn tạo thêm một **Secondary Index** (ví dụ: Index trên cột `email`), MySQL không hề copy hay di chuyển các dòng dữ liệu thực tế (row data) đi chỗ khác.
    
- Dữ liệu thực tế vẫn nằm yên tại các trang (pages) của Clustered Index.
    

### 2. Sự thay đổi nằm ở đâu?

Việc đánh Secondary Index thực chất là tạo ra một **cấu trúc dữ liệu tách biệt hoàn toàn** so với bảng chính.

- **Secondary Index** là một cây B+Tree mới.
    
- Mỗi "dòng" (entry) trong Secondary Index chỉ chứa: **Giá trị của cột được index** + **Giá trị của Primary Key**.
    

### 3. "Vị trí logic" và cơ chế Row Pointer

Bạn nói "thay đổi vị trí logic" có thể hiểu là: đối với cái Index mới đó, các hàng được sắp xếp theo một thứ tự khác.

**Ví dụ:** Bạn có bảng `USER` với Primary Key là `ID` (1, 2, 3) và Secondary Index trên `Name`.

- **Vật lý (Clustered Index):** Dòng ID=1 nằm trước ID=2.
    
- **Logic (Secondary Index):** Nếu tên của ID=2 là "An" và ID=1 là "Bình", thì trong cây Index của `Name`, bản ghi "An" sẽ đứng trước "Bình".
    

### 4. Quá trình tìm kiếm (Lookup)

Đây là lúc sự tách biệt giữa vật lý và logic thể hiện rõ nhất:

1. MySQL tìm trong **Secondary Index** để lấy ra giá trị **Primary Key** (đây là tìm kiếm logic theo thứ tự của Index).
    
2. Sau khi có Primary Key, MySQL thực hiện một bước gọi là **Bookmark Lookup (hay Key Lookup)**: Nó cầm cái ID đó quay lại **Clustered Index** để tìm đúng vị trí vật lý của dòng dữ liệu nhằm lấy ra các cột còn lại.
    

---

### Tại sao kiến thức này quan trọng với bạn?

1. **Ghi dữ liệu (Write Overhead):** Vì Secondary Index là một cấu trúc vật lý riêng biệt, nên mỗi khi bạn `INSERT` hoặc `UPDATE` cột đó, MySQL phải ghi vào ít nhất 2 nơi: bảng chính và các cây index phụ. Điều này giải thích tại sao càng nhiều index thì ghi càng chậm.
    
2. **Change Buffer:** Như chúng ta đã thảo luận trước đó, vì Secondary Index nằm ở vị trí vật lý riêng (thường là ngẫu nhiên trên đĩa), nên MySQL mới cần đến **Change Buffer** để trì hoãn việc ghi, tránh việc phải nhảy đi nhảy lại trên ổ đĩa để cập nhật các "vị trí vật lý" phân mảnh đó.
    

**Tóm lại:**

- **Dữ liệu gốc (Physical Row):** Không đổi.
    
- **Index phụ:** Là một "bản đồ" mới, lưu thứ tự sắp xếp khác và trỏ ngược về dữ liệu gốc thông qua Primary Key.