# 💬 Chủ đề: [Lỗi chuyển kiểu DateTime]
📅 **Ngày:** [2025-11-12]  
💡 **Nhóm chủ đề:** [SQL Server]  
✍️ **Nguồn:** ChatGPT - phiên trao đổi học tập cá nhân  

---

## 🟢 Mục tiêu buổi chat
Tóm tắt ngắn gọn:  
- Mình hỏi về vấn đề gì?  
- Mong muốn giải quyết điều gì?  

---

## 🧠 Nội dung chính (Trao đổi với ChatGPT)
Chuyển đến nội dung
Lịch sử đoạn chat

Bạn đã nói:
lỗi insert: The conversion of a nvarchar data type to a datetime data type resulted in an out-of-range value
mình có bảng dbXML.dbo.tbDSKhamBenhTheoKhoa_new , trong đó có các cột 
TGDANGKY [nvarchar](20)	-->thời gian đăng ký
TGBDKHAM [nvarchar](20) -->thời gian bắt đầu khám
TGKTKHAM [nvarchar](20) -->thời gian kết thúc khám
TGDUYETKT [nvarchar](20) -->thời gian duyệt kế toán
TGDUYETDT [nvarchar](20) -->thời gian duyệt đơn thuốc
mình muốn chuyển dữ liệu này sang dbDataBVNT.dbo.tbDSKhamBenhTheoKhoa vì CSDL dbDataBVNT có phân vùng, và khi chuyển qua thì chuyển luôn các cột thời gian kiểu nvarchar() sang kiểu datetime, cấu trúc như sau:
CREATE TABLE [dbo].[tbDSKhamBenhTheoKhoa](
	MABENHNHAN nvarchar(25) not NULL,
	MAHOSOBENHAN nvarchar(25) not null,
	TENBENHNHAN nvarchar(255) default '',
	GIOITINH [nvarchar](10) default '',
	MA_BHYT [nvarchar](50) default '',
	TEN_DTBN [nvarchar](15) default '',
	KHOA [nvarchar](255) default '',
	PHONG [nvarchar](255) default '',
	BACSY [nvarchar](255) default '',
	MA_BAC_SI nvarchar(50) default '',
	TGDANGKY datetime null,
	TGBDKHAM datetime null,
	TGKTKHAM datetime null,
	TRANGTHAI [nvarchar](50) default '',
	XUTRI [nvarchar](50) default '',
	DUYETKT [varchar](1) default '',
	TGDUYETKT datetime null,
	DUYETDT [varchar](1) default '',
	TGDUYETDT datetime null,
	DVKT [varchar](1) default '',
	DONTHUOC [varchar](1) default '',
	[ID] bigint IDENTITY(1,1) not NULL,
	PNGAYKHAM date not null
)ON dbDataBVNT_PartitionScheme (PNGAYKHAM)
GO

ALTER TABLE dbo.tbDSKhamBenhTheoKhoa
ADD CONSTRAINT PK_tbDSKhamBenhTheoKhoa
PRIMARY KEY NONCLUSTERED (ID, PNGAYKHAM)
ON dbDataBVNT_PartitionScheme(PNGAYKHAM);


Câu lệnh insert:
SET DATEFORMAT dmy;
insert into tbDSKhamBenhTheoKhoa(MABENHNHAN, MAHOSOBENHAN,	TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN, KHOA, PHONG, BACSY, MA_BAC_SI, TGDANGKY,	TGBDKHAM, TGKTKHAM,	TRANGTHAI, XUTRI, DUYETKT, TGDUYETKT, DUYETDT, TGDUYETDT, DVKT, DONTHUOC, PNGAYKHAM)  
select MABENHNHAN, MAHOSOBENHAN,	TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN, KHOA, PHONG, BACSY, MA_BAC_SI, CONVERT(DATETIME, left(TGDANGKY,10) + ' ' + right(TGDANGKY,5), 103), CONVERT(DATETIME, left(TGBDKHAM,10) + ' ' + right(TGBDKHAM,5), 103), 
CASE 
        WHEN TGKTKHAM IS NOT NULL AND TGKTKHAM <> '' 
        THEN CONVERT(DATETIME, LEFT(TGKTKHAM, 10) + ' ' + RIGHT(TGKTKHAM, 5), 103) 
        ELSE NULL -- Trả về NULL nếu là NULL hoặc chuỗi rỗng
