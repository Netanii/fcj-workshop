---
title: "Worklog Tuần 5"
date: 2026-10-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Chốt thiết kế bảng `recs` với trưởng nhóm.
* Nộp threat model v0 và bảng phân loại dữ liệu.
* Làm cả hai mô hình gợi ý chạy offline và đo so với baseline hàng bán chạy.

### Các công việc đã làm trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 4 | - Lên văn phòng <br> - Bàn thiết kế `recs` với trưởng nhóm: một con trỏ ACTIVE chung cho mỗi loại gợi ý thay vì mỗi người dùng hay mỗi sản phẩm một con trỏ <br> - Đọc checklist phát hành v0.1.0 và xác nhận các bài nghiệm thu C1–C7 (xem sản phẩm) của mình | 14/10 | 14/10 | `docs/releases/v0.1.0.md` |
| 5 | - Cập nhật thiết kế `recs`: item con trỏ `pk=META`, `sk=ACTIVE#FOR_YOU` / `ACTIVE#ALSO_BOUGHT`, rank đệm số 0, `ttl` kiểu số (DynamoDB TTL cần epoch giây), TTL 7 ngày kèm quy tắc gia hạn để phiên bản đang bật không bao giờ hết hạn | 15/10 | 16/10 | Issue #165, PR #166 |
| 6 | - Viết threat model v0 theo STRIDE cho ba luồng (đăng nhập, đặt hàng, ghi sự kiện) và bảng phân loại dữ liệu <br> - Kiểm tra mọi chỗ ghi "đã có" với code thật (cấu hình Cognito, CloudFront, giới hạn tần suất, các bước kiểm tra bảo mật trong CI) <br> - Phát hiện mới: CloudFront chưa có header CSP trong khi token nằm trong cookie JavaScript đọc được; tồn kho có thể bị giữ bằng đơn COD giả; role của ordering có thể sửa giá; URL API Gateway đi vòng qua WAF | 16/10 | 16/10 | Issue #167, PR #168 |
| 6 | - Viết luật mua kèm (support, confidence, lift) và baseline hàng bán chạy bằng Python thuần <br> - Viết bước đánh giá offline: chia train/test theo thời gian, precision@10, hit rate@10, coverage <br> - 10 test mới | 16/10 | 16/10 | Issue #169, PR #170 |
| 7 | - Viết item-based collaborative filtering (độ tương tự cosine). Mỗi cặp người dùng–sản phẩm lấy trọng số sự kiện lớn nhất thay vì cộng dồn, nên spam lượt xem không đẩy điểm lên được <br> - Đánh giá so với baseline trên bốn random seed; 7 test mới | 17/10 | 17/10 | Issue #171, PR #172 |
| 7 | - Gửi năm câu hỏi thiết kế cho nhóm (route sự kiện, cách ordering báo đơn đã mua, nơi lưu khoá HMAC, mã hoá bucket dữ liệu, các việc tiếp theo của threat model); trưởng nhóm trả lời hết ngay tối đó | 17/10 | 17/10 | Chat nhóm |
| CN | - Chuẩn bị thay đổi hợp đồng `POST /api/v1/events` (PR mở ngày 19/10, xem tuần 6) | 18/10 | 18/10 | Issue #173 |

### Kết quả đạt được tuần 5:

* Merge 4 pull request (#166, #168, #170, #172), CI đều xanh.
* Cả hai loại gợi ý đã chạy offline và thắng baseline hàng bán chạy trên dữ liệu giả lập (seed 42):

| Gợi ý | Cách | precision@10 | coverage |
| --- | --- | --- | --- |
| Thường mua kèm | Baseline bán chạy | 0,0200 | 0,106 |
| | Luật mua kèm | 0,0495 | 0,654 |
| Dành cho bạn | Baseline bán chạy | 0,0207 | 0,164 |
| | Item-based CF | 0,0435 | 0,942 |

* Giới hạn đã biết: dữ liệu giả lập có cài sẵn quy luật mua kèm, nên các con số này chứng minh pipeline chạy đúng, chưa chứng minh mô hình tốt hơn ngoài đời. Với một seed (2026), item-based CF có hit rate thấp hơn baseline một chút; điều này đã ghi trong `data/README.md`.
* Threat model v0 cho nhóm một danh sách rủi ro chung; ngày 17/10 trưởng nhóm đã chia các điểm còn mở thành việc có người phụ trách và hạn.

### Việc còn lại / tuần sau:

* Merge route sự kiện vào hợp đồng API sau khi review.
* Viết PR hạ tầng cho bucket dữ liệu huấn luyện (SSE-KMS với khoá AWS quản lý, bật bucket key).
* Chạy nghiệm thu C1–C7 trên staging khi có nhánh `release/v0.1.0` (dự kiến 16/10, đến cuối tuần vẫn chưa tạo).
* Viết Blog 1 (hạn 26/10).
