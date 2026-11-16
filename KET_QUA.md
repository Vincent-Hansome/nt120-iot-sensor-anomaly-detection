# Kết quả chạy kiểm chứng — 04/11/2026

Notebook đã chạy toàn bộ 8 ô code trên dữ liệu gốc, không có lỗi; các assert về parser, chia tập, scaler, mô phỏng và số dòng xuất đều đạt. Đã kiểm tra trực quan biểu đồ toàn kỳ node 1, biểu đồ chỉ số và bốn cửa sổ cục bộ của hai node. Cấu hình và phiên bản thư viện nằm trong `outputs/run_metadata.json`.

## Dữ liệu và số điểm bị đánh dấu

Tệp gốc có 2.313.682 dòng. Node 1 có 43.047 bản ghi, node 2 có 46.915 bản ghi. Với nhiệt độ và điện áp, cả hai node không mất dòng ở các bước kiểm tra timestamp, giá trị thiếu/không hữu hạn và trùng hoàn toàn. Điều này không có nghĩa mọi giá trị đo đều chính xác. Có các khoảng ngắt dài hơn 24 giờ ở cả hai node.

Huấn luyện trên khoảng 70% đầu theo thời gian, kiểm tra trên phần còn lại; `contamination=0.02`, `random_state=42`, 200 cây.

| Node | Đặc trưng | Số điểm test | Điểm bị đánh dấu | Tỷ lệ |
| --- | --- | ---: | ---: | ---: |
| 1 | Nhiệt độ | 12.915 | 6.991 | 54,13% |
| 1 | Nhiệt độ + điện áp | 12.915 | 9.988 | 77,34% |
| 2 | Nhiệt độ | 14.075 | 9.399 | 66,78% |
| 2 | Nhiệt độ + điện áp | 14.075 | 11.526 | 81,89% |

Trong lần chạy này, mọi điểm test bị cấu hình nhiệt độ đánh dấu cũng bị cấu hình kết hợp đánh dấu. Cấu hình kết hợp thêm 2.997 điểm ở node 1 và 2.127 điểm ở node 2. Đây là khác biệt phạm vi cảnh báo, chưa phải bằng chứng cải thiện độ chính xác.

## Nhận xét từ biểu đồ

Biểu đồ toàn kỳ node 1 cho thấy nhiệt độ tăng mạnh sau khoảng 22/03, rồi đạt gần 122,153°C; điện áp có xu hướng giảm. Đây là bằng chứng phân phối cuối kỳ khác rõ giai đoạn huấn luyện. Chưa có thông tin đủ để xác nhận nguyên nhân là pin hay phần cứng.

Bốn cửa sổ ±2 giờ đã xem, theo `raw_line` trong `outputs/evaluation/manual_review.csv`, cấu hình nhiệt độ + điện áp:

- Node 1, dòng 22347: nhiệt độ tăng liên tục khoảng 26–34°C, điện áp giảm theo bậc; cảnh báo nằm trong một đợt thay đổi kéo dài, không chỉ là xung đơn lẻ.
- Node 2, dòng 61469: nhiệt độ tăng khoảng 26–32°C, điện áp giảm; có hình thái tương tự node 1 trong cùng đêm.
- Node 1, dòng 42094: điểm đối chứng nằm trên đoạn nhiệt độ giảm đều, chưa thấy xung cục bộ rõ tại điểm chọn.
- Node 2, dòng 86877: điểm đối chứng nằm trên đoạn nhiệt độ tăng đều; chưa thấy xung đơn lẻ tại điểm chọn. Những điểm lân cận vẫn có thể bị đánh dấu.

Đã ghi nhận xét cho bốn dòng này trong CSV; các dòng còn lại chưa được rà soát. Đây là kiểm tra trực quan sơ bộ bằng trợ lý, chưa phải nhãn được chuyên gia xác nhận.

## Thí nghiệm bán tổng hợp

| Node | Đặc trưng | Precision | Recall | F1 |
| --- | --- | ---: | ---: | ---: |
| 1 | Nhiệt độ | 0,0719 | 0,7981 | 0,1319 |
| 1 | Nhiệt độ + điện áp | 0,0585 | 0,9146 | 0,1100 |
| 2 | Nhiệt độ | 0,0613 | 0,8333 | 0,1143 |
| 2 | Nhiệt độ + điện áp | 0,0563 | 0,9316 | 0,1062 |

Cấu hình kết hợp tăng Recall nhưng giảm Precision/F1 trong kịch bản này. Rất nhiều điểm nền không sửa đã bị mô hình đánh dấu, nên giả định tất cả chúng là âm tính làm Precision thấp. Không thể dùng bảng này để ước lượng độ chính xác trên lỗi thật.

Recall cho can thiệp chỉ điện áp của mô hình chỉ nhiệt độ vẫn lớn hơn 0 vì một số điểm đã bị đánh dấu do nhiệt độ ban đầu. Assert xác nhận việc sửa điện áp không đổi dự đoán của mô hình chỉ nhiệt độ; không được diễn giải đó là khả năng nhận biết sụt điện áp.

Các số trên chỉ áp dụng cho cấu hình/lần chạy đã ghi. Xem CSV và biểu đồ trong `outputs/` để phân tích chi tiết; chạy cấu hình khác sẽ cập nhật đầu ra nhưng không tự cập nhật bản nhận xét này.