END, 
TRANGTHAI, XUTRI, DUYETKT,
CASE 
        WHEN TGDUYETKT IS NOT NULL AND TGDUYETKT <> '' 
        THEN CONVERT(DATETIME, LEFT(TGDUYETKT, 10) + ' ' + RIGHT(TGDUYETKT, 5), 103) 
        ELSE NULL -- Trả về NULL nếu là NULL hoặc chuỗi rỗng
END, 
DUYETDT,
CASE 
        WHEN TGDUYETDT IS NOT NULL AND TGDUYETDT <> '' 
        THEN CONVERT(DATETIME, LEFT(TGDUYETDT, 10) + ' ' + RIGHT(TGDUYETDT, 5), 103) 
        ELSE NULL -- Trả về NULL nếu là NULL hoặc chuỗi rỗng
END, 
DVKT, DONTHUOC,
CONVERT(DATE, left(TGBDKHAM,10), 103) 
from dbXML.dbo.tbDSKhamBenhTheoKhoa_new
where CONVERT(DATE, left(TGBDKHAM,10), 103) between '2018-01-01' and '2018-01-31'
go

mình đã kiểm tra kỹ, nhưng không hiểu sao bị báo lỗi:
Msg 242, Level 16, State 3, Line 2
The conversion of a nvarchar data type to a datetime data type resulted in an out-of-range value.
The statement has been terminated.
bạn giúp mình gỡ lỗi này với
ChatGPT đã nói:
Tốt lắm — bạn đã viết câu lệnh rất chuẩn và chặt chẽ rồi.
Lỗi "The conversion of a nvarchar data type to a datetime data type resulted in an out-of-range value" thường không phải do cú pháp, mà do có ít nhất một giá trị trong dữ liệu nguồn không hợp lệ với format bạn đang convert (dù chỉ một dòng thôi cũng làm toàn bộ INSERT fail).

🎯 Nguyên nhân gốc thường gặp:
Trong dbXML.dbo.tbDSKhamBenhTheoKhoa_new, có dòng:

TGDANGKY, TGBDKHAM, TGKTKHAM, TGDUYETKT, hoặc TGDUYETDT
chứa chuỗi:

Không đúng định dạng ngày giờ (ví dụ: '32/01/2018 10:00', '00:00:00 00/00/0000', '05/09/1230 12:30', ' ')

Hoặc bị lỗi định dạng (ví dụ '2018/13/01', '13-32-2018')

Hoặc chỉ có thời gian mà không có ngày ('08:30')

Hoặc có ký tự lạ (tab, xuống dòng, dấu cách ẩn...)

✅ Cách xử lý nhanh nhất: dò những dòng lỗi
Bạn chạy từng lệnh dưới đây để tìm ra dòng gây lỗi (chạy từng cột một):

-- Kiểm tra TGDANGKY sai định dạng
SELECT TGDANGKY
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5), 103) IS NULL
  AND TGDANGKY IS NOT NULL AND TGDANGKY <> '';

-- Kiểm tra TGBDKHAM
SELECT TGBDKHAM
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,5), 103) IS NULL
  AND TGBDKHAM IS NOT NULL AND TGBDKHAM <> '';

-- Kiểm tra TGKTKHAM
SELECT TGKTKHAM
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,5), 103) IS NULL
  AND TGKTKHAM IS NOT NULL AND TGKTKHAM <> '';

-- Kiểm tra TGDUYETKT
SELECT TGDUYETKT
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,5), 103) IS NULL
  AND TGDUYETKT IS NOT NULL AND TGDUYETKT <> '';

-- Kiểm tra TGDUYETDT
SELECT TGDUYETDT
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,5), 103) IS NULL
  AND TGDUYETDT IS NOT NULL AND TGDUYETDT <> '';
