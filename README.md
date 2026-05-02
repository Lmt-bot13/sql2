# BÀI KIỂM TRA SỐ 2 – HỆ QUẢN TRỊ CSDL SQL SERVER

**Họ và tên:** Lưu Minh Trí

**Mã sinh viên:** K235480106104


**Lớp:** K59KMT.K01  

**Đề tài:** Quản lý lịch thi  


# PHẦN MỞ ĐẦU

## 1. Giới thiệu chung

Trong phạm vi bài kiểm tra này, em lựa chọn xây dựng cơ sở dữ liệu cho bài toán **quản lý lịch thi**. Đây là một bài toán rất gần với môi trường đào tạo thực tế vì mỗi học kỳ nhà trường phải tổ chức nhiều môn thi, nhiều phòng thi, nhiều ca thi và nhiều sinh viên đăng ký dự thi.

Nếu quản lý thủ công bằng bảng tính hoặc giấy tờ thì rất dễ xảy ra các vấn đề như:

- trùng lịch thi giữa các môn,
- xếp quá số lượng sinh viên vào một phòng,
- nhập sai giờ thi hoặc ngày thi,
- khó thống kê số sinh viên dự thi,
- khó phát hiện lịch thi bất hợp lý.

Vì vậy, việc xây dựng cơ sở dữ liệu trên **SQL Server** sẽ giúp quản lý dữ liệu chặt chẽ hơn, kiểm soát ràng buộc tốt hơn và hỗ trợ truy vấn – báo cáo nhanh hơn.

## 2. Định hướng thực hiện

Bài làm được triển khai theo 5 phần đúng yêu cầu đề bài:

1. Thiết kế và khởi tạo cấu trúc dữ liệu.  
2. Xây dựng Function.  
3. Xây dựng Stored Procedure.  
4. Trigger và xử lý logic nghiệp vụ.  
5. Cursor và duyệt dữ liệu.  

Ngoài việc đáp ứng đủ nội dung kỹ thuật, em đặc biệt chú ý:

- tên bảng, tên cột đặt theo kiểu **BướuLạcĐà**,
- dùng dấu **[ ]** để bọc tên bảng và tên cột trong script,
- các đoạn mã **SQL Server đều có chú thích rõ ràng**,
- mỗi phần đều có mô tả ngắn về **bài toán đặt ra** và **phân tích cách giải**.

## 3. Thông tin database sử dụng

Tên cơ sở dữ liệu có gắn mã sinh viên đúng yêu cầu:

```sql
QuanLyLichThi_K235480106104
```

---

# PHẦN 1: THIẾT KẾ VÀ KHỞI TẠO CẤU TRÚC DỮ LIỆU

## 1.1 Phân tích bài toán

Bài toán quản lý lịch thi cần xử lý các thông tin chính sau:

- **Môn học**: tên môn, số tín chỉ.
- **Phòng thi**: mã phòng, sức chứa, khu nhà.
- **Ca thi / lịch thi**: ngày thi, giờ bắt đầu, giờ kết thúc, phòng thi, môn thi.
- **Sinh viên dự thi**: sinh viên nào đăng ký thi ở lịch nào.

Từ đó, em xây dựng 4 bảng chính:

- `[MonHoc]`
- `[PhongThi]`
- `[LichThi]`
- `[DangKyThi]`

## 1.2 Mối quan hệ dữ liệu

- Một **môn học** có thể có nhiều **lịch thi**.
- Một **phòng thi** có thể được sử dụng cho nhiều **lịch thi** ở các thời điểm khác nhau.
- Một **lịch thi** có thể có nhiều **sinh viên đăng ký thi**.

Sơ đồ quan hệ đơn giản:

```text
[MonHoc] 1 ---- N [LichThi] 1 ---- N [DangKyThi]
[PhongThi] 1 ---- N [LichThi]
```

## 1.3 Mô tả bảng dữ liệu

### Bảng `[MonHoc]`

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaMonHoc] | INT | PK, IDENTITY(1,1) | Mã môn học |
| [TenMonHoc] | NVARCHAR(150) | NOT NULL | Tên môn học |
| [SoTinChi] | INT | NOT NULL, CHECK > 0 | Số tín chỉ |
| [TrangThai] | NVARCHAR(30) | DEFAULT N'DangMo' | Trạng thái môn học |

### Bảng `[PhongThi]`

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaPhong] | INT | PK, IDENTITY(1,1) | Mã phòng thi |
| [TenPhong] | NVARCHAR(50) | NOT NULL, UNIQUE | Tên phòng |
| [SucChua] | INT | NOT NULL, CHECK > 0 | Sức chứa |
| [KhuNha] | NVARCHAR(50) | NOT NULL | Khu nhà |
| [TrangThai] | NVARCHAR(30) | DEFAULT N'SanSang' | Trạng thái phòng |

### Bảng `[LichThi]`

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaLichThi] | INT | PK, IDENTITY(1,1) | Mã lịch thi |
| [MaMonHoc] | INT | FK | Môn thi |
| [MaPhong] | INT | FK | Phòng thi |
| [NgayThi] | DATE | NOT NULL | Ngày thi |
| [GioBatDau] | TIME | NOT NULL | Giờ bắt đầu |
| [GioKetThuc] | TIME | NOT NULL | Giờ kết thúc |
| [SoLuongToiDa] | INT | NOT NULL, CHECK > 0 | Số lượng SV tối đa |
| [GhiChu] | NVARCHAR(200) | NULL | Ghi chú |

### Bảng `[DangKyThi]`

| Cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaDangKy] | INT | PK, IDENTITY(1,1) | Mã đăng ký thi |
| [MaLichThi] | INT | FK | Lịch thi |
| [MaSinhVien] | NVARCHAR(20) | NOT NULL | Mã sinh viên |
| [HoTenSinhVien] | NVARCHAR(150) | NOT NULL | Họ tên sinh viên |
| [NgayDangKy] | DATETIME | DEFAULT GETDATE() | Thời điểm đăng ký |
| [TrangThaiDuThi] | NVARCHAR(30) | DEFAULT N'DaDangKy' | Trạng thái dự thi |

## 1.4 Tạo database

### Bài toán đặt ra
Tạo một cơ sở dữ liệu mới đúng tên quy định để lưu toàn bộ dữ liệu cho hệ thống quản lý lịch thi.

### Code SQL

