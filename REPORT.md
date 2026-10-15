# REPORT — Lab Day 12

| | |
|---|---|
| Họ và tên | Đinh Quốc Bảo |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602933 |
| GitHub | nova-io-vn |
| Seed chia dữ liệu / seed mô hình | 6005 / 2671, 2672, 2673 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 3 |

## Kết quả cuối

Số liệu lấy từ `evidence/openloop.json`, `closedloop.json` và `gate.json`. L2 là trung bình qua seed; tỷ lệ va chạm của mô hình học là dải qua 3 seed trên 2.883 quỹ đạo kiểm tra.

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---:|---:|---|
| constant-velocity | 1,273 | 20,26 | BLOCK |
| IDM | 1,537 | 0,104 | Baseline |
| EgoMLP | 0,760 | 17,66–36,04 | BLOCK |
| PercMLP | 0,616 | 0,45–2,43 | BLOCK |
| PercMLP+ (sau khi sửa) | 0,817 | 0,17–0,21 | RELEASE theo cổng lab |

## Câu 1 · Dữ liệu
Có 992.679 hàng thuộc 9.993 quỹ đạo thô; phép loại trùng bỏ 383 quỹ đạo, còn 9.610. Bản sao ở các ID xa nhau có thể lọt vào cả train và test, làm điểm kiểm thử lạc quan giả.
Tương quan của `Acc_LV` với đạo hàm `Speed_LV` chỉ 0,045, trong khi cột gia tốc xe theo sau đạt 0,929; vì vậy tôi không dùng `Acc_LV` để tìm xe dẫn phanh.
Tôi lấy sai phân của `Speed_LV` rồi làm mượt thành `a_lead`; kết quả có phân vị giảm tốc p1 khoảng −1,95 m/s². Tập được chia theo toàn bộ quỹ đạo với seed 6005.
Nếu chia theo khung, các khung liền kề của cùng xe sẽ có mặt ở cả hai phía và làm rò rỉ chuỗi thời gian, nhất là với cửa sổ lịch sử 2 giây và dự báo 3 giây.

## Câu 2 · Phát lại
Tôi phát lại 600 đoạn. Chạy lại cùng mã cho 0 đoạn lệch; bản float32/refactor khác bit ở cả 600 đoạn nhưng 0 đoạn vượt `atol=rtol=0,001`, sai khác tuyệt đối lớn nhất 0,000403 m/s². Dung sai này chấp nhận khác biệt số học mà vẫn bắt thay đổi hành vi.
Thêm AEB làm 8/600 đoạn vượt dung sai. Ở đoạn 61, các khung 98–101 có TTC 1,474; 1,433; 1,431; 1,490 giây, đều dưới ngưỡng 1,5 giây; lệnh cũ từ −5,436 đến −5,911 m/s² được ép thành −6,0 m/s².
Chạy nhanh nhất có 0 khung trễ và 0 khung rớt. Chạy theo thời gian thật tạo 584 khung trễ, 578 khung rớt trên 59.849 khung, khiến 379/600 đoạn thay đổi vượt dung sai.
Vì lệnh phải sẵn sàng trước khung kế tiếp, các đột biến thời gian xử lý chỉ lộ ra khi đồng hồ phát lại và deadline thực được áp dụng.

## Câu 3 · Kịch bản
Lưới cắt vào dùng tốc độ cruise 12 m/s, khoảng cách 2,76–59,31 m và tốc độ tiến gần 0–12 m/s: 60 × 45 = 2.700 ô. Với độ trễ 0, vùng bất khả thi là 2,22%; ở 0,6 giây tăng lên 7,44% (thêm 5,22 điểm phần trăm).
Vùng va chạm của IDM cũng đổi từ 2,22% lên 7,44%, trùng vùng không thể tránh theo biên phanh tối đa; số ô mô phỏng sai khác xa biên là 0 ở cả hai độ trễ.
Chỉ 614/9.610 quỹ đạo (6,39%) chứa sự kiện xe dẫn phanh được khai thác. Giới hạn phanh mạnh p99,9 của người lái trong log là 3,62 m/s², trong khi 8,56% ô lưới đòi hỏi phanh mạnh hơn.
Do đó log phủ rất thưa vùng khó. Đánh giá trung bình trên log không đủ để kết luận về các ca cắt vào có khoảng cách ngắn hoặc tốc độ tiến gần cao.

## Câu 4 · Vòng hở và vòng kín
Theo L2, thứ hạng là PercMLP 0,616 m (1), EgoMLP 0,760 m (2), constant-velocity 1,273 m (3), IDM 1,537 m (4). Theo va chạm trung bình trên log, thứ hạng đổi thành IDM 0,104% (1), PercMLP 1,226% (2), constant-velocity 20,26% (3), EgoMLP 25,37% (4).
Các cặp đổi thứ tự gồm IDM/PercMLP, IDM/EgoMLP, IDM/constant-velocity và EgoMLP/constant-velocity. Điều đó cho thấy sai số dự báo trên quỹ đạo người thật không giữ nguyên thứ tự khi bộ điều khiển tự tạo trạng thái mới.
IDM có L2 cao vì mục tiêu của luật là giữ khoảng cách và tránh va chạm, không sao chép đúng tốc độ tương lai của người lái. Dù vậy nó chỉ đâm 3/2.883 quỹ đạo, còn trên lưới độ trễ 0,6 giây nó không có vùng va chạm có thể tránh.
EgoMLP nhìn lịch sử tốc độ xe mình nhưng thiếu khoảng cách và tốc độ xe dẫn; L2 của nó tốt hơn constant-velocity, song đâm 17,66–36,04% và tới 65,4% trên lát cắt xe dẫn phanh ở seed xấu nhất.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP đổi từ 17,66% (seed 2673) đến 36,04% (seed 2672), biên độ 18,39 điểm phần trăm trên cùng 2.883 quỹ đạo kiểm tra. Seed 2671 nằm giữa, ở 22,41%.
Khoảng Wilson 95% của seed tốt nhất là 16,31–19,09%; của seed xấu nhất là 34,31–37,81%, không chồng nhau. Chênh lệch này lớn hơn nhiễu lấy mẫu đơn thuần của từng tỷ lệ.
Nếu chỉ báo seed 2673, ta có thể tuyên bố va chạm 17,66%; chọn seed 2672 lại là 36,04%. Cả hai đều đúng cho một lần huấn luyện nhưng che mất sự bất ổn của mô hình.
Vì vậy cổng xét seed xấu nhất và quy tắc R5 về độ dao động; EgoMLP rớt R5, còn PercMLP+ ổn định 0,17–0,21% sau sửa.