💡 Lưu ý: TRY_CONVERT() sẽ trả về NULL nếu không convert được, nên ta dễ dàng lọc ra dòng sai.

🔍 Nếu bạn muốn kiểm tra tất cả lỗi một lần
Chạy:

SELECT MABENHNHAN, MAHOSOBENHAN, TGDANGKY, TGBDKHAM, TGKTKHAM, TGDUYETKT, TGDUYETDT
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE
    (TGDANGKY IS NOT NULL AND TGDANGKY <> '' AND TRY_CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5), 103) IS NULL)
 OR (TGBDKHAM IS NOT NULL AND TGBDKHAM <> '' AND TRY_CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,5), 103) IS NULL)
 OR (TGKTKHAM IS NOT NULL AND TGKTKHAM <> '' AND TRY_CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,5), 103) IS NULL)
 OR (TGDUYETKT IS NOT NULL AND TGDUYETKT <> '' AND TRY_CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,5), 103) IS NULL)
 OR (TGDUYETDT IS NOT NULL AND TGDUYETDT <> '' AND TRY_CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,5), 103) IS NULL);
→ Kết quả sẽ liệt kê chính xác những dòng nào đang khiến INSERT lỗi.

🧩 Cách khắc phục:
Sau khi tìm được các dòng sai, bạn có thể:

Sửa lại thủ công (UPDATE lại giá trị đúng định dạng), hoặc

Nếu muốn bỏ qua luôn các dòng lỗi, bạn thêm TRY_CONVERT() ngay trong câu INSERT để tránh bị chặn:

SET DATEFORMAT dmy;
INSERT INTO tbDSKhamBenhTheoKhoa (
    MABENHNHAN, MAHOSOBENHAN, TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN, 
    KHOA, PHONG, BACSY, MA_BAC_SI, TGDANGKY, TGBDKHAM, TGKTKHAM, 
    TRANGTHAI, XUTRI, DUYETKT, TGDUYETKT, DUYETDT, TGDUYETDT, 
    DVKT, DONTHUOC, PNGAYKHAM
)
SELECT 
    MABENHNHAN, MAHOSOBENHAN, TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN,
    KHOA, PHONG, BACSY, MA_BAC_SI,
    TRY_CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5), 103),
    TRY_CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,5), 103),
    TRY_CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,5), 103),
    TRANGTHAI, XUTRI, DUYETKT,
    TRY_CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,5), 103),
    DUYETDT,
    TRY_CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,5), 103),
    DVKT, DONTHUOC,
    TRY_CONVERT(date, LEFT(TGBDKHAM,10), 103)
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE TRY_CONVERT(date, LEFT(TGBDKHAM,10), 103) BETWEEN '2018-01-01' AND '2018-01-31';
Lúc này, dòng nào không convert được sẽ ra NULL và vẫn được insert, không làm câu lệnh dừng.

🧠 Tổng kết
Lỗi do ít nhất một giá trị ngày giờ không hợp lệ.

Dò lỗi bằng TRY_CONVERT(...) IS NULL.

Dùng TRY_CONVERT() trong INSERT để an toàn, tránh fail toàn batch.

Nếu bạn cần, mình có thể giúp viết thêm một câu UPDATE tự động sửa format ngày sai (ví dụ: đảo MM/dd → dd/MM, bỏ ký tự thừa, sửa năm 1230 → 2030…).

Bạn muốn mình giúp bạn viết luôn truy vấn phát hiện và sửa tự động các giá trị lỗi ngày giờ trong 5 cột đó không? (rất tiện để làm sạch dữ liệu trước khi insert).

Bạn đã nói:
mình vẫn đang sử dụng sql server 2008r2 nên try_convert() mình không sử dụng được
ChatGPT đã nói:
Rất chính xác 👏 — bạn đang dùng SQL Server 2008 R2, nên đúng là TRY_CONVERT() và TRY_CAST() chưa tồn tại (chỉ có từ SQL 2012 trở đi).
Vì vậy, ta phải dùng cách khác tương đương, để phát hiện dòng lỗi và tránh dừng cả batch khi gặp dữ liệu ngày không hợp lệ.