```sql
-- Tạo database mới có gắn mã sinh viên đúng yêu cầu đề bài
CREATE DATABASE [QuanLyLichThi_K235480106104];
GO

-- Chuyển ngữ cảnh làm việc sang database vừa tạo
USE [QuanLyLichThi_K235480106104];
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a96db0ce-4de3-402f-ae4d-7d65d4809dfc" />


Ảnh này cho thấy em đã tạo thành công database đúng tên yêu cầu và đã chuyển sang database đó để tiếp tục làm bài.

## 1.5 Tạo bảng

### 1.5.1 Tạo bảng `[MonHoc]`

### Bài toán đặt ra
Cần một bảng lưu danh mục môn học để về sau khi tạo lịch thi có thể biết lịch đó thuộc môn nào.

### Code SQL

```sql
-- Tạo bảng MonHoc để lưu thông tin môn thi
CREATE TABLE [MonHoc]
(
    -- Mã môn học tự tăng, đóng vai trò khóa chính
    [MaMonHoc] INT IDENTITY(1,1) NOT NULL,

    -- Tên môn học, bắt buộc phải có
    [TenMonHoc] NVARCHAR(150) NOT NULL,

    -- Số tín chỉ của môn học, bắt buộc > 0
    [SoTinChi] INT NOT NULL,

    -- Trạng thái môn học: đang mở hoặc ngừng mở
    [TrangThai] NVARCHAR(30) NOT NULL DEFAULT N'DangMo',

    -- Khóa chính của bảng MonHoc
    CONSTRAINT [PK_MonHoc] PRIMARY KEY ([MaMonHoc]),

    -- Ràng buộc kiểm tra số tín chỉ phải lớn hơn 0
    CONSTRAINT [CK_MonHoc_SoTinChi] CHECK ([SoTinChi] > 0),

    -- Ràng buộc kiểm tra trạng thái chỉ nhận các giá trị hợp lệ
    CONSTRAINT [CK_MonHoc_TrangThai] CHECK ([TrangThai] IN (N'DangMo', N'NgungMo'))
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/76b1b1a2-f57a-4f41-a69a-6ded6168fa68" />

Thấy bảng MonHoc đã được tạo thành công, trong đó MaMonHoc là PK và SoTinChi có ràng buộc CHECK.

### 1.5.2 Tạo bảng `[PhongThi]`

### Bài toán đặt ra
Cần lưu danh sách phòng thi để biết mỗi lịch thi tổ chức ở đâu và giới hạn số lượng sinh viên theo sức chứa phòng.

### Code SQL

```sql
-- Tạo bảng PhongThi để lưu thông tin phòng thi
CREATE TABLE [PhongThi]
(
    -- Mã phòng thi tự tăng
    [MaPhong] INT IDENTITY(1,1) NOT NULL,

    -- Tên phòng, ví dụ A101, B203, không được trùng
    [TenPhong] NVARCHAR(50) NOT NULL,

    -- Sức chứa tối đa của phòng thi
    [SucChua] INT NOT NULL,

    -- Khu nhà của phòng thi
    [KhuNha] NVARCHAR(50) NOT NULL,

    -- Trạng thái phòng thi
    [TrangThai] NVARCHAR(30) NOT NULL DEFAULT N'SanSang',

    -- Khóa chính của bảng PhongThi
    CONSTRAINT [PK_PhongThi] PRIMARY KEY ([MaPhong]),

    -- Tên phòng không được trùng nhau
    CONSTRAINT [UQ_PhongThi_TenPhong] UNIQUE ([TenPhong]),

    -- Sức chứa phải lớn hơn 0
    CONSTRAINT [CK_PhongThi_SucChua] CHECK ([SucChua] > 0),

    -- Trạng thái phòng chỉ nhận giá trị hợp lệ
    CONSTRAINT [CK_PhongThi_TrangThai] CHECK ([TrangThai] IN (N'SanSang', N'BaoTri', N'NgungSuDung'))
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/715b9e66-6fed-4084-ad95-0b8af66ca435" />


Thấy bảng PhongThi đã được tạo với ràng buộc UNIQUE cho TenPhong và CHECK cho SucChua.

### 1.5.3 Tạo bảng `[LichThi]`

### Bài toán đặt ra
Bảng này là trung tâm của hệ thống, dùng để lưu từng lịch thi cụ thể theo môn, phòng, ngày giờ thi.

### Code SQL

```sql
-- Tạo bảng LichThi để lưu thông tin lịch thi cụ thể
CREATE TABLE [LichThi]
(
    -- Mã lịch thi tự tăng
    [MaLichThi] INT IDENTITY(1,1) NOT NULL,

    -- Mã môn học, khóa ngoại tham chiếu bảng MonHoc
    [MaMonHoc] INT NOT NULL,

    -- Mã phòng thi, khóa ngoại tham chiếu bảng PhongThi
    [MaPhong] INT NOT NULL,

    -- Ngày tổ chức thi
    [NgayThi] DATE NOT NULL,

    -- Giờ bắt đầu thi
    [GioBatDau] TIME NOT NULL,

    -- Giờ kết thúc thi
    [GioKetThuc] TIME NOT NULL,

    -- Số lượng sinh viên tối đa được phép đăng ký lịch này
    [SoLuongToiDa] INT NOT NULL,

    -- Ghi chú thêm nếu cần
    [GhiChu] NVARCHAR(200) NULL,

    -- Khóa chính của bảng LichThi
    CONSTRAINT [PK_LichThi] PRIMARY KEY ([MaLichThi]),

    -- Khóa ngoại liên kết đến bảng MonHoc
    CONSTRAINT [FK_LichThi_MonHoc] FOREIGN KEY ([MaMonHoc]) REFERENCES [MonHoc]([MaMonHoc]),

    -- Khóa ngoại liên kết đến bảng PhongThi
    CONSTRAINT [FK_LichThi_PhongThi] FOREIGN KEY ([MaPhong]) REFERENCES [PhongThi]([MaPhong]),

    -- Giờ kết thúc phải lớn hơn giờ bắt đầu
    CONSTRAINT [CK_LichThi_GioHopLe] CHECK ([GioKetThuc] > [GioBatDau]),

    -- Số lượng tối đa phải lớn hơn 0
    CONSTRAINT [CK_LichThi_SoLuongToiDa] CHECK ([SoLuongToiDa] > 0)
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/eecf75e4-1153-4f34-b13b-ed24cc919786" />

Thấy bảng LichThi có đủ PK, FK và ràng buộc kiểm tra thời gian thi hợp lệ.

### 1.5.4 Tạo bảng `[DangKyThi]`

### Bài toán đặt ra
Cần lưu thông tin sinh viên đăng ký thi ở lịch nào để phục vụ danh sách dự thi và thống kê sĩ số.

### Code SQL

```sql
-- Tạo bảng DangKyThi để lưu danh sách sinh viên đăng ký dự thi
CREATE TABLE [DangKyThi]
(
    -- Mã đăng ký tự tăng
    [MaDangKy] INT IDENTITY(1,1) NOT NULL,

    -- Mã lịch thi mà sinh viên đăng ký
    [MaLichThi] INT NOT NULL,

    -- Mã sinh viên
    [MaSinhVien] NVARCHAR(20) NOT NULL,

    -- Họ tên sinh viên
    [HoTenSinhVien] NVARCHAR(150) NOT NULL,

    -- Ngày giờ đăng ký thi
    [NgayDangKy] DATETIME NOT NULL DEFAULT GETDATE(),

    -- Trạng thái dự thi
    [TrangThaiDuThi] NVARCHAR(30) NOT NULL DEFAULT N'DaDangKy',

    -- Khóa chính bảng DangKyThi
    CONSTRAINT [PK_DangKyThi] PRIMARY KEY ([MaDangKy]),

    -- Khóa ngoại liên kết đến bảng LichThi
    CONSTRAINT [FK_DangKyThi_LichThi] FOREIGN KEY ([MaLichThi]) REFERENCES [LichThi]([MaLichThi]),

    -- Ràng buộc trạng thái dự thi chỉ nhận các giá trị hợp lệ
    CONSTRAINT [CK_DangKyThi_TrangThaiDuThi] CHECK ([TrangThaiDuThi] IN (N'DaDangKy', N'DaHuy', N'DaDuThi', N'VangThi'))
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ec626247-175a-455f-ada1-5ea4ce0c8cc1" />

Thấy bảng DangKyThi đã được tạo thành công để lưu danh sách sinh viên tham gia từng lịch thi.

## 1.6 Giải thích PK, FK, CK

- **PK**:
  - `[MonHoc].[MaMonHoc]`
  - `[PhongThi].[MaPhong]`
  - `[LichThi].[MaLichThi]`
  - `[DangKyThi].[MaDangKy]`

- **FK**:
  - `[LichThi].[MaMonHoc]` → `[MonHoc].[MaMonHoc]`
  - `[LichThi].[MaPhong]` → `[PhongThi].[MaPhong]`
  - `[DangKyThi].[MaLichThi]` → `[LichThi].[MaLichThi]`

- **CK tiêu biểu**:
  - `[SoTinChi] > 0`
  - `[SucChua] > 0`
  - `[GioKetThuc] > [GioBatDau]`
  - `[SoLuongToiDa] > 0`
  - các cột trạng thái chỉ nhận giá trị hợp lệ

## 1.7 Chèn dữ liệu mẫu

### Bài toán đặt ra
Cần có dữ liệu mẫu để kiểm thử function, procedure, trigger và cursor.

### Code SQL

```sql
-- Thêm dữ liệu môn học mẫu
INSERT INTO [MonHoc] ([TenMonHoc], [SoTinChi], [TrangThai])
VALUES
    (N'Cơ sở dữ liệu', 3, N'DangMo'),
    (N'Lập trình Java', 4, N'DangMo'),
    (N'Mạng máy tính', 3, N'DangMo'),
    (N'Hệ điều hành', 4, N'DangMo');
GO

-- Thêm dữ liệu phòng thi mẫu
INSERT INTO [PhongThi] ([TenPhong], [SucChua], [KhuNha], [TrangThai])
VALUES
    (N'A101', 40, N'Nhà A', N'SanSang'),
    (N'A102', 35, N'Nhà A', N'SanSang'),
    (N'B201', 50, N'Nhà B', N'SanSang'),
    (N'C301', 60, N'Nhà C', N'SanSang');
GO

-- Thêm dữ liệu lịch thi mẫu
INSERT INTO [LichThi] ([MaMonHoc], [MaPhong], [NgayThi], [GioBatDau], [GioKetThuc], [SoLuongToiDa], [GhiChu])
VALUES
    (1, 1, '2026-05-10', '07:00', '09:00', 40, N'Thi giữa kỳ'),
    (2, 2, '2026-05-10', '09:30', '11:30', 35, N'Thi cuối kỳ'),
    (3, 3, '2026-05-11', '13:30', '15:30', 50, N'Thi cuối kỳ'),
    (4, 4, '2026-05-12', '07:30', '09:30', 60, N'Thi thực hành');
GO

-- Thêm dữ liệu đăng ký thi mẫu
INSERT INTO [DangKyThi] ([MaLichThi], [MaSinhVien], [HoTenSinhVien], [TrangThaiDuThi])
VALUES
    (1, N'K235480106001', N'Nguyễn Văn An', N'DaDangKy'),
    (1, N'K235480106002', N'Trần Thị Bình', N'DaDangKy'),
    (2, N'K235480106003', N'Lê Văn Cường', N'DaDangKy'),
    (3, N'K235480106004', N'Phạm Thu Dung', N'DaDangKy'),
    (4, N'K235480106005', N'Hoàng Văn Em', N'DaDangKy');
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/64c729bf-df2e-4fc9-8146-e23e12d0f034" />


Thấy dữ liệu mẫu đã được thêm vào đầy đủ để phục vụ việc kiểm thử các chức năng phía sau.

---

# PHẦN 2: XÂY DỰNG FUNCTION

## 2.1 Built-in function trong SQL Server

SQL Server có nhiều nhóm hàm dựng sẵn như:

- Hàm chuỗi: `LEN`, `UPPER`, `LOWER`, `CONCAT`
- Hàm ngày giờ: `GETDATE`, `DATEDIFF`, `DATEADD`, `FORMAT`
- Hàm số học: `ROUND`, `ABS`
- Hàm tổng hợp: `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`
- Hàm hệ thống: `DB_NAME`, `SUSER_NAME`, `@@VERSION`

## 2.2 Một vài system function tiêu biểu

### Bài toán đặt ra
Cần khai thác một vài hàm hệ thống để kiểm tra môi trường SQL Server đang làm việc.

### Code SQL

```sql
-- Lấy tên database hiện tại
SELECT DB_NAME() AS TenDatabaseHienTai;

-- Lấy tài khoản đang đăng nhập SQL Server
SELECT SUSER_NAME() AS TaiKhoanDangNhap;

-- Lấy thời gian hiện tại của hệ thống
SELECT GETDATE() AS ThoiGianHeThong;

-- Lấy thông tin phiên bản SQL Server
SELECT @@VERSION AS ThongTinPhienBan;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/7c6a9168-fca1-40ed-b399-88bc159de945" />

Thấy em đã khai thác được các hàm hệ thống để lấy tên CSDL, tài khoản đăng nhập và thông tin phiên bản SQL Server.

## 2.3 Khai thác built-in function trong đề tài quản lý lịch thi

### Bài toán đặt ra
Trong đề tài lịch thi, cần tính thời lượng ca thi, chuẩn hóa tên phòng, và đếm số sinh viên đã đăng ký.

### Code SQL

```sql
-- Tính thời lượng của mỗi lịch thi theo phút
SELECT
    [MaLichThi],
    [NgayThi],
    [GioBatDau],
    [GioKetThuc],
    DATEDIFF(MINUTE, [GioBatDau], [GioKetThuc]) AS ThoiLuongThiPhut
FROM [LichThi];

-- Chuẩn hóa tên phòng thành chữ in hoa
SELECT
    [MaPhong],
    UPPER([TenPhong]) AS TenPhongVietHoa,
    LEN([TenPhong]) AS DoDaiTenPhong
FROM [PhongThi];

-- Đếm số sinh viên đăng ký cho từng lịch thi
SELECT
    [MaLichThi],
    COUNT(*) AS SoSinhVienDangKy
FROM [DangKyThi]
GROUP BY [MaLichThi];
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/6cf20295-8817-419f-801c-f5102f53bf55" />

Cho thấy các built-in function đã được dùng để tính thời lượng thi, chuẩn hóa tên phòng và đếm số sinh viên dự thi.

## 2.4 User-defined function dùng để làm gì?

Trong bài toán thực tế, có nhiều logic nghiệp vụ mà hàm có sẵn không giải quyết trực tiếp được, ví dụ:

- phân loại ca thi ngắn hay dài,
- lấy danh sách lịch thi theo ngày,
- thống kê mức độ sử dụng phòng thi.

Vì vậy cần viết thêm các hàm riêng để tái sử dụng logic.

## 2.5 Scalar Function

### Bài toán đặt ra
Viết hàm trả về **thời lượng thi tính theo phút** của một lịch thi cụ thể.

### Phân tích sơ qua
Hàm nhận vào mã lịch thi, truy xuất giờ bắt đầu và giờ kết thúc, sau đó dùng `DATEDIFF` để tính số phút. Đây là loại hàm trả về **một giá trị duy nhất**, nên phù hợp với Scalar Function.

### Code SQL

```sql
-- Tạo hàm scalar để tính thời lượng thi theo phút
CREATE OR ALTER FUNCTION [dbo].[fn_TinhThoiLuongThiPhut]
(
    -- Tham số đầu vào là mã lịch thi
    @MaLichThi INT
)
RETURNS INT
AS
BEGIN
    -- Khai báo biến lưu giờ bắt đầu
    DECLARE @GioBatDau TIME;

    -- Khai báo biến lưu giờ kết thúc
    DECLARE @GioKetThuc TIME;

    -- Khai báo biến kết quả thời lượng
    DECLARE @ThoiLuong INT;

    -- Lấy giờ bắt đầu và giờ kết thúc từ bảng LichThi
    SELECT
        @GioBatDau = [GioBatDau],
        @GioKetThuc = [GioKetThuc]
    FROM [LichThi]
    WHERE [MaLichThi] = @MaLichThi;

    -- Nếu không tìm thấy lịch thi thì trả về NULL
    IF @GioBatDau IS NULL OR @GioKetThuc IS NULL
        RETURN NULL;

    -- Tính số phút giữa giờ bắt đầu và giờ kết thúc
    SET @ThoiLuong = DATEDIFF(MINUTE, @GioBatDau, @GioKetThuc);

    -- Trả về kết quả thời lượng thi
    RETURN @ThoiLuong;
END;
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d59034e8-5833-4cb5-8456-60212e472840" />

Tạo ra hàm mới 

### Khai thác hàm

```sql
-- Gọi hàm để xem thời lượng thi của từng lịch thi
SELECT
    [MaLichThi],
    [NgayThi],
    [GioBatDau],
    [GioKetThuc],
    [dbo].[fn_TinhThoiLuongThiPhut]([MaLichThi]) AS ThoiLuongThiPhut
FROM [LichThi];
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/052b323f-f10a-4c69-ab74-f553edc33497" />

Cho thấy hàm scalar đã tính chính xác thời lượng của từng lịch thi theo đơn vị phút.

## 2.6 Inline Table-Valued Function

### Bài toán đặt ra
Viết hàm trả về danh sách các lịch thi diễn ra trong một ngày cụ thể.

### Phân tích sơ qua
Hàm chỉ cần một câu `SELECT` để trả về nhiều dòng dữ liệu, vì vậy phù hợp với **Inline Table-Valued Function**.

### Code SQL

```sql
-- Tạo hàm inline TVF để lấy danh sách lịch thi theo ngày
CREATE OR ALTER FUNCTION [dbo].[fn_LayLichThiTheoNgay]
(
    -- Tham số đầu vào là ngày cần tra cứu lịch thi
    @NgayThi DATE
)
RETURNS TABLE
AS
RETURN
(
    -- Trả về danh sách lịch thi theo ngày cùng thông tin môn học và phòng thi
    SELECT
        lt.[MaLichThi],
        mh.[TenMonHoc],
        pt.[TenPhong],
        lt.[NgayThi],
        lt.[GioBatDau],
        lt.[GioKetThuc],
        lt.[SoLuongToiDa],
        [dbo].[fn_TinhThoiLuongThiPhut](lt.[MaLichThi]) AS ThoiLuongThiPhut
    FROM [LichThi] lt
    INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc]
    INNER JOIN [PhongThi] pt ON lt.[MaPhong] = pt.[MaPhong]
    WHERE lt.[NgayThi] = @NgayThi
);
GO
```

### Khai thác hàm

```sql
-- Lấy toàn bộ lịch thi diễn ra trong ngày 10/05/2026
SELECT *
FROM [dbo].[fn_LayLichThiTheoNgay]('2026-05-10');
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b1c7665b-5c6b-4ea5-a481-64af3c109c5e" />


Cho thấy hàm đã trả về đúng danh sách các lịch thi trong ngày được yêu cầu.

## 2.7 Multi-statement Table-Valued Function

### Bài toán đặt ra
Viết hàm thống kê mức sử dụng phòng thi: mỗi phòng hiện có bao nhiêu lịch thi, tổng số chỗ ngồi được phân bổ và trạng thái sử dụng của phòng.

### Phân tích sơ qua
Bài toán cần nhiều bước xử lý hơn một câu `SELECT` đơn giản, do đó phù hợp với **Multi-statement TVF**.

### Code SQL

```sql
-- Tạo hàm multi-statement TVF để thống kê sử dụng phòng thi
CREATE OR ALTER FUNCTION [dbo].[fn_ThongKeSuDungPhongThi]()
RETURNS @KetQua TABLE
(
    -- Mã phòng thi
    [MaPhong] INT,

    -- Tên phòng thi
    [TenPhong] NVARCHAR(50),

    -- Số lịch thi đã xếp vào phòng
    [SoLichThi] INT,

    -- Tổng số chỗ tối đa đã phân bổ cho các lịch thi ở phòng đó
    [TongChoPhanBo] INT,

    -- Đánh giá mức sử dụng phòng
    [MucDoSuDung] NVARCHAR(50)
)
AS
BEGIN
    -- Ghi dữ liệu thống kê vào bảng kết quả
    INSERT INTO @KetQua
    SELECT
        pt.[MaPhong],
        pt.[TenPhong],
        COUNT(lt.[MaLichThi]) AS SoLichThi,
        ISNULL(SUM(lt.[SoLuongToiDa]), 0) AS TongChoPhanBo,
        CASE
            WHEN COUNT(lt.[MaLichThi]) = 0 THEN N'ChuaSuDung'
            WHEN COUNT(lt.[MaLichThi]) <= 1 THEN N'SuDungIt'
            ELSE N'SuDungNhieu'
        END AS MucDoSuDung
    FROM [PhongThi] pt
    LEFT JOIN [LichThi] lt ON pt.[MaPhong] = lt.[MaPhong]
    GROUP BY pt.[MaPhong], pt.[TenPhong];

    -- Trả về bảng kết quả
    RETURN;
END;
GO
```

### Khai thác hàm

```sql
-- Gọi hàm thống kê mức sử dụng phòng thi
SELECT *
FROM [dbo].[fn_ThongKeSuDungPhongThi]();
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a0498710-7df0-4639-b6a1-63e3152c0f1c" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c3c502d3-d502-4287-b57a-73f965aed397" />

Hàm đã thống kê được số lịch thi, tổng chỗ phân bổ và mức sử dụng của từng phòng.

---

# PHẦN 3: XÂY DỰNG STORED PROCEDURE

## 3.1 Một vài system stored procedure tiêu biểu

### Code SQL

```sql
-- Xem cấu trúc bảng LichThi
EXEC sp_help 'LichThi';

-- Xem các ràng buộc trên bảng DangKyThi
EXEC sp_helpconstraint 'DangKyThi';
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3ef103fd-d236-4b41-8919-ad6b208bd037" />
Ảnh này cho thấy SQL Server hỗ trợ các system stored procedure để kiểm tra cấu trúc bảng và ràng buộc dữ liệu.

## 3.2 Stored Procedure 1: thêm lịch thi có kiểm tra logic

### Bài toán đặt ra
Khi thêm lịch thi mới cần kiểm tra:

- môn học có tồn tại hay không,
- phòng thi có tồn tại và sẵn sàng hay không,
- giờ kết thúc phải lớn hơn giờ bắt đầu,
- số lượng tối đa không vượt quá sức chứa phòng.

### Phân tích sơ qua
Procedure này dùng để xử lý nghiệp vụ **INSERT có kiểm tra điều kiện** trước khi thêm dữ liệu vào bảng.

### Code SQL

```sql
-- Tạo procedure thêm lịch thi mới có kiểm tra điều kiện logic
CREATE OR ALTER PROCEDURE [dbo].[sp_ThemLichThi]
    -- Mã môn học cần xếp lịch thi
    @MaMonHoc INT,

    -- Mã phòng thi được chọn
    @MaPhong INT,

    -- Ngày thi
    @NgayThi DATE,

    -- Giờ bắt đầu thi
    @GioBatDau TIME,

    -- Giờ kết thúc thi
    @GioKetThuc TIME,

    -- Số lượng sinh viên tối đa
    @SoLuongToiDa INT,

    -- Ghi chú thêm
    @GhiChu NVARCHAR(200) = NULL
AS
BEGIN
    SET NOCOUNT ON;

    -- Kiểm tra môn học có tồn tại không
    IF NOT EXISTS (SELECT 1 FROM [MonHoc] WHERE [MaMonHoc] = @MaMonHoc)
    BEGIN
        RAISERROR(N'Môn học không tồn tại.', 16, 1);
        RETURN;
    END;

    -- Kiểm tra phòng thi có tồn tại và sẵn sàng không
    IF NOT EXISTS (
        SELECT 1
        FROM [PhongThi]
        WHERE [MaPhong] = @MaPhong
          AND [TrangThai] = N'SanSang'
    )
    BEGIN
        RAISERROR(N'Phòng thi không tồn tại hoặc chưa sẵn sàng sử dụng.', 16, 1);
        RETURN;
    END;

    -- Kiểm tra giờ thi hợp lệ
    IF @GioKetThuc <= @GioBatDau
    BEGIN
        RAISERROR(N'Giờ kết thúc phải lớn hơn giờ bắt đầu.', 16, 1);
        RETURN;
    END;

    -- Kiểm tra số lượng tối đa không vượt sức chứa phòng
    IF EXISTS (
        SELECT 1
        FROM [PhongThi]
        WHERE [MaPhong] = @MaPhong
          AND [SucChua] < @SoLuongToiDa
    )
    BEGIN
        RAISERROR(N'Số lượng tối đa vượt quá sức chứa của phòng thi.', 16, 1);
        RETURN;
    END;

    -- Thêm lịch thi mới vào bảng LichThi
    INSERT INTO [LichThi]
    (
        [MaMonHoc],
        [MaPhong],
        [NgayThi],
        [GioBatDau],
        [GioKetThuc],
        [SoLuongToiDa],
        [GhiChu]
    )
    VALUES
    (
        @MaMonHoc,
        @MaPhong,
        @NgayThi,
        @GioBatDau,
        @GioKetThuc,
        @SoLuongToiDa,
        @GhiChu
    );
END;
GO
```

### Khai thác procedure

```sql
-- Thêm một lịch thi mới hợp lệ
EXEC [dbo].[sp_ThemLichThi]
    @MaMonHoc = 1,
    @MaPhong = 2,
    @NgayThi = '2026-05-15',
    @GioBatDau = '13:00',
    @GioKetThuc = '15:00',
    @SoLuongToiDa = 30,
    @GhiChu = N'Thi bổ sung';
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/80a4235b-3d77-4406-8a45-a8115cb6d8cb" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/0764fb18-5d17-4e9e-9a8d-7c255bdb8101" />

Procedure đã kiểm tra điều kiện nghiệp vụ trước khi thêm lịch thi mới.

## 3.3 Stored Procedure 2: dùng OUTPUT để trả về giá trị tính toán

### Bài toán đặt ra
Viết procedure tính số sinh viên đã đăng ký ở một lịch thi và trả kết quả qua tham số OUTPUT.

### Phân tích sơ qua
Bài toán yêu cầu procedure trả về một giá trị tổng hợp, nên dùng tham số OUTPUT là phù hợp.

### Code SQL

```sql
-- Tạo procedure đếm số sinh viên đã đăng ký ở một lịch thi
CREATE OR ALTER PROCEDURE [dbo].[sp_DemSoSinhVienDangKy]
    -- Mã lịch thi cần thống kê
    @MaLichThi INT,

    -- Biến OUTPUT trả về số sinh viên đã đăng ký
    @SoSinhVienDangKy INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- Đếm số lượng đăng ký còn hiệu lực ở lịch thi
    SELECT @SoSinhVienDangKy = COUNT(*)
    FROM [DangKyThi]
    WHERE [MaLichThi] = @MaLichThi
      AND [TrangThaiDuThi] = N'DaDangKy';
END;
GO
```

### Khai thác procedure

```sql
-- Khai báo biến nhận kết quả OUTPUT
DECLARE @SoLuong INT;

-- Gọi procedure
EXEC [dbo].[sp_DemSoSinhVienDangKy]
    @MaLichThi = 1,
    @SoSinhVienDangKy = @SoLuong OUTPUT;

-- Xem kết quả trả về
SELECT @SoLuong AS SoSinhVienDangKy;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8383c3f4-4f20-415a-9cc6-e3e75c7dd2a1" />

<img width="1918" height="1079" alt="image" src="https://github.com/user-attachments/assets/55abfae9-02c0-4b39-a3ea-c7114badd4f1" />

Procedure đã trả về số lượng sinh viên đăng ký qua tham số OUTPUT.

## 3.4 Stored Procedure 3: trả về result set sau khi join nhiều bảng

### Bài toán đặt ra
Viết procedure báo cáo chi tiết danh sách dự thi gồm môn học, phòng thi, ngày giờ thi và sinh viên dự thi.

### Phân tích sơ qua
Vì cần join nhiều bảng và trả về một tập kết quả, đây là kiểu bài phù hợp với procedure trả về result set.

### Code SQL

```sql
-- Tạo procedure báo cáo chi tiết danh sách dự thi
CREATE OR ALTER PROCEDURE [dbo].[sp_BaoCaoDanhSachDuThi]
AS
BEGIN
    SET NOCOUNT ON;

    -- Trả về danh sách dự thi sau khi join 4 bảng
    SELECT
        dk.[MaDangKy],
        dk.[MaSinhVien],
        dk.[HoTenSinhVien],
        mh.[TenMonHoc],
        pt.[TenPhong],
        lt.[NgayThi],
        lt.[GioBatDau],
        lt.[GioKetThuc],
        dk.[TrangThaiDuThi]
    FROM [DangKyThi] dk
    INNER JOIN [LichThi] lt ON dk.[MaLichThi] = lt.[MaLichThi]
    INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc]
    INNER JOIN [PhongThi] pt ON lt.[MaPhong] = pt.[MaPhong]
    ORDER BY lt.[NgayThi], lt.[GioBatDau], dk.[HoTenSinhVien];
END;
GO
```

### Khai thác procedure

```sql
-- Xem toàn bộ danh sách dự thi chi tiết
EXEC [dbo].[sp_BaoCaoDanhSachDuThi];
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/79a208c5-c4c9-42ba-bd94-75a6b7ba7a6a" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2fd7ca4a-eebb-4f30-80b8-d3498d610b61" />

Procedure đã join nhiều bảng để tạo ra báo cáo danh sách dự thi đầy đủ.

---

# PHẦN 4: TRIGGER VÀ XỬ LÝ LOGIC NGHIỆP VỤ

## 4.1 Trigger 1: khi đăng ký thi thì kiểm tra số lượng tối đa

### Bài toán đặt ra
Khi sinh viên đăng ký thi, hệ thống phải tự động kiểm tra xem lịch thi đó còn chỗ hay không. Nếu vượt quá số lượng tối đa thì phải từ chối đăng ký.

### Phân tích sơ qua
Trigger này giúp kiểm soát dữ liệu ngay tại database, tránh vượt chỉ tiêu phòng thi.

### Code SQL

```sql
-- Tạo trigger kiểm tra số lượng đăng ký trước khi chèn dữ liệu mới
CREATE OR ALTER TRIGGER [trg_DangKyThi_KiemTraSoLuong]
ON [DangKyThi]
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu tồn tại lịch thi bị vượt quá số lượng tối đa sau khi đăng ký mới
    IF EXISTS
    (
        SELECT 1
        FROM [LichThi] lt
        INNER JOIN inserted i ON lt.[MaLichThi] = i.[MaLichThi]
        WHERE
            (
                SELECT COUNT(*)
                FROM [DangKyThi] dk
                WHERE dk.[MaLichThi] = lt.[MaLichThi]
                  AND dk.[TrangThaiDuThi] = N'DaDangKy'
            ) > lt.[SoLuongToiDa]
    )
    BEGIN
        -- Báo lỗi và hủy giao dịch nếu vượt quá số lượng tối đa
        RAISERROR(N'Lịch thi đã đủ số lượng sinh viên đăng ký.', 16, 1);
        ROLLBACK TRANSACTION;
        RETURN;
    END;
END;
GO
```

### Khai thác trigger

```sql
-- Thử thêm đăng ký thi mới
INSERT INTO [DangKyThi]
(
    [MaLichThi],
    [MaSinhVien],
    [HoTenSinhVien],
    [TrangThaiDuThi]
)
VALUES
(
    1,
    N'K235480106099',
    N'Sinh viên thử nghiệm',
    N'DaDangKy'
);
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/14c65f74-0a1e-4862-b15e-0e537ddf1dce" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4b575c40-aac5-4648-9f67-d0d26eeb0637" />

Ảnh này cho thấy khi số lượng đăng ký vượt mức cho phép, trigger đã chặn thao tác insert.

## 4.2 Trigger 2: khi thêm lịch thi thì kiểm tra trùng giờ cùng phòng

### Bài toán đặt ra
Không được phép xếp hai lịch thi trùng thời gian trong cùng một phòng thi.

### Phân tích sơ qua
Trigger kiểm tra dữ liệu vừa insert với các lịch thi đã có, nếu cùng phòng và giao nhau về thời gian thì báo lỗi.

### Code SQL

```sql
-- Tạo trigger kiểm tra trùng giờ thi trong cùng một phòng
CREATE OR ALTER TRIGGER [trg_LichThi_KiemTraTrungPhong]
ON [LichThi]
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu tồn tại lịch thi mới bị trùng giờ với lịch cũ trong cùng phòng
    IF EXISTS
    (
        SELECT 1
        FROM inserted i
        INNER JOIN [LichThi] lt
            ON i.[MaPhong] = lt.[MaPhong]
           AND i.[NgayThi] = lt.[NgayThi]
           AND i.[MaLichThi] <> lt.[MaLichThi]
           AND i.[GioBatDau] < lt.[GioKetThuc]
           AND i.[GioKetThuc] > lt.[GioBatDau]
    )
    BEGIN
        -- Báo lỗi và hủy giao dịch nếu trùng phòng, trùng giờ
        RAISERROR(N'Phòng thi đã có lịch khác trong khoảng thời gian này.', 16, 1);
        ROLLBACK TRANSACTION;
        RETURN;
    END;
END;
GO
```

### Khai thác trigger

```sql
-- Thử thêm lịch thi có thời gian bị chồng lấn với phòng đã dùng
INSERT INTO [LichThi]
(
    [MaMonHoc],
    [MaPhong],
    [NgayThi],
    [GioBatDau],
    [GioKetThuc],
    [SoLuongToiDa],
    [GhiChu]
)
VALUES
(
    2,
    1,
    '2026-05-10',
    '08:00',
    '10:00',
    20,
    N'Test trùng phòng'
);
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/41220827-00fd-4e94-b709-bd050279e780" />

  Cho thấy trigger đã phát hiện lịch thi mới bị trùng giờ cùng phòng và tự động hủy giao dịch.

## 4.3 Mô phỏng trigger đệ quy giữa hai bảng

### Bài toán đặt ra
Tạo trigger trên bảng A cập nhật bảng B, rồi trigger trên bảng B lại cập nhật bảng A để quan sát phản ứng của SQL Server.

Trong đề tài này, em mô phỏng:

- Bảng A: `[LichThi]`
- Bảng B: `[PhongThi]`

### Code SQL

```sql
-- Bật cơ chế recursive trigger để mô phỏng tình huống đệ quy
ALTER DATABASE CURRENT SET RECURSIVE_TRIGGERS ON;
GO

-- Nếu trigger cũ đã tồn tại thì xóa đi
IF OBJECT_ID('trg_LichThi_UpdatePhongThi', 'TR') IS NOT NULL
    DROP TRIGGER trg_LichThi_UpdatePhongThi;
GO

-- Nếu trigger cũ đã tồn tại thì xóa đi
IF OBJECT_ID('trg_PhongThi_UpdateLichThi', 'TR') IS NOT NULL
    DROP TRIGGER trg_PhongThi_UpdateLichThi;
GO

-- Trigger trên bảng LichThi: khi update thì update bảng PhongThi
CREATE TRIGGER trg_LichThi_UpdatePhongThi
ON [LichThi]
AFTER UPDATE
AS
BEGIN
    UPDATE pt
    SET pt.[SucChua] = pt.[SucChua] + 1
    FROM [PhongThi] pt
    INNER JOIN inserted i ON pt.[MaPhong] = i.[MaPhong];
END;
GO

-- Trigger trên bảng PhongThi: khi update thì update ngược lại bảng LichThi
CREATE TRIGGER trg_PhongThi_UpdateLichThi
ON [PhongThi]
AFTER UPDATE
AS
BEGIN
    UPDATE lt
    SET lt.[SoLuongToiDa] = lt.[SoLuongToiDa] + 1
    FROM [LichThi] lt
    INNER JOIN inserted i ON lt.[MaPhong] = i.[MaPhong];
END;
GO

-- Câu lệnh thử nghiệm gây đệ quy gián tiếp
UPDATE [LichThi]
SET [SoLuongToiDa] = [SoLuongToiDa] + 1
WHERE [MaLichThi] = 1;
GO
```

### Nhận xét

Khi thực hiện mô hình này, SQL Server có thể phát sinh lỗi như:

```text
Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32).
```

### Kết luận

- Trigger cập nhật chéo giữa 2 bảng rất dễ tạo vòng lặp đệ quy.
- Khi vượt quá số mức lồng nhau cho phép, SQL Server sẽ dừng giao dịch.
- Trong thiết kế thực tế cần tránh kiểu cập nhật qua lại không có điều kiện dừng.


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/37603b39-cd0e-4127-a349-8717ec02fff2" />

 Ảnh này cho thấy trigger đệ quy có thể gây lỗi lồng nhau quá mức và làm hủy transaction.

---

# PHẦN 5: CURSOR VÀ DUYỆT DỮ LIỆU

## 5.1 Bài toán dùng CURSOR

### Bài toán đặt ra
Duyệt từng lịch thi, tính số lượng sinh viên đăng ký thực tế và in ra cảnh báo nếu lịch thi gần đầy hoặc đã đầy.

### Phân tích sơ qua
Bài toán này có thể xử lý theo từng dòng để tạo thông báo riêng cho từng lịch thi, vì vậy phù hợp để minh họa CURSOR.

### Code SQL

```sql
-- Khai báo các biến dùng trong quá trình duyệt CURSOR
DECLARE @MaLichThi INT,
        @TenMonHoc NVARCHAR(150),
        @SoLuongToiDa INT,
        @SoLuongDangKy INT,
        @ThongBao NVARCHAR(200);

-- Khai báo cursor lấy danh sách lịch thi và sức chứa tối đa
DECLARE cur_LichThi CURSOR FOR
SELECT
    lt.[MaLichThi],
    mh.[TenMonHoc],
    lt.[SoLuongToiDa]
FROM [LichThi] lt
INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc];

-- Mở cursor
OPEN cur_LichThi;

-- Đọc dòng đầu tiên
FETCH NEXT FROM cur_LichThi INTO @MaLichThi, @TenMonHoc, @SoLuongToiDa;

-- Lặp cho đến khi đọc hết dữ liệu
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Đếm số sinh viên đã đăng ký ở lịch thi hiện tại
    SELECT @SoLuongDangKy = COUNT(*)
    FROM [DangKyThi]
    WHERE [MaLichThi] = @MaLichThi
      AND [TrangThaiDuThi] = N'DaDangKy';

    -- Xác định thông báo theo mức độ đầy phòng
    IF @SoLuongDangKy >= @SoLuongToiDa
        SET @ThongBao = N'Da day phong';
    ELSE IF @SoLuongDangKy >= @SoLuongToiDa * 0.8
        SET @ThongBao = N'Sap day phong';
    ELSE
        SET @ThongBao = N'Con cho';

    -- In thông tin từng lịch thi
    PRINT N'Ma lich thi: ' + CAST(@MaLichThi AS NVARCHAR)
        + N' | Mon: ' + @TenMonHoc
        + N' | So da dang ky: ' + CAST(@SoLuongDangKy AS NVARCHAR)
        + N' / ' + CAST(@SoLuongToiDa AS NVARCHAR)
        + N' | Trang thai: ' + @ThongBao;

    -- Đọc tiếp dòng tiếp theo
    FETCH NEXT FROM cur_LichThi INTO @MaLichThi, @TenMonHoc, @SoLuongToiDa;
END;

-- Đóng cursor
CLOSE cur_LichThi;

-- Giải phóng cursor khỏi bộ nhớ
DEALLOCATE cur_LichThi;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bb01c7a4-ec95-490b-a663-1a8717e1f8c1" />

Cho thấy CURSOR đã duyệt từng lịch thi và in cảnh báo riêng theo mức độ đầy phòng.

## 5.2 Giải bài toán không dùng CURSOR

### Bài toán đặt ra
Cùng bài toán trên, thử giải theo kiểu set-based để so sánh với CURSOR.

### Code SQL

```sql
-- Thống kê mức độ đầy phòng của từng lịch thi mà không cần dùng cursor
SELECT
    lt.[MaLichThi],
    mh.[TenMonHoc],
    lt.[SoLuongToiDa],
    COUNT(dk.[MaDangKy]) AS SoLuongDangKy,
    CASE
        WHEN COUNT(dk.[MaDangKy]) >= lt.[SoLuongToiDa] THEN N'Da day phong'
        WHEN COUNT(dk.[MaDangKy]) >= lt.[SoLuongToiDa] * 0.8 THEN N'Sap day phong'
        ELSE N'Con cho'
    END AS TrangThaiPhongThi
FROM [LichThi] lt
INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc]
LEFT JOIN [DangKyThi] dk
    ON lt.[MaLichThi] = dk.[MaLichThi]
   AND dk.[TrangThaiDuThi] = N'DaDangKy'
