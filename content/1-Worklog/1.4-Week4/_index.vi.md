---
title: "Worklog Tuần 4"
date: 2026-10-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Viết tài liệu nghiệp vụ cho phần của mình trong shop-ai: gợi ý hoạt động thế nào và sự kiện hành vi được ghi ra sao.
* Biến script dữ liệu giả lập thành một gói Python đúng chuẩn trong repo của nhóm, có test chạy trong CI.

### Các công việc đã làm trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 3 | - Lên văn phòng <br> - Đọc ADR-0004, ADR-0009, ADR-0012, ADR-0016 và mẫu tài liệu nghiệp vụ của nhóm <br> - Kiểm tra `contracts/openapi.yaml` xem đã có route sự kiện và route gợi ý chưa (lúc đó chưa có) | 06/10 | 09/10 | `docs/adr/`, `docs/business/_template.md` |
| 6 | - Viết `docs/business/recommendation.md`: gợi ý hiện ở đâu, cách tính "Thường mua kèm" (luật mua kèm) và "Dành cho bạn" (item-based collaborative filtering), baseline hàng bán chạy, bảng kết quả `recs`, đánh giá bằng precision@10 và coverage <br> - Viết `docs/business/events.md`: các loại sự kiện, quy tắc chỉ module ordering được tạo `PURCHASE`, ẩn danh `userId` bằng HMAC <br> - Đề xuất thiết kế khoá cho bảng `events` và `recs` trong `dynamodb-design.md` | 09/10 | 10/10 | Issue #106, PR #107 |
| 7 | - Mở và merge PR #107 vào `develop` | 10/10 | 10/10 | PR #107 |
| CN | - Đóng gói bộ sinh dữ liệu giả lập bằng uv (`pyproject.toml`, `uv.lock`), thêm tuỳ chọn `--products` để đọc danh mục sản phẩm thật của nhóm <br> - Viết 11 test pytest: cùng seed ra cùng dữ liệu, không rò rỉ thời gian giữa train và test, đơn bị huỷ không tạo sự kiện `PURCHASE` <br> - Bật khối `uv` trong `.github/dependabot.yml` <br> - Mở và merge PR #109 (CI chạy ruff, mypy, pytest cho `data/`) | 10/10 | 11/10 | Issue #108, PR #109 |
| CN | - Merge ba PR của Dependabot (#110, #111, #115) <br> - Hai PR cùng sửa `pnpm-lock.yaml` nên lockfile bị hỏng sau khi merge cả hai; một bạn trong nhóm đã sửa ở #119 | 11/10 | 11/10 | PR #110, #111, #115, #119 |

### Kết quả đạt được tuần 4:

* Nhóm đã có quy tắc nghiệp vụ bằng văn bản cho gợi ý và sự kiện, nên API, pipeline dữ liệu và phần bảo mật có thể làm theo cùng một tài liệu.
* `data/` đã là một gói Python thật: một lệnh (`uv run data`) sinh dữ liệu giả lập chạy lại ra đúng kết quả, và CI kiểm tra bằng ruff, mypy và 11 test ở mọi pull request.
* Bài học: các PR Dependabot cùng sửa một lockfile phải merge từng cái một và kiểm tra lại CI sau mỗi lần merge. Từ đó nhóm có quy tắc "merge PR Dependabot từng cái một".

### Việc còn lại / tuần sau:

* Thống nhất với trưởng nhóm thiết kế phiên bản và con trỏ ACTIVE của bảng `recs`.
* Viết threat model v0 trong `docs/security/`.
* Làm baseline hàng bán chạy và luật mua kèm chạy offline, so sánh bằng precision@10.