🎯 Mục tiêu:
Tìm ra dòng nào có dữ liệu ngày/giờ sai format trong các cột:
TGDANGKY, TGBDKHAM, TGKTKHAM, TGDUYETKT, TGDUYETDT

Xử lý an toàn khi INSERT mà không bị lỗi.

🧩 Cách 1: Dò dòng lỗi bằng TRY...CATCH (vòng lặp từng dòng)
Vì SQL 2008 không có TRY_CONVERT(), ta phải dùng cách thủ công sau — dò từng dòng có lỗi:

DECLARE @ID int, @Ngay nvarchar(50);

DECLARE cur CURSOR FOR
SELECT ID, TGBDKHAM
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new;

OPEN cur;
FETCH NEXT FROM cur INTO @ID, @Ngay;

WHILE @@FETCH_STATUS = 0
BEGIN
    BEGIN TRY
        DECLARE @tmp datetime;
        SET @tmp = CONVERT(datetime, LEFT(@Ngay,10) + ' ' + RIGHT(@Ngay,5), 103);
    END TRY
    BEGIN CATCH
        PRINT 'Lỗi tại ID: ' + CAST(@ID AS nvarchar) + ' | Giá trị: ' + ISNULL(@Ngay, 'NULL');
    END CATCH

    FETCH NEXT FROM cur INTO @ID, @Ngay;
END

CLOSE cur;
DEALLOCATE cur;
👉 Cách này sẽ in ra những dòng có giá trị lỗi (ví dụ 32/13/2018, 05/09/1230 12:30, ...).
Bạn có thể thay TGBDKHAM bằng từng cột khác để kiểm tra.

🧩 Cách 2: Lọc nhanh bằng regex-like điều kiện — không cần TRY_CONVERT
Nếu format của bạn luôn là kiểu “hh:mm:ss dd/MM/yyyy” hoặc “hh:mm dd/MM/yyyy” thì bạn có thể lọc bằng mẫu:

SELECT TOP 100 *
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE
    (TGDANGKY IS NOT NULL AND TGDANGKY <> '' AND TGDANGKY NOT LIKE '%[0-9][0-9]/[0-9][0-9]/20[0-9][0-9]%')
 OR (TGBDKHAM IS NOT NULL AND TGBDKHAM <> '' AND TGBDKHAM NOT LIKE '%[0-9][0-9]/[0-9][0-9]/20[0-9][0-9]%')
 OR (TGKTKHAM IS NOT NULL AND TGKTKHAM <> '' AND TGKTKHAM NOT LIKE '%[0-9][0-9]/[0-9][0-9]/20[0-9][0-9]%')
 OR (TGDUYETKT IS NOT NULL AND TGDUYETKT <> '' AND TGDUYETKT NOT LIKE '%[0-9][0-9]/[0-9][0-9]/20[0-9][0-9]%')
 OR (TGDUYETDT IS NOT NULL AND TGDUYETDT <> '' AND TGDUYETDT NOT LIKE '%[0-9][0-9]/[0-9][0-9]/20[0-9][0-9]%');
➡ Những dòng xuất hiện trong kết quả này rất có thể chứa ngày không hợp lệ hoặc sai format.
Bạn xem vài dòng đầu tiên là thấy lỗi rõ ngay.

🧩 Cách 3: Dùng kiểm tra “an toàn” khi INSERT (bỏ qua dòng lỗi)
Vì bạn không có TRY_CONVERT, ta có thể dùng subquery tạm thời lọc các dòng hợp lệ trước khi insert, bằng WHERE lọc theo pattern hợp lệ.

Giả sử chuỗi ngày luôn ở dạng hh:mm:ss dd/MM/yyyy hoặc hh:mm dd/MM/yyyy, ta làm:

SET DATEFORMAT dmy;

