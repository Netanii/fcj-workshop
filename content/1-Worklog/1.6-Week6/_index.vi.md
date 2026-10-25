---
title: "Worklog Tuần 6"
date: 2026-10-23
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

> Đang cập nhật. Đã ghi đến thứ 6 ngày 23/10.

### Mục tiêu tuần 6:

* Bắt đầu Sprint 2 (20/10–02/11): module events, bucket dữ liệu huấn luyện và pipeline chạy mỗi đêm.
* Chạy nghiệm thu C1–C7 trên staging cho bản v0.1.0.
* Chuẩn bị Blog 1 và trang workshop cho báo cáo này.

### Các công việc đã làm trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thêm `POST /api/v1/events` vào `contracts/openapi.yaml`: bắt buộc đăng nhập, body chỉ có `{ productId, eventType }` với `VIEW` hoặc `ADD_TO_CART`, `additionalProperties: false` để chặn `userId`, thời gian và `PURCHASE` <br> - Sinh lại kiểu API cho web và lint hợp đồng (hợp lệ, không có cảnh báo mới) <br> - Mở PR #174 và nhờ bạn phụ trách hợp đồng review <br> - Trưởng nhóm merge PR #174 vào `develop` | 18/10 | 19/10 | Issue #173, PR #174 |
| 2 | - Review PR Dependabot #175 (ruff 0.17.0 → 0.17.1 trong `data/`): chạy ruff, mypy và 28 test trên nhánh PR, approve | 19/10 | 19/10 | PR #175 |
| 3 | - Thêm bucket S3 dữ liệu huấn luyện trong `infra/`: SSE-KMS với khoá AWS quản lý `aws/s3`, bật bucket key, chặn truy cập công khai, bắt buộc TLS, bật versioning và xoá bản cũ sau 30 ngày <br> - 6 unit test CDK; typecheck, lint, 60 test infra và `cdk synth` với cdk-nag đều qua <br> - Đọc `cdk diff` so với staging trong CI và giải thích từng thay đổi cho người review <br> - Mở PR #179; trưởng nhóm review và merge vào `develop` | 20/10 | 21/10 | Issue #178, PR #179 |
| 4 | - Review bảo mật trust policy OIDC của GitHub Actions (`infra/bootstrap/github-oidc.yaml`): trust policy khớp chính xác `aud` và `sub`, nhưng role deploy có thể assume role thực thi của CDK bootstrap (quyền AdministratorAccess), và dev, staging, prod dùng chung một tài khoản AWS <br> - Trưởng nhóm kiểm tra quy tắc environment trên GitHub và chọn chấp nhận rủi ro | 21/10 | 21/10 | Issue #180 |
| 4 | - Review và approve PR #182 (Lambda pre sign-up bị timeout khi cold start: 512 MB, 5 s); chạy test infra trên nhánh PR (22/22) | 21/10 | 21/10 | PR #182 |
| 4 | - Nghiệm thu C1–C7 trên staging cho v0.1.0: qua 5/7 <br> - Mở bug #184 (trang chi tiết không có thông số và ảnh) và #185 (biểu tượng giỏ không hiện số món) <br> - Review và approve PR sửa lỗi #190, rồi kiểm tra lại trên staging: thông số đã hiện | 21/10 | 21/10 | Issue #184, #185, PR #190 |
| 4 | - Cập nhật threat model: rủi ro mới C6 (role deploy là admin toàn tài khoản, đã chấp nhận, từ #180) và E9 (tin nhắn giả trên topic `order-confirmed` có thể tạo lượt mua giả); ghi lại các quyết định ngày 17/10 của nhóm và người phụ trách | 21/10 | 21/10 | Issue #186, PR #187 |
| 4 | - Sửa bộ sinh dữ liệu theo review của trưởng nhóm: mặc định dùng danh mục sản phẩm thật, loại sự kiện viết hoa giống API, một định dạng thời gian ISO 8601 UTC, 10% người dùng mới không có lịch sử huấn luyện, xuất UTF-8 trên Windows <br> - 10 test mới (tổng 38); chạy lại hai bài đánh giá offline trên danh mục thật: mô hình vẫn thắng baseline trên 4 seed | 21/10 | 21/10 | Issue #188, PR #189 |
| 5 | - Cài máy để chạy integration test: WSL 2, Docker Desktop (tắt tự khởi động cho đỡ tốn RAM), AWS CLI | 22/10 | 22/10 | Tài liệu Docker |
| 5 | - Cập nhật các mục O6, O8 và B6 của threat model theo yêu cầu của một bạn trong nhóm | 22/10 | 22/10 | Issue #197, PR #198 |
| 5 | - Module events (`POST /api/v1/events`): domain, application và adapter DynamoDB theo kiến trúc hexagonal; `userId` được thay bằng mã băm HMAC-SHA256 với khoá lấy từ SSM Parameter Store; ID sự kiện là ULID; item tự hết hạn sau 90 ngày bằng DynamoDB TTL <br> - Unit test và integration test với DynamoDB Local trong Docker | 22/10 | 22/10 | Issue #199, PR #200 |
| 5 | - Lõi pipeline mỗi đêm bằng Python: ghi dòng `recs` theo phiên bản, con trỏ ACTIVE đổi cho mọi người dùng cùng lúc, phiên bản kém hơn baseline thì không bật, điểm "Dành cho bạn" theo từng người | 22/10 | 22/10 | Issue #201, PR #202 |
| 5 | - Hạ tầng cho module events bằng CDK: Lambda, bảng `events`, route API, chỉ được đọc đúng một khoá SSM <br> - Đọc `cdk diff` so với staging trong CI: phát hiện một thay đổi pre sign-up sẽ bị đảo ngược (trưởng nhóm đã sửa bằng merge ngược từ nhánh release) và số tài khoản AWS bị lộ trong phần Summary của CI (rủi ro thấp, đã báo) | 22/10 | 22/10 | Issue #203, PR #204 |
| 5 | - Ghi kết quả C1–C7 vào checklist phát hành | 22/10 | 22/10 | Issue #218, PR #220 |
| 5 | - Chuyển hàm băm `userId` ra chỗ dùng chung để API gợi ý băm người dùng giống hệt module events (không import chéo module) | 22/10 | 22/10 | Issue #222, PR #225 |
| 5 | - Review lõi API gợi ý của trưởng nhóm (PR #224) và chốt định dạng `recs`: cột `productId`, tối đa 20 dòng mỗi anchor, loại `BESTSELLER` nới cửa sổ 30/90/365 ngày và cột `basis` | 22/10 | 22/10 | PR #213, PR #224 |
| 6 | - Sửa pipeline theo định dạng `recs` mới và thêm job hàng bán chạy; ghi quyền đọc bảng chéo module vào ADR-0017 | 22/10 | | PR định dạng `recs` |
| 6 | - Tạo trang workshop này từ template FCAJ, đăng lên GitHub Pages, viết worklog bằng tiếng Anh và tiếng Việt | 23/10 | 23/10 | [fcj-workshop](https://github.com/Netanii/fcj-workshop) |

### Kết quả đạt được tuần 6:

* Module events đã merge cùng hạ tầng: website ghi được lượt xem và thêm giỏ, và sự kiện lưu lại không bao giờ chứa ID thật của người dùng.
* Lõi pipeline mỗi đêm đã merge; API gợi ý và pipeline giờ dùng chung một hàm băm và một định dạng bảng.
* Bản v0.1.0 đã lên production sau khi mình nghiệm thu trên staging; bug #184 mình tìm ra đã được sửa và kiểm tra lại.
* [Bổ sung vào cuối tuần.]
