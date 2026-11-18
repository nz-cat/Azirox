# Azirox Macros

Vài cái cần nhớ trước khi gắn:

- Gõ thẳng tên macro ra đừng gán nó vào biến rồi gọi vòng vòng (`local f = AZR_CRASH` rồi `f()` là toang ngay)
- Chạy thẳng file gốc chưa build thì mấy macro này coi như vô hình - trừ `AZR_OBFUSCATED` sinh ra chính là để phân biệt 2 cảnh đó
- 2 dạng chính:
  - **Gọi được** - có `()` phía sau nhét đối số vào thoải mái kiểu `AZR_CRASH()`
  - **Giá trị** - khỏi cần `()` xài y như 1 biến có sẵn trên trời rơi xuống kiểu `AZR_LINE`

---

## Danh sách macro

| Macro | Dạng | Để làm gì |
|---|---|---|
| [`AZR_CRASH()`](#azr_crash) | Gọi được | Tự tay bóp script cho ngủm |
| [`AZR_OBFUSCATED`](#azr_obfuscated) | Giá trị | Biết đang ở bản build hay bản mộc |
| [`AZR_LINE`](#azr_line) | Giá trị | Số dòng hiện tại debug cho đỡ mù |
| [`AZR_ENCSTR(str)`](#azr_encstr) | Gọi được | Giấu chuỗi chữ khỏi ai hóng |
| [`AZR_ENCNUM(n)`](#azr_encnum) | Gọi được | Giấu số khỏi ai hóng |
| [`AZR_OPAQUE(bool)`](#azr_opaque) | Gọi được | Làm if trông ảo khó đoán nhánh thật |
| [`AZR_PRECHECK(fn, key)`](#azr_precheck) | Gọi được | Tự kiểm tra sai phát là châm ngòi crash |
| [`AZR_COMPRESS(str)`](#azr_compress) | Gọi được | Nén + giấu 1 cục chuỗi dài |
| [`AZR_CFF(fn)`](#azr_cff) | Gọi được | Băm nhỏ thân hàm thành 1 state machine rối não |
| [`AZR_NO_VIRTUALIZE(fn)`](#AZR_NO_VIRTUALIZEfn) | Gọi được | Rút hàm khỏi VM, cho chạy native cho nhanh |

---

## `AZR_CRASH()`

Tự tay làm script lăn ra chết không kèn không trống

Khác `error()` bình thường ở chỗ: lỡ có đứa nào bọc code m trong `pcall` định nuốt lỗi cho chạy tiếp như chưa có gì xảy ra thì `AZR_CRASH()` không cho cơ hội đó - chết là chết hẳn

Hợp cho mấy chỗ phát hiện có mùi bị dòm ngó debug hay chọt ngoáy muốn dẹp luôn tại chỗ

**Cú pháp:**

```lua
AZR_CRASH()
```

**Demo:**

```lua
local function kiemtraIntegrity()
    local biCanThiep = false
    if biCanThiep then
        AZR_CRASH()
    end
end

kiemtraIntegrity()
print'chay toi day la an toan'
```

**Lưu ý:** Gọi bao nhiêu lần cũng được đứa nào gọi đứa nấy tự chết không đụng gì nhau

---

## `AZR_OBFUSCATED`

Cho biết đoạn code đang chạy là hàng đã build hay còn mộc

Dùng để tách 2 kiểu hành xử - 1 bản xịn gửi khách 1 bản xuề xoà tự test ở nhà

**Cú pháp:**

```lua
AZR_OBFUSCATED
```

(không `()` gì hết cứ coi nó như biến có sẵn)

**Demo:**

```lua
local function taiConfig()
    if AZR_OBFUSCATED then
        return layConfigTuServer()
    else
        return { debug = true, endpoint = "http://localhost:3000" }
    end
end
```

Build qua Azirox thì nhánh `layConfigTuServer()` chạy

Mở thẳng file gốc chạy test thì nó tự rớt vào nhánh `else` khỏi cần sửa code qua lại cho mệt

**Lưu ý:** Đừng dại gọi `AZR_OBFUSCATED()` - nó là giá trị không phải hàm

---

## `AZR_LINE`

Trả số dòng ngay tại chỗ m đặt nó xuống

Debug mà không biết lỗi nằm dòng nào thì khổ - cái này cứu m khỏi cảnh ngồi đếm dòng bằng mắt

**Cú pháp:**

```lua
AZR_LINE
```

**Demo:**

```lua
print("bat dau tu dong", AZR_LINE)

local function f()
    error("no chet tai dong " .. AZR_LINE)
end
```

**Lưu ý:** Số dòng tính theo file gốc m gõ cố định luôn không đổi dù hàm bị gọi lại bao nhiêu lần từ đâu - thuần debug thôi đừng mong nó bảo vệ được gì

---

## `AZR_ENCSTR(str)`

Giấu 1 cục chuỗi - URL key lời chửi lỗi tên API gì cũng được - để lúc build xong ai mở file lên soi chữ chỉ thấy rác không đọc được chuỗi gốc

**Cú pháp:**

```lua
AZR_ENCSTR("chuoi cua ban")
```

**Demo:**

```lua
local url = AZR_ENCSTR("https://example.com/api")
print(url) -- luc chay no tu giai ma in dung y chang chuoi goc
```

**Lưu ý:**
- Chuỗi phải viết thẳng tay ngay tại chỗ gọi đừng truyền biến hay chuỗi ghép sẵn vô (`AZR_ENCSTR(myVar)` là ăn lỗi ngay lúc build)
- Giá trị trả về xài y như chuỗi thường không cần làm gì thêm nó tự lo khâu giải mã

---

## `AZR_ENCNUM(n)`

Anh em sinh đôi với `AZR_ENCSTR` chỉ khác là giấu số - ID mã số flag số nào nhạy cảm thì nhét vô đây khỏi để trần truồng trong file build

**Cú pháp:**

```lua
AZR_ENCNUM(123)
AZR_ENCNUM(-9999)
AZR_ENCNUM(0x1A2B)
```

**Demo:**

```lua
local soLanThuLai = AZR_ENCNUM(5)
```

**Lưu ý:**
- Chỉ nhận số nguyên viết thẳng - không biến không số thực kiểu `3.14`
- Số to quá khổ (ngoài tầm ±2 tỷ) thì chịu báo lỗi thẳng lúc build luôn

---

## `AZR_OPAQUE(true | false)`

Biến 1 điều kiện `if` trông như phải vắt óc tính toán mới ra đúng sai mà thiệt ra kết quả đã cố định sẵn đúng như m đưa vào

Dùng để che nhánh code thật khỏi để ai lướt qua là đoán ngay chắc if này luôn đúng rồi

**Cú pháp:**

```lua
AZR_OPAQUE(true)
AZR_OPAQUE(false)
```

**Demo:**

```lua
if AZR_OPAQUE(true) then
    lamThatSu()
else
    lamMoiNhuThat() -- nhanh mon khong bao gio toi luot chay
end
```

Chạy thì y chang `if true then` / `if false then` chỉ khác người ngồi đọc code đã build chịu không nhìn ra liền được

**Lưu ý:** Đối số chỉ nhận `true`/`false` trần - biến hay biểu thức gì khác đều không chơi

---

## `AZR_PRECHECK(fn, expectedKey)`

Tự chạy 1 hàm kiểm tra so kết quả với giá trị m kỳ vọng sẵn - trật phát nào là châm ngòi `AZR_CRASH()` liền không hỏi han gì thêm

Dùng để tự soi xem 1 điều kiện nào đó có đúng y boong như mong đợi trước khi cho phần code xịn chạy tiếp hay không

**Cú pháp:**

```lua
AZR_PRECHECK(function()
    return <gia tri can kiem tra>
end, <gia tri dung>)
```

**Demo - so 1 giá trị:**

```lua
AZR_PRECHECK(function()
    return kiemtraValue()
end, 0x12345678)
```

**Demo - so nhiều giá trị 1 lượt gói chung table:**

```lua
AZR_PRECHECK(function()
    return { layValueA(), layValueB() }
end, { 0x1234, 0x5678 })
```

**Lưu ý:**
- Viết nó thành 1 câu lệnh đứng riêng đừng ráng gán vô biến - nó không nhả giá trị gì ra để xài tiếp
- Hàm kiểm tra (`fn`) nên là hàm vô danh viết liền tại chỗ đừng dựa dẫm thứ bên ngoài ngoài những gì thật sự cần để tính ra kết quả

---

## `AZR_COMPRESS(str)`

Giống `AZR_ENCSTR` ở chỗ cũng giấu chuỗi, nhưng nén qua LZSS trước khi mã hoá - hợp cho mấy cục chuỗi dài (JSON cấu hình nhúng sẵn, danh sách dài, 1 đoạn Lua source nhúng làm string...) chứ chuỗi ngắn thì xài `AZR_ENCSTR` cho lẹ, nén chuỗi ngắn có khi còn phình to hơn gốc do overhead của flag-byte.

**Cú pháp:**

```lua
AZR_COMPRESS("chuoi cua ban")
```

**Demo:**

```lua
local config = AZR_COMPRESS("{\"endpoint\":\"https://api.example.com\",\"retries\":5,\"timeout\":3000}")
print(config) -- tu giai nen + giai ma, in ra dung y chang chuoi goc
```

**Lưu ý:**
- Chuỗi phải viết thẳng tay ngay tại chỗ gọi, y chang `AZR_ENCSTR` - không truyền biến
- Hỗ trợ full 256 giá trị byte (UTF-8, binary gì cũng được), không riêng gì ASCII

---

## `AZR_CFF(fn)`

Băm thân 1 hàm thành dispatcher-loop (`while true do if state==... end`) + opaque predicate + vài nhánh rác chen vào - đọc code đã build thì mất luôn trình tự thực thi tuần tự ban đầu, phải tự dò từng state một

Dùng cho những hàm logic nhạy cảm mà m muốn giấu luồng chạy riêng, không cần đợi build nguyên file mới có CFF (control-flow flatten vốn Azirox đã làm ở bước Minifier cho toàn bộ code, cái này cho phép áp riêng 1 hàm ngay từ source)

**Cú pháp:**

```lua
AZR_CFF(function(...)
    -- than ham binh thuong
end)
```

**Demo:**

```lua
local tinhDiem = AZR_CFF(function(hp, armor)
    local dmg = hp - armor
    if dmg < 0 then
        dmg = 0
    end
    return dmg * 2
end)

print(tinhDiem(100, 30))
```

Macro trả về nguyên 1 hàm (đã băm), gán vào biến/gọi liền thoải mái như hàm bình thường

**Lưu ý:**
- Đối số chỉ nhận `function(...) ... end` viết thẳng tại chỗ, không nhận biến đã gán sẵn hàm
- Hàm càng ít statement (dưới 2 câu lệnh) thì khỏi băm, băm 1-2 dòng chẳng ăn thua gì

---

## `AZR_NO_VIRTUALIZE(fn)`

Rút hẳn hàm m đưa vào ra khỏi VM của Azirox - hàm không bị dịch thành bytecode VM nữa mà được giấu dưới dạng mảng byte, tới lúc chạy mới tự giải mã rồi đưa thẳng qua `loadstring` của chính môi trường (executor) để biên dịch native. Từ đó hàm chạy nhanh hơn hẳn vì không còn tốn chi phí dispatch từng opcode qua VM phần mềm nữa - hợp cho vòng lặp render, tính toán nặng... chỗ nào cần tốc độ hơn là cần bưng kín

**Cú pháp:**

```lua
local f = AZR_NO_VIRTUALIZE(function(...) end)
AZR_NO_VIRTUALIZE(function(...) end)()
someFunction(AZR_NO_VIRTUALIZE(function(...) end))
```

**Demo:**

```lua
local tinhTong = AZR_NO_VIRTUALIZE(function(n)
    local s = 0
    for i = 1, n do
        s = s + i * i
    end
    return s
end)

print(tinhTong(1000000)) -- chay native, khong qua VM nen nhanh hon han
```

**Lưu ý:**
- Đối số chỉ nhận `function(...) ... end` viết thẳng tại chỗ, giống `AZR_CFF` - không nhận biến đã gán sẵn hàm
- Hàm **không được** đụng tới local/upvalue nằm ngoài nó - vì hàm được biên dịch độc lập qua vm khác nên không thấy local bên ngoài, chỉ thấy global và tham số truyền vào qua `(...)`. Đụng vào local ngoài là lỗi `attempt to ... nil value` thẳng lúc chạy, không phải lỗi build
- Không dùng cho chỗ cần bảo mật cao - hàm thoát khỏi VM nghĩa là thoát luôn mọi lớp bảo vệ Azirox áp cho phần còn lại của script, chỉ còn 1 lớp XOR mỏng che source thô
