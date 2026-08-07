# VN Screener 11 Metrics — AFL cho AmiBroker

File: [`VN_Screener_11_Metrics.afl`](VN_Screener_11_Metrics.afl)

Bộ lọc (Exploration) quét toàn bộ danh mục mã và chấm 11 tiêu chí:

| # | Tiêu chí | Cách tính trong code |
|---|---|---|
| 1 | P / Tangible Book Value Per Share | `Close / ((Equity − Intangible) / SharesOut)` |
| 2 | (Cash + Đầu tư TC dài hạn) / Tổng tài sản | `100 * (CASH + LTINV) / ASSET` |
| 3 | EV / EBITDA | `EV = MarketCap + Debt + Minority − Cash`; EBITDA lấy TTM |
| 4 | Sở hữu nước ngoài (%) | đọc từ nguồn dữ liệu, tự quy đổi nếu nguồn trả 0..1 |
| 5 | SMA(100) < giá hiện tại | `MA(C,100) < C` |
| 6 | SMA(20) > SMA(100) | `MA(C,20) > MA(C,100)` |
| 7 | SMA(5) > SMA(20) | `MA(C,5) > MA(C,20)` |
| 8 | SMA(20) > SMA(100) | trùng #6 — xem ghi chú bên dưới |
| 9 | Khối ngoại mua ròng 1 tuần | `Sum(NetValue, 5) > 0` |
| 10 | Khối ngoại mua ròng 1 tháng | `Sum(NetValue, 21) > 0` |
| 11 | Khối ngoại mua ròng 1 quý | `Sum(NetValue, 63) > 0` |

> **Ghi chú tiêu chí #8:** trong yêu cầu gốc, #6 và #8 giống hệt nhau (SMA20 > SMA100).
> Code vẫn giữ đủ 11 cột đúng như đề bài, nhưng #8 có 2 tham số riêng
> (`TC8: SMA nhanh/chậm`) nên nếu ý bạn là một cặp khác — ví dụ SMA(20) > SMA(50) —
> chỉ cần đổi Param, không phải sửa code.

---

## 1. Cách chạy

1. `Analysis → Formula Editor` → mở/paste file `VN_Screener_11_Metrics.afl`.
2. `Analysis → New Analysis`:
   - **Apply to:** All symbols (hoặc Watchlist HOSE/HNX/UPCOM của bạn)
   - **Range:** `1 recent bar` (bộ lọc đã tự khóa `Status("lastbarinrange")`)
3. Bấm **Explore**.
4. Bấm nút **Parameters** để chỉnh ngưỡng: `P/TangBV <= 2.0`, `(Cash+Inv)/Asset >= 20%`,
   `EV/EBITDA <= 10`, `Sở hữu NN >= 5%`, số phiên cho tuần/tháng/quý, lọc thanh khoản…

Kết quả: mỗi ô tiêu chí **xanh = đạt**, **hồng = không đạt**, cột `Điểm /11` cho biết
mã đạt bao nhiêu tiêu chí và bảng được sắp xếp giảm dần theo điểm này.

Bật `Hiển thị TẤT CẢ mã = Có` để xem toàn bộ danh sách kèm tô màu (chế độ soi dữ liệu),
tắt đi để chỉ giữ những mã đạt **đủ** các tiêu chí đang bật.

Cột **Dữ liệu** cuối bảng báo `Thiếu FA` / `Thiếu NN` — để phân biệt "mã bị loại vì
cơ bản xấu" với "mã bị loại vì bạn chưa nạp số liệu".

---

## 2. Nạp dữ liệu (bắt buộc)

AmiBroker **không** có sẵn báo cáo tài chính và giao dịch khối ngoại của TTCK Việt Nam.
Bạn phải nạp trước rồi trỏ đúng nguồn ở phần CONFIG đầu file. File hỗ trợ các kiểu sau.

### 2.1 Dữ liệu cơ bản — Chế độ 0 (mặc định): mỗi chỉ tiêu một composite

Đặt `[A] Nguồn FA = 0`. Code đọc `Foreign("~FA_<FIELD>_<TICKER>", "C")`.

Tạo/nhập các mã dạng `~FA_EQUITY_VNM`, `~FA_ASSET_VNM`, … với **giá trị nằm ở cột Close**,
ngày = ngày công bố BCTC (giá trị được pad sang các phiên sau).

| FIELD | Nội dung | Đơn vị gợi ý |
|---|---|---|
| `EQUITY` | Vốn chủ sở hữu | VND |
| `ASSET` | Tổng tài sản | VND |
| `CASH` | Tiền và tương đương tiền (+ đầu tư ngắn hạn) | VND |
| `LTINV` | Đầu tư tài chính dài hạn | VND |
| `SHARES` | Số CP đang lưu hành | cổ phiếu |
| `DEBT` | Tổng nợ vay ngắn + dài hạn | VND |
| `EBITDA` | EBITDA (từng quý hoặc TTM) | VND |
| `INTANGIBLE` | TSCĐ vô hình + lợi thế thương mại | VND |
| `MINORITY` | Lợi ích cổ đông thiểu số (tùy chọn) | VND |
| `FOWN` | Tỷ lệ sở hữu NN (%) — nếu không dùng `~FO_` | % hoặc 0..1 |

Giá và vốn hóa phải cùng đơn vị: nếu giá trong AmiBroker là VND (25000) thì
`SHARES` là số cổ phiếu tuyệt đối → `MarketCap` ra VND, khớp với `EQUITY`/`ASSET` VND.

