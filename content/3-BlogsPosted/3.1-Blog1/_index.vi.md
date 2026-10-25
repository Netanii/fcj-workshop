---
title: "Blog 1"
date: 2026-10-23
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tự xây gợi ý mua kèm trên Lambda + DynamoDB thay cho Amazon Personalize

*Nguyễn Trọng Nhân · Dự án shop-ai, First Cloud AI Journey 2026*

Nhóm mình đang làm shop-ai, một cửa hàng linh kiện máy tính chạy serverless trên AWS. Phần của mình là gợi ý sản phẩm: "Thường mua kèm" ở trang chi tiết và "Dành cho bạn" ở trang chủ. Ban đầu nhóm định dùng Amazon Personalize. Bài này kể vì sao nhóm bỏ Personalize, tự xây mô hình đơn giản như thế nào, và đo được gì.

## 1. Vì sao không dùng Amazon Personalize

Personalize là dịch vụ gợi ý dựng sẵn, chỉ cần đưa dữ liệu vào là có gợi ý. Nhưng khi nhóm đọc kỹ cách tính tiền (ghi lại trong ADR-0016 của repo, ngày 01/10/2026) thì thấy ba vấn đề:

1. **Tính tiền ở ba chỗ:** dữ liệu đưa vào, giờ huấn luyện, và campaign để phục vụ gợi ý theo thời gian thực.
2. **Campaign bị tính tối thiểu 1 request mỗi giây**, kể cả khi không ai gọi. Nhóm ước tính khoảng $0,54 mỗi giờ với recipe v2. Nếu quên tắt, một tháng có thể lên khoảng $394, trong khi ngân sách cả dự án của nhóm dưới $15.
3. **Personalize chỉ dùng được ở Paid plan**, mà cả ba tài khoản của nhóm đang ở Free plan.

Mentor góp ý: nếu chỉ cần gợi ý sản phẩm với chi phí thấp thì nên tự xây mô hình đơn giản trên Lambda và DynamoDB. Kiến trúc của nhóm vốn đã có sẵn chỗ cho việc này: gợi ý được tính theo lô mỗi đêm và ghi vào một bảng, API chỉ việc đọc bảng đó.

> Giá trên là con số nhóm ghi lại lúc ra quyết định. Trước khi dùng cho dự án khác, bạn nên xem lại trang giá của AWS vì giá có thể đổi.

## 2. Kiến trúc

```text
EventBridge Scheduler (mỗi đêm)
        │
        ▼
Lambda Python: đọc sự kiện và đơn hàng → kiểm dữ liệu → tính gợi ý
        │
        ▼
DynamoDB bảng recs: ghi phiên bản mới → so với baseline → đổi con trỏ ACTIVE
        │
        ▼
API GET /api/v1/recs/... (đọc con trỏ, cache 60 giây) → widget trên web
```

Hai quyết định thiết kế quan trọng:

- **Ghi theo phiên bản, bật bằng một con trỏ.** Mỗi lần chạy, job ghi một phiên bản gợi ý mới (số phiên bản là ngày chạy, ví dụ `20261027`). Mỗi loại gợi ý có đúng một item con trỏ (`pk=META`, `sk=ACTIVE#FOR_YOU`, `ACTIVE#ALSO_BOUGHT` hoặc `ACTIVE#BESTSELLER`) cho biết phiên bản nào đang hiển thị. Bật bản mới hay quay về bản cũ chỉ cần sửa đúng một item, áp dụng cho mọi người dùng cùng lúc. Nếu job đứt giữa chừng, web không bao giờ trộn hai phiên bản.
- **Phiên bản kém hơn baseline thì không bật.** Job tự đo chỉ số; bản mới không tốt bằng gợi ý hàng bán chạy thì giữ nguyên bản cũ.

## 3. Ba cách gợi ý

**Baseline: hàng bán chạy.** Đếm số đơn mua của từng sản phẩm, cho tất cả và cho từng loại sản phẩm. Lúc web mới chạy thì chưa có nhiều đơn, nên job dùng cửa sổ ngắn nhất có ít nhất 5 đơn: 30 ngày, nếu chưa đủ thì 90 ngày, rồi 365 ngày. Nếu vẫn chưa đủ thì xếp theo hàng mới nhất, và con trỏ ghi lại cách xếp (`basis`) để API biết. Cách này dùng cho người mới chưa có lịch sử, và là mốc để so sánh hai mô hình bên dưới.

**"Thường mua kèm": luật mua kèm (association rules).** Với mỗi cặp sản phẩm A, B nằm chung một đơn:

- support = số đơn có cả A và B / tổng số đơn
- confidence(A→B) = số đơn có cả A và B / số đơn có A
- lift(A→B) = confidence(A→B) / tỉ lệ đơn có B

Lift lớn hơn 1 nghĩa là người mua A mua B nhiều hơn mức bình thường. Mình lọc các cặp xuất hiện ít nhất 2 đơn, confidence tối thiểu 0,05, rồi xếp theo lift.