## Câu 6 · Tác nhân phản ứng
Xe sau IDM được hiệu chỉnh ở `T=1,4 s`: khoảng cách Wasserstein đến phân bố headway người thật là 0,354, tốt hơn tác nhân không bao giờ phản ứng (0,850); trung vị headway người thật là 1,856 giây. Đây vẫn chỉ là một mô hình gần đúng.
Với độ trễ 0,6 giây, tỷ lệ ô có bất kỳ va chạm nào khi xe sau phát lại / phản ứng là: constant-velocity 77,6% / 77,6%; IDM 75,7% / 7,4%; EgoMLP 77,9% / 77,9%; PercMLP 77,2% / 26,7%.
Riêng va chạm phía sau của IDM là 68,3% khi phát lại nhưng 0% khi xe sau phản ứng. Xe sau phát lại giữ tốc độ cũ dù IDM phanh, nên nhiều trường hợp bị gán lỗi cho xe thử dù tương tác thực tế đã thay đổi.
Khi dùng xe sau phản ứng, IDM trở lại nhóm tốt nhất trên lưới và PercMLP vẫn kém hơn. Kết luận phải nêu rõ giả định hành vi của tác nhân khác, không dựa một thế giới phát lại cố định.

## Câu 7 · Cổng phát hành và sửa
PercMLP ban đầu bị chặn bởi R2–R6: seed xấu nhất đâm 2,43% trên log (cận Wilson 3,06%, so IDM 0,10%), lát cắt phanh 8,5%, vùng cắt vào có thể tránh mà vẫn đâm 19,26%, seed dao động 0,45–2,43%, và va chạm trong thế giới xe sau phản ứng 26,7% so IDM 7,4%.
Hai nguyên nhân chính là log thiếu ca cắt vào mạnh (8,56% ô đòi phanh vượt người lái trong log) và sai số tích luỹ khi mạng tự điều khiển sang trạng thái khác log. Tôi thêm 197.000 cửa sổ từ 20.000 kịch bản cắt vào, gắn nhãn theo IDM, rồi chạy 2 vòng DAgger trên trạng thái mô hình tự gặp.
Sau sửa, PercMLP+ qua cả 7 quy tắc: seed xấu nhất 0,21% va chạm log (cận 0,45%), lát cắt phanh 0,5%, vùng lưới có thể tránh mà vẫn đâm 0,33%, thế giới phản ứng 7,8%. L2 vòng hở lại tăng từ 0,616 lên 0,817 m, cho thấy tối ưu L2 không đồng nghĩa an toàn vòng kín.
PercMLP+ chưa tốt hơn IDM: IDM đâm 0,104% và có vùng va chạm có thể tránh 0%. Mạng học nhãn từ chính luật IDM. Tôi chấp nhận kết quả RELEASE của cổng lab như điều kiện để nghiên cứu tiếp, nhưng chưa đề xuất thay IDM ngoài đời trước kiểm thử ngoài phân bố và trên hệ thống thật.

## Câu 8 · Bài lab này không chứng minh được gì
Mô phỏng chỉ có chuyển động dọc một chiều và xe dẫn từ log không phản ứng với xe thử. Vì thế 0,21% va chạm log và 0,33% vùng lưới của PercMLP+ không bao quát đổi làn, người đi bộ, xe cắt ngang hoặc thương lượng giữa nhiều tác nhân.
Thứ hai, quan sát trong bài gần như sạch; thử nhiễu chỉ dùng khoảng cách ±1 m và tốc độ xe dẫn ±0,5 m/s. Nó chưa đo lỗi phát hiện vật thể, mất cảm biến, đồng bộ thời gian hay các khung trễ/rớt thực tế (mô phỏng replay đã cho 584 trễ và 578 rớt).
Thứ ba, lưới huấn luyện cắt vào chỉ thấy tốc độ cruise đến 16 m/s. Ở kiểm tra 20 m/s, PercMLP+ vẫn có 0,07–2,04% vùng va chạm có thể tránh theo seed, trong khi IDM là 0%; kết luận không chắc khi vượt phân bố huấn luyện.
Cuối cùng, cổng chỉ dựa 2.883 quỹ đạo giữ lại và mô hình vật lý giả định; nó không chứng minh an toàn ở xác suất sự cố hiếm trong triển khai. Bước tiếp theo là tạo kịch bản đa tác nhân, đưa lỗi cảm biến và độ trễ đo thật vào hardware-in-the-loop, kiểm thử ngoài phân bố với nhiều seed hơn, rồi đánh giá có giám sát trên xe thật trước quyết định phát hành.
