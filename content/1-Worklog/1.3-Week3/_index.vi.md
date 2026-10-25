---
title: "Worklog Tuần 3"
date: 2026-10-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu dự án nhóm (shop-ai: cửa hàng linh kiện điện tử serverless trên AWS, có gợi ý mua kèm bằng AI) và xác định vai trò của mình: dữ liệu, gợi ý, bảo mật dữ liệu và AI.
* Chuẩn bị nền dữ liệu và bảo mật cho tính năng gợi ý: dữ liệu giả lập, mô hình dữ liệu, threat model đầu tiên.

### Các công việc đã làm trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế và viết bộ sinh dữ liệu giả lập bằng Python: danh mục sản phẩm, 4 nhóm khách hàng, quy luật mua kèm, nhiễu, chia train/test theo thời gian <br> - Thêm hành vi giỏ hàng thực tế: đơn bị huỷ, món bị bỏ khỏi giỏ, món để trong giỏ mua sau | 28/09 | 02/10 | `generate_synthetic.py` |
| 3 | - Tìm hiểu cách tính tiền của Amazon Personalize (campaign thời gian thực bị tính mức tối thiểu so với batch inference) và giới hạn của Free plan <br> - 01/10: theo góp ý của mentor, nhóm quyết định không dùng Personalize mà tự xây mô hình trên Lambda + DynamoDB (ADR-0016) | 29/09 | 01/10 | Trang giá Amazon Personalize, ADR-0016 |
| 4 | - Đọc kế hoạch dự án nhóm và ghi chú review ngày 30/09 <br> - Thống nhất phạm vi của mình với nhóm: dữ liệu, gợi ý, bảo mật dữ liệu và AI | 30/09 | 30/09 | README của repo và `docs/project-plan.md` |
| 5 | - Lên văn phòng <br> - Viết bản nháp đầu tiên của threat model STRIDE cho pipeline gợi ý: 6 ranh giới tin cậy, các mối đe doạ và cách giảm thiểu đề xuất <br> - Lên kế hoạch thí nghiệm đầu độc dữ liệu cho tuần 5 | 01/10 | 01/10 | `threat-model.md`, Microsoft STRIDE |
| 6 | - Thiết kế mô hình dữ liệu (ERD) bằng dbdiagram.io: users, products, orders, order_items, cart_items, events, recs <br> - Định nghĩa các loại sự kiện và quy tắc chỉ dịch vụ đặt hàng mới được tạo sự kiện `purchase` <br> - Chia sẻ schema với các bạn trong nhóm để làm hợp đồng API | 02/10 | 02/10 | dbdiagram.io |

### Kết quả đạt được tuần 3:

* Hiểu kiến trúc dự án và trách nhiệm của mình trong nhóm.
* Hiểu rủi ro chi phí của Amazon Personalize; nhóm thay nó bằng mô hình tự xây (luật mua kèm, item-based collaborative filtering, baseline hàng bán chạy) chạy trên Lambda + DynamoDB (ADR-0016).
* Viết được bộ sinh dữ liệu giả lập chạy lại ra đúng kết quả (cố định random seed), tạo ra:
  * sản phẩm, người dùng, đơn hàng, món trong đơn, món trong giỏ;
  * sự kiện tương tác theo định dạng tương thích Amazon Personalize;
  * chia train/test theo thời gian để tránh rò rỉ dữ liệu.
* Thiết kế mô hình dữ liệu 7 bảng và thống nhất quy tắc sự kiện với nhóm, gồm cả cách chống sự kiện `purchase` giả.
* Viết threat model đầu tiên, bao quát ranh giới trình duyệt–API, bucket dữ liệu huấn luyện, Personalize và bảng gợi ý; hai rủi ro chính mình sẽ kiểm thử là đầu độc dữ liệu và IDOR.

### Việc còn lại / tuần sau:

* Làm baseline theo luật (support, confidence, lift) và các chỉ số đánh giá (precision@k, recall@k, coverage).
* Viết hàm ẩn danh `user_id` bằng HMAC.