Nếu nguồn của bạn cấp **EBITDA từng quý**, để `EBITDA nguồn là số TỪNG QUÝ = Có`
(mặc định) — code tự cộng 4 quý gần nhất thành TTM bằng `ValueWhen` trên các điểm
đổi giá trị. Nếu nguồn đã là TTM thì tắt toggle này (nếu để bật với dữ liệu TTM,
`EV/EBITDA` sẽ ra rỗng vì không đủ 4 điểm đổi kỳ).

**Định dạng file CSV để import (ASCII Importer):**

```
$FORMAT Ticker, Date_YMD, Close
$SKIPLINES 1
$SEPARATOR ,
$CONT 1
$GROUP 253
$AUTOADD 1
```

```csv
Ticker,Date,Close
~FA_EQUITY_VNM,20260331,35120000000000
~FA_ASSET_VNM,20260331,55480000000000
~FA_SHARES_VNM,20260331,2089955445
```

### 2.2 Dữ liệu cơ bản — Chế độ 1: gộp tất cả vào một composite/mã

Đặt `[A] Nguồn FA = 1`. Code đọc `~FA_<TICKER>` và map các trường vào từng cột:

| Cột | Trường |
|---|---|
| Close | Equity |
| Open | Total Asset |
| High | Cash |
| Low | Long-term Investment |
| Volume | Shares Outstanding |
| OpenInt | Total Debt |
| Aux1 | EBITDA |
| Aux2 | Intangible |

Chế độ này gọn hơn (1 mã/1 cổ phiếu) nhưng không có ô cho `MINORITY` (code lấy = 0).

Chế độ 1 cũng **nhanh hơn hẳn**: code dùng một lần `SetForeign()` để lấy cả 8 trường,
thay vì 8 lần gọi `Foreign()` như chế độ 0. Nếu AmiBroker báo `Warning 512 — Calling
Foreign() too many times`, đó là do chế độ 0 và chỉ là **cảnh báo hiệu năng, không phải
lỗi** — muốn hết cảnh báo thì chuyển sang chế độ 1.

```
$FORMAT Ticker, Date_YMD, Open, High, Low, Close, Volume, OpenInt, Aux1, Aux2
```

### 2.3 Giao dịch khối ngoại

`[B] Nguồn khối ngoại`:

- **0 (mặc định)** — nạp thẳng lên mã cổ phiếu: `Aux1 = KL mua NN`, `Aux2 = KL bán NN`.
  Giá trị ròng mỗi phiên = `(Aux1 − Aux2) * Close`.
- **1** — `Aux1`/`Aux2` là **giá trị** mua/bán (VND), không nhân giá.
- **2** — composite `~FN_<TICKER>`, cột Close = **giá trị mua ròng của phiên**
  (dương = mua ròng, âm = bán ròng).

Import kèm dữ liệu giá (chế độ 0/1):

```
$FORMAT Ticker, Date_YMD, Open, High, Low, Close, Volume, Aux1, Aux2
```

**Sở hữu nước ngoài (#4):** composite `~FO_<TICKER>`, Close = % sở hữu NN.
Nếu không có, ở chế độ FA 0 code tự fallback sang `~FA_FOWN_<TICKER>`.
Nguồn trả 0..1 hay 0..100 đều được — code tự nhận diện và quy về %.

Đổi tiền tố `~FA_` / `~FN_` / `~FO_` bằng Param nếu bạn đang dùng quy ước tên khác.

---

## 3. Tham số mặc định

| Tham số | Mặc định |
|---|---|
| P/TangibleBV ≤ | 2.0 |
| (Cash+LTInv)/Asset ≥ | 20% |
| EV/EBITDA ≤ | 10 |
| Sở hữu NN ≥ | 5% |
| Số phiên tuần / tháng / quý | 5 / 21 / 63 |
| Giá tối thiểu | 1.000 đ |
| GTGD TB20 tối thiểu | 1.000 triệu (1 tỷ) |
| Đơn vị hiển thị giá trị ròng | 1e9 (tỷ đồng) |

Mỗi tiêu chí có toggle bật/tắt riêng — tắt tiêu chí nào thì tiêu chí đó không loại mã
nữa nhưng cột vẫn hiển thị và vẫn tính vào `Điểm /11`.

---

## 4. Lưu ý kỹ thuật

- Các phép chia đều qua `SafeDiv()`, hàm này thay mẫu bằng 1 trước khi chia (vì `IIf`
  tính **cả hai** nhánh — để nguyên `num/den` sẽ sinh `Warning 505 Division by zero`);
  mẫu = 0 hoặc Null trả về Null, ô hiển thị trống
  thay vì ra số vô nghĩa — và điều kiện lọc `NOT IsNull(...)` sẽ không cho mã thiếu
  dữ liệu lọt qua.
- `P/TangibleBV` và `EV/EBITDA` chỉ tính đạt khi **dương** — vốn chủ hữu hình âm hoặc
  EBITDA âm bị loại thay vì cho ra tỷ số nhỏ giả tạo.
- `Foreign(..., True)` bật fixup nên số liệu quý được giữ nguyên (pad) qua các phiên
  giữa hai kỳ công bố.
- File dùng được luôn làm Indicator: vẽ nến + 3 đường SMA, mũi tên xanh tại phiên đầu
  tiên mã thỏa **toàn bộ** điều kiện, và Title hiện điểm cùng 3 chỉ số cơ bản chính.
- Với ngân hàng/chứng khoán/bảo hiểm, `EV/EBITDA` và `(Cash+Inv)/Asset` gần như không
  có ý nghĩa so sánh — nên tắt tiêu chí 2 và 3, hoặc chạy riêng theo watchlist ngành.