GROUP BY lt.[MaLichThi], mh.[TenMonHoc], lt.[SoLuongToiDa];
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1a0c3cd7-6698-4969-84db-65177a960750" />

Ảnh này cho thấy cùng một bài toán có thể giải bằng truy vấn set-based mà không cần dùng CURSOR.

## 5.3 So sánh tốc độ giữa CURSOR và không dùng CURSOR

### Code SQL

```sql
-- Bật chế độ thống kê thời gian thực thi câu lệnh
SET STATISTICS TIME ON;

-- Chạy đoạn xử lý bằng cursor
-- Khai báo các biến dùng trong quá trình duyệt CURSOR
DECLARE @MaLichThi INT,
        @TenMonHoc NVARCHAR(150),
        @SoLuongToiDa INT,
        @SoLuongDangKy INT,
        @ThongBao NVARCHAR(200);

-- Khai báo cursor lấy danh sách lịch thi và sức chứa tối đa
DECLARE cur_LichThi CURSOR FOR
SELECT
    lt.[MaLichThi],
    mh.[TenMonHoc],
    lt.[SoLuongToiDa]
FROM [LichThi] lt
INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc];

-- Mở cursor
OPEN cur_LichThi;

-- Đọc dòng đầu tiên
FETCH NEXT FROM cur_LichThi INTO @MaLichThi, @TenMonHoc, @SoLuongToiDa;

-- Lặp cho đến khi đọc hết dữ liệu
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Đếm số sinh viên đã đăng ký ở lịch thi hiện tại
    SELECT @SoLuongDangKy = COUNT(*)
    FROM [DangKyThi]
    WHERE [MaLichThi] = @MaLichThi
      AND [TrangThaiDuThi] = N'DaDangKy';

    -- Xác định thông báo theo mức độ đầy phòng
    IF @SoLuongDangKy >= @SoLuongToiDa
        SET @ThongBao = N'Da day phong';
    ELSE IF @SoLuongDangKy >= @SoLuongToiDa * 0.8
        SET @ThongBao = N'Sap day phong';
    ELSE
        SET @ThongBao = N'Con cho';

    -- In thông tin từng lịch thi
    PRINT N'Ma lich thi: ' + CAST(@MaLichThi AS NVARCHAR)
        + N' | Mon: ' + @TenMonHoc
        + N' | So da dang ky: ' + CAST(@SoLuongDangKy AS NVARCHAR)
        + N' / ' + CAST(@SoLuongToiDa AS NVARCHAR)
        + N' | Trang thai: ' + @ThongBao;

    -- Đọc tiếp dòng tiếp theo
    FETCH NEXT FROM cur_LichThi INTO @MaLichThi, @TenMonHoc, @SoLuongToiDa;
END;

-- Đóng cursor
CLOSE cur_LichThi;

-- Giải phóng cursor khỏi bộ nhớ
DEALLOCATE cur_LichThi;
-- Chạy đoạn xử lý bằng set-based
-- Thống kê mức độ đầy phòng của từng lịch thi mà không cần dùng cursor
SELECT
    lt.[MaLichThi],
    mh.[TenMonHoc],
    lt.[SoLuongToiDa],
    COUNT(dk.[MaDangKy]) AS SoLuongDangKy,
    CASE
        WHEN COUNT(dk.[MaDangKy]) >= lt.[SoLuongToiDa] THEN N'Da day phong'
        WHEN COUNT(dk.[MaDangKy]) >= lt.[SoLuongToiDa] * 0.8 THEN N'Sap day phong'
        ELSE N'Con cho'
    END AS TrangThaiPhongThi
FROM [LichThi] lt
INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc]
LEFT JOIN [DangKyThi] dk
    ON lt.[MaLichThi] = dk.[MaLichThi]
   AND dk.[TrangThaiDuThi] = N'DaDangKy'
GROUP BY lt.[MaLichThi], mh.[TenMonHoc], lt.[SoLuongToiDa];
-- Tắt chế độ thống kê thời gian
SET STATISTICS TIME OFF;
```