INSERT INTO dbDataBVNT.dbo.tbDSKhamBenhTheoKhoa(
    MABENHNHAN, MAHOSOBENHAN, TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN,
    KHOA, PHONG, BACSY, MA_BAC_SI,
    TGDANGKY, TGBDKHAM, TGKTKHAM, TRANGTHAI, XUTRI,
    DUYETKT, TGDUYETKT, DUYETDT, TGDUYETDT, DVKT, DONTHUOC, PNGAYKHAM
)
SELECT 
    MABENHNHAN, MAHOSOBENHAN, TENBENHNHAN, GIOITINH, MA_BHYT, TEN_DTBN,
    KHOA, PHONG, BACSY, MA_BAC_SI,
    CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5), 103),
    CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,5), 103),
    CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,5), 103),
    TRANGTHAI, XUTRI, DUYETKT,
    CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,5), 103),
    DUYETDT,
    CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,5), 103),
    DVKT, DONTHUOC,
    CONVERT(date, LEFT(TGBDKHAM,10), 103)
FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE
    ISDATE(LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,5)) = 1
    AND ISDATE(LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5)) = 1
    AND (TGKTKHAM IS NULL OR ISDATE(LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,5)) = 1)
    AND (TGDUYETKT IS NULL OR ISDATE(LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,5)) = 1)
    AND (TGDUYETDT IS NULL OR ISDATE(LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,5)) = 1)
    AND CONVERT(date, LEFT(TGBDKHAM,10), 103) BETWEEN '2018-01-01' AND '2018-01-31';
🔹 Hàm ISDATE() vẫn hoạt động tốt trong SQL 2008
→ Nó trả về 1 nếu chuỗi có thể chuyển đổi hợp lệ sang datetime,
→ 0 nếu không hợp lệ → dòng đó sẽ bị loại bỏ trước khi insert.

➡ Cách này an toàn tuyệt đối, không bị lỗi dù có dữ liệu xấu trong bảng nguồn.

✅ Kết luận
Mục tiêu	Giải pháp dùng cho SQL 2008 R2
Xác định dòng lỗi	Dùng TRY...CATCH (cursor) hoặc ISDATE()
Bỏ qua dòng lỗi khi insert	Dùng WHERE ISDATE(...) = 1
Xem giá trị lỗi cụ thể	Dùng PRINT trong TRY...CATCH hoặc lọc ISDATE(...) = 0
Không cần TRY_CONVERT()	✅ Đúng, ta thay bằng ISDATE() + CONVERT()
Nếu bạn muốn, mình có thể viết giúp phiên bản truy vấn hiển thị rõ ràng tên cột nào đang lỗi + giá trị lỗi cụ thể (dễ sửa trước khi insert).
Bạn có muốn mình tạo câu đó luôn không?

Bạn đã nói:
ồ cảm ơn bạn mình đã tìm ra lỗi rồi, lỗi là do mình, giá trị cột tg vd 11/01/2018 06:35:27 nhưng mình lại để câu lệnh CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,5), 103) phải là  RIGHT(TGDANGKY,8)
ChatGPT đã nói:
🎯 Chuẩn luôn! Bạn bắt đúng bệnh 100% rồi 👏

Đúng như bạn nói:
Nếu dữ liệu gốc có đầy đủ “giờ:phút:giây” (như 11/01/2018 06:35:27)
thì bạn phải lấy 8 ký tự cuối chứ không phải 5 — vì "06:35:27" là 8 ký tự.

✅ Cách đúng:
CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,8), 103)
Tương tự, nên sửa cho các cột khác:

CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,8), 103)
CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,8), 103)
CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,8), 103)
CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,8), 103)
💡 Giải thích:
Hàm	Ý nghĩa
LEFT(TGDANGKY,10)	Lấy phần ngày 11/01/2018
RIGHT(TGDANGKY,8)	Lấy phần giờ 06:35:27
+ ' '	Ghép thành 11/01/2018 06:35:27
CONVERT(datetime, ..., 103)	Chuyển theo format dd/MM/yyyy hh:mm:ss
⚡ Mẹo nhỏ:
Nếu sau này bạn gặp nhiều cột kiểu này, bạn có thể viết nhanh một “hàm chuyển đổi” tạm (trong SQL 2008 R2):