**"Dành cho bạn": gợi ý theo món tương tự (item-based collaborative filtering).** Mỗi sản phẩm là một vector điểm theo người dùng: xem được 1 điểm, thêm giỏ 3, mua 5. Hai sản phẩm giống nhau khi cùng được một nhóm người quan tâm, đo bằng độ tương tự cosine. Gợi ý cho một người là các sản phẩm tương tự những gì họ đã xem, thêm giỏ hay mua, trừ những món họ đã tương tác.

Một chi tiết nhỏ nhưng quan trọng về bảo mật: mỗi cặp (người dùng, sản phẩm) chỉ lấy **trọng số lớn nhất**, không cộng dồn. Một người xem cùng sản phẩm 50 lần vẫn chỉ được 1 điểm. Như vậy kẻ tấn công không thể chỉ spam lượt xem để đẩy một sản phẩm lên gợi ý. Đây là một cách giảm thiểu rủi ro "đầu độc gợi ý" trong threat model của nhóm.

**Không dùng thư viện nặng.** Lambda giới hạn gói code 250 MB khi giải nén, nên cả ba cách đều viết bằng thư viện chuẩn của Python, không cần pandas hay scikit-learn. Ở quy mô demo (khoảng 100 sản phẩm, 600 người dùng) tính toán chạy trong chưa tới một giây.

## 4. Đánh giá

Để đánh giá đúng thì không được chia train và test ngẫu nhiên, mà phải **chia theo thời gian**: mô hình chỉ học từ dữ liệu trước một mốc, rồi được chấm trên dữ liệu sau mốc đó. Như vậy mới giống việc đoán tương lai từ quá khứ.

Ba chỉ số:

- **precision@10:** trong 10 gợi ý, trung bình bao nhiêu cái trúng.
- **hit rate@10:** tỉ lệ lần gợi ý có ít nhất 1 cái trúng.
- **coverage:** bao nhiêu phần trăm sản phẩm trong danh mục từng được gợi ý. Baseline gần như gợi ý cùng một danh sách cho mọi người nên coverage rất thấp.

Kết quả trên dữ liệu giả lập (seed 42; 104 sản phẩm; 896 đơn train, 199 đơn test):

| Gợi ý | Cách | precision@10 | hit rate@10 | coverage |
|---|---|---|---|---|
| Thường mua kèm | Baseline bán chạy | 0,0200 | 0,1905 | 0,106 |
| | Luật mua kèm | 0,0495 | 0,3667 | 0,654 |
| Dành cho bạn | Baseline bán chạy | 0,0207 | 0,1957 | 0,164 |
| | Item-based CF | 0,0435 | 0,3804 | 0,942 |

Cả hai mô hình có precision@10 cao gấp khoảng 2 lần baseline và gợi ý được nhiều sản phẩm hơn hẳn. precision@10 nhìn thấp vì mỗi đơn chỉ có 2–3 sản phẩm, nên mỗi lượt chấm chỉ có 1–2 đáp án đúng trong 10 gợi ý.

## 5. Những điều cần nói thật

- **Dữ liệu là giả lập và đã cài sẵn quy luật mua kèm**, ví dụ người mua card màn hình hay mua thêm nguồn. Mô hình bắt được quy luật đó chứng minh pipeline chạy đúng, chứ chưa chứng minh mô hình tốt hơn ngoài đời.
- **Kết quả dao động giữa các seed.** Với seed 42, 1 và 7, item-based CF thắng baseline ở cả ba chỉ số. Với seed 2026, precision vẫn cao hơn (0,0540 so với 0,0490) nhưng hit rate thấp hơn một chút (0,40 so với 0,42). Chỉ có 92 người dùng được chấm nên con số dễ lệch.
- **Các ngưỡng (2 đơn, confidence 0,05, trọng số 1/3/5) là số tạm.** Nhóm sẽ chốt lại khi có dữ liệu sự kiện thật trên web.

## 6. Đã làm được và bước tiếp theo

Đã xong:

- Module `events` ghi lượt xem và thêm giỏ qua `POST /api/v1/events`, ẩn danh `userId` bằng HMAC trước khi lưu (PR #200, #204).
- Lõi pipeline mỗi đêm: ghi bảng `recs` theo phiên bản và đổi con trỏ ACTIVE (PR #202).

Tiếp theo:

1. Chạy pipeline trên Lambda theo lịch của EventBridge Scheduler, có cảnh báo khi job lỗi hoặc không chạy.
2. Ghi lượt mua (`PURCHASE`) từ đơn hàng đã xác nhận, qua hàng đợi SQS.
3. Thí nghiệm đầu độc gợi ý: tạo tài khoản ảo spam lượt xem, đo thứ hạng sản phẩm mục tiêu trước và sau khi bật các lớp phòng thủ.

Bài học lớn nhất của mình: chưa cần một dịch vụ AI đắt tiền để có gợi ý dùng được. Bắt đầu bằng một baseline đơn giản, đo cẩn thận, và chỉ thay bằng thứ phức tạp hơn khi số liệu chứng minh nó tốt hơn.

*Code và số liệu: thư mục `data/` trong repo [hhieuann/shop-ai](https://github.com/hhieuann/shop-ai), PR #170, #172 và #202.*