### Nhận xét

- Với dữ liệu nhỏ, cả hai cách đều cho cùng kết quả.
- Khi dữ liệu lớn, cách **không dùng CURSOR** thường nhanh hơn.
- CURSOR xử lý từng dòng nên chậm và tốn tài nguyên hơn.
- Set-based phù hợp hơn với bài toán thống kê tổng hợp trong SQL Server.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bf549e1a-d937-424a-894b-746baa39e971" />
cho thấy truy vấn set-based có thời gian xử lý tốt hơn so với CURSOR trong bài toán thống kê lịch thi.

## 5.4 Một bài toán mà CURSOR phù hợp hơn

### Bài toán đặt ra
Duyệt từng lịch thi sắp đầy phòng để gửi thông báo riêng cho bộ phận khảo thí.

### Phân tích sơ qua
Bài toán này phù hợp với CURSOR vì mỗi lịch thi cần tạo một nội dung cảnh báo riêng, có thể kết hợp gửi email hoặc ghi log từng trường hợp.

### Code SQL

```sql
-- Khai báo biến phục vụ việc duyệt các lịch thi cần cảnh báo
DECLARE @MaLichThiCanhBao INT,
        @TenMonCanhBao NVARCHAR(150),
        @SoLuongToiDaCanhBao INT,
        @SoLuongDangKyCanhBao INT;

-- Khai báo cursor lấy các lịch thi gần đầy hoặc đã đầy
DECLARE cur_CanhBao CURSOR FOR
SELECT
    lt.[MaLichThi],
    mh.[TenMonHoc],
    lt.[SoLuongToiDa],
    COUNT(dk.[MaDangKy]) AS SoLuongDangKy
FROM [LichThi] lt
INNER JOIN [MonHoc] mh ON lt.[MaMonHoc] = mh.[MaMonHoc]
LEFT JOIN [DangKyThi] dk
    ON lt.[MaLichThi] = dk.[MaLichThi]
   AND dk.[TrangThaiDuThi] = N'DaDangKy'
GROUP BY lt.[MaLichThi], mh.[TenMonHoc], lt.[SoLuongToiDa]
HAVING COUNT(dk.[MaDangKy]) >= lt.[SoLuongToiDa] * 0.8;

-- Mở cursor
OPEN cur_CanhBao;

-- Đọc dòng đầu tiên
FETCH NEXT FROM cur_CanhBao INTO @MaLichThiCanhBao, @TenMonCanhBao, @SoLuongToiDaCanhBao, @SoLuongDangKyCanhBao;

-- Lặp qua từng lịch thi cần cảnh báo
WHILE @@FETCH_STATUS = 0
BEGIN
    -- In thông báo cảnh báo riêng cho từng lịch thi
    PRINT N'Canh bao lich thi ' + CAST(@MaLichThiCanhBao AS NVARCHAR)
        + N' - Mon ' + @TenMonCanhBao
        + N' dang co ' + CAST(@SoLuongDangKyCanhBao AS NVARCHAR)
        + N' / ' + CAST(@SoLuongToiDaCanhBao AS NVARCHAR)
        + N' sinh vien dang ky.';

    -- Đọc tiếp dòng tiếp theo
    FETCH NEXT FROM cur_CanhBao INTO @MaLichThiCanhBao, @TenMonCanhBao, @SoLuongToiDaCanhBao, @SoLuongDangKyCanhBao;
END;

-- Đóng và giải phóng cursor
CLOSE cur_CanhBao;
DEALLOCATE cur_CanhBao;
```

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/9ec10e8e-2b0e-436f-a76e-fd5b4b75fd54" />

CURSOR phù hợp khi cần tạo thông báo riêng cho từng lịch thi sắp đầy hoặc đã đầy.