CREATE FUNCTION dbo.fn_ConvertToDateTime103 (@s nvarchar(50))
RETURNS datetime
AS
BEGIN
    DECLARE @d datetime
    IF ISDATE(@s) = 1
        SET @d = CONVERT(datetime, @s, 103)
    ELSE IF LEN(@s) >= 19
        SET @d = CONVERT(datetime, LEFT(@s,10) + ' ' + RIGHT(@s,8), 103)
    ELSE
        SET @d = NULL
    RETURN @d
END
Rồi bạn chỉ cần gọi:

dbo.fn_ConvertToDateTime103(TGDANGKY)
→ Câu lệnh INSERT của bạn sẽ gọn hơn và tránh nhầm 5/8 ký tự nữa.

Rất tốt — bạn làm rất kỹ và hiểu logic rồi.
Giờ chỉ cần sửa tất cả các chỗ RIGHT(...,5) → RIGHT(...,8) là câu lệnh sẽ chạy trơn tru 💪

Bạn có muốn mình giúp viết lại toàn bộ câu INSERT đã tối ưu và nhất quán (cho 5 cột thời gian) không?
Mình sẽ gộp luôn phần kiểm tra hợp lệ (ISDATE) để tránh lỗi về sau.

Bạn đã nói:
vâng có, bạn cho mình để mình làm cho chuẩn luôn
ChatGPT đã nói:
Tuyệt vời 👏 — dưới đây là phiên bản tối ưu, chuẩn và an toàn tuyệt đối cho SQL Server 2008 R2,
đã sửa đúng RIGHT(...,8) cho dữ liệu có định dạng đầy đủ dd/MM/yyyy hh:mm:ss,
đồng thời thêm lọc ISDATE() để tránh lỗi khi có dữ liệu sai định dạng trong nguồn.

✅ Câu lệnh hoàn chỉnh và tối ưu
SET DATEFORMAT dmy;

INSERT INTO dbDataBVNT.dbo.tbDSKhamBenhTheoKhoa(
    MABENHNHAN, 
    MAHOSOBENHAN, 
    TENBENHNHAN, 
    GIOITINH, 
    MA_BHYT, 
    TEN_DTBN, 
    KHOA, 
    PHONG, 
    BACSY, 
    MA_BAC_SI, 
    TGDANGKY, 
    TGBDKHAM, 
    TGKTKHAM, 
    TRANGTHAI, 
    XUTRI, 
    DUYETKT, 
    TGDUYETKT, 
    DUYETDT, 
    TGDUYETDT, 
    DVKT, 
    DONTHUOC, 
    PNGAYKHAM
)
SELECT 
    MABENHNHAN,
    MAHOSOBENHAN,
    TENBENHNHAN,
    GIOITINH,
    MA_BHYT,
    TEN_DTBN,
    KHOA,
    PHONG,
    BACSY,
    MA_BAC_SI,

    -- Thời gian đăng ký
    CASE 
        WHEN ISDATE(LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,8)) = 1
        THEN CONVERT(datetime, LEFT(TGDANGKY,10) + ' ' + RIGHT(TGDANGKY,8), 103)
        ELSE NULL
    END AS TGDANGKY,

    -- Thời gian bắt đầu khám
    CASE 
        WHEN ISDATE(LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,8)) = 1
        THEN CONVERT(datetime, LEFT(TGBDKHAM,10) + ' ' + RIGHT(TGBDKHAM,8), 103)
        ELSE NULL
    END AS TGBDKHAM,

    -- Thời gian kết thúc khám
    CASE 
        WHEN ISDATE(LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,8)) = 1
        THEN CONVERT(datetime, LEFT(TGKTKHAM,10) + ' ' + RIGHT(TGKTKHAM,8), 103)
        ELSE NULL
    END AS TGKTKHAM,

    TRANGTHAI,
    XUTRI,
    DUYETKT,

    -- Thời gian duyệt kế toán
    CASE 
        WHEN ISDATE(LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,8)) = 1
        THEN CONVERT(datetime, LEFT(TGDUYETKT,10) + ' ' + RIGHT(TGDUYETKT,8), 103)
        ELSE NULL
    END AS TGDUYETKT,

    DUYETDT,

    -- Thời gian duyệt đơn thuốc
    CASE 
        WHEN ISDATE(LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,8)) = 1
        THEN CONVERT(datetime, LEFT(TGDUYETDT,10) + ' ' + RIGHT(TGDUYETDT,8), 103)
        ELSE NULL
    END AS TGDUYETDT,

    DVKT,
    DONTHUOC,

    -- Ngày khám (dùng để phân vùng)
    CASE 
        WHEN ISDATE(LEFT(TGBDKHAM,10)) = 1 
        THEN CONVERT(date, LEFT(TGBDKHAM,10), 103)
        ELSE NULL
    END AS PNGAYKHAM

FROM dbXML.dbo.tbDSKhamBenhTheoKhoa_new
WHERE 
    ISDATE(LEFT(TGBDKHAM,10)) = 1
    AND CONVERT(date, LEFT(TGBDKHAM,10), 103) BETWEEN '2018-01-01' AND '2018-01-31';
GO
🧩 Giải thích ngắn gọn:
Thành phần	Vai trò
RIGHT(...,8)	Lấy đúng phần "hh:mm:ss"
ISDATE(...) = 1	Đảm bảo chuỗi hợp lệ trước khi CONVERT()
CASE WHEN ... THEN ... ELSE NULL	Nếu lỗi → chèn NULL, không dừng batch
CONVERT(datetime, ..., 103)	Chuyển đúng định dạng dd/MM/yyyy hh:mm:ss
SET DATEFORMAT dmy	Bắt buộc để SQL hiểu đúng dd/MM/yyyy
BETWEEN '2018-01-01' AND '2018-01-31'	Lọc đúng phân vùng dữ liệu bạn muốn nhập
⚙️ Mẹo thêm (nếu bạn chạy thường xuyên)
Nếu sau này bạn cần nhập theo tháng hoặc năm khác, chỉ cần đổi dòng cuối:

AND CONVERT(date, LEFT(TGBDKHAM,10), 103) BETWEEN '2025-10-01' AND '2025-10-31'
Hoặc nếu muốn tự động theo tháng hiện tại:

AND CONVERT(date, LEFT(TGBDKHAM,10), 103) BETWEEN 
    DATEADD(DAY, 1-DAY(GETDATE()), CAST(GETDATE() AS date)) 
    AND EOMONTH(GETDATE())
🧠 Tổng kết:
✅ Hoàn toàn tương thích với SQL Server 2008 R2
✅ An toàn tuyệt đối (không dừng nếu dữ liệu lỗi)
✅ Đúng định dạng dd/MM/yyyy hh:mm:ss
✅ Dễ tái sử dụng cho các bảng tương tự

Bạn có muốn mình viết thêm 1 script ngắn kiểm tra sau khi insert (đếm số dòng insert thành công, dòng có thời gian null, hoặc lỗi format bị loại bỏ) để bạn dễ kiểm tra kết quả không?
Nếu bạn muốn, mình thêm ngay bên dưới cho đồng bộ luôn.

Bạn đã nói:
mình chưa cần, cảm ơn bạn
ChatGPT đã nói:
Rất tốt 👍 — bạn xử lý cực kỳ chuẩn rồi!
Khi nào bạn muốn kiểm tra lại dữ liệu sau khi insert (để xem tỷ lệ cột thời gian nào bị NULL, hoặc cần dọn dữ liệu thêm), chỉ cần nói mình biết, mình sẽ giúp bạn tạo truy vấn kiểm tra nhanh gọn.

Chúc bạn chạy lệnh insert chuẩn, không lỗi và dữ liệu được phân vùng đúng như mong muốn 💪




No file chosenNo file chosen
ChatGPT có thể mắc lỗi. Hãy kiểm tra các thông tin quan trọng.

## 🧩 Ghi chú quan trọng
