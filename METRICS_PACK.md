# Metrics Pack — Nera

Học viên: Vũ Thế Lực · MHV 2A202602008
Dự án: Nera — trợ lý AI tìm nhà và đặt lịch xem nhà (nerahome.space)

---

## 00 — Dự án, persona, core job

**Dự án:** Nera, trợ lý AI giúp người tìm nhà mô tả nhu cầu bằng lời nói tự nhiên, nhận gợi ý bất động sản có thật kèm lý do phù hợp, và đặt lịch đi xem với nhân viên sale.

**Persona:** Người mua nhà để ở lần đầu, 25–35 tuổi, đang đi làm tại Hà Nội, ngân sách 2–6 tỷ. Tìm nhà vào buổi tối và cuối tuần, không rành thuật ngữ bất động sản.

**Core job:** *"Tôi muốn tìm được vài căn đáng để bỏ một buổi đi xem thật, mà không phải đọc hàng trăm tin đăng và không phải kể lại nhu cầu từ đầu mỗi lần quay lại."*

**Use case chọn phân tích:** Từ lúc mô tả nhu cầu tới lúc có lịch hẹn xem nhà. Không phân tích phần quản lý của sale hay admin.

---

## 01 — Core Action Card

### Phân biệt bốn khái niệm

| Khái niệm | Câu trả lời cho Nera |
|---|---|
| Core job | Tìm được căn đáng bỏ công đi xem, không phải khai lại nhu cầu |
| Core action | Gửi yêu cầu đặt lịch xem một căn cụ thể |
| Core value | Có một buổi hẹn thật với sale thật, để đánh giá căn nhà bằng mắt mình |
| Core value event | `viewing_completed` — buổi xem đã diễn ra |

Core action và core value event **không trùng nhau**. Gửi yêu cầu là hành vi của khách; buổi xem hoàn tất mới là bằng chứng value đã xảy ra, và nó phụ thuộc vào sale xác nhận.

### Core Action Card

| Thành phần | Nội dung |
|---|---|
| **Target user** | Người mua nhà để ở lần đầu, đã đăng nhập, đang trong một chu kỳ tìm nhà |
| **Core job** | Tìm được căn đáng đi xem mà không phải tự lọc và không phải khai lại nhu cầu |
| **Core action** | Gửi yêu cầu đặt lịch xem một căn cụ thể, kèm khung giờ đã chọn |
| **Object** | Một bất động sản cụ thể trong kho (`property_id`) và một khung giờ của một sale |
| **Preconditions** | Đã đăng nhập; hồ sơ nhu cầu có tối thiểu khu vực và ngân sách; đã xem ít nhất một danh sách gợi ý; căn còn trạng thái có thể xem |
| **Completion rule** | Hệ thống tạo được bản ghi yêu cầu với `property_id`, khung giờ, `sale_id`, và trả về mã yêu cầu cho khách. Bấm nút mà lỗi hoặc hết slot thì chưa tính là hoàn tất |
| **Core value** | Khách có lịch hẹn xác định với người thật, không phải gọi điện dò từng nơi |
| **Evidence of value** | Sale xác nhận lịch và buổi xem diễn ra |
| **Candidate event** | `booking_requested` |

### Tự kiểm 5 tiêu chí

| Tiêu chí | Đạt | Lý do |
|---|:--:|---|
| Gần core value | ✓ | Khách chỉ bỏ một buổi đi xem khi thật sự tin căn đó đáng xem. Đây là lúc họ trả giá bằng thời gian thật |
| Có thể lặp lại | ✓ | Một chu kỳ tìm nhà thường đi xem vài căn để so sánh trước khi chốt |
| Có thể quan sát | ✓ | Completion rule rõ: có bản ghi yêu cầu kèm mã. Không mơ hồ |
| Có ý nghĩa | ✓ | Nhiều yêu cầu đặt lịch hơn nghĩa là Nera đưa ra được nhiều căn đáng đi xem hơn. Có counter-metric ở mục 03 để chặn trường hợp đặt bừa |
| Có thể tác động | ✓ | Cải thiện chất lượng trích xuất nhu cầu, chất lượng gợi ý và phần giải thích lý do đều làm tăng khả năng hành vi xảy ra |

**Qua 5/5.**

**Vì sao không phải "mở app" hay "hỏi AI":** Mở app là thao tác giao diện, xảy ra cả khi khách vào rồi thoát ngay. Hỏi AI là hành vi của sản phẩm chat bất kỳ, không riêng Nera, và hỏi xong vẫn có thể không tìm được gì. Cả hai đều không chứng minh khách nhận được value nào.

---

## 02 — Action Nature Card + kết luận cadence

| Thành phần | Nội dung |
|---|---|
| **Actor** | Cá nhân người dùng. Không phải account chung hay team |
| **Intent** | "Tôi đã thấy một căn đáng đi xem, tôi muốn xem thật" |
| **Trigger** | Khách chủ động, sau khi đọc phần *Vì sao Nera gợi ý* và so sánh vài căn. Không do notification đẩy |
| **Effort** | Cao. Phải chọn căn, chọn khung giờ, và cam kết bỏ ra khoảng nửa buổi gồm di chuyển và xem nhà |
| **Value timing** | Trễ và phụ thuộc người khác. Value thật xảy ra khi sale xác nhận và buổi xem diễn ra, thường sau 1–3 ngày |
| **State** | Sau action, hệ thống giữ: bản ghi yêu cầu (căn, khung giờ, sale), căn được đánh dấu đã quan tâm, và hồ sơ nhu cầu được cập nhật |
| **Dependency** | Phụ thuộc sale xác nhận, phụ thuộc trạng thái căn còn cho xem, phụ thuộc khung giờ trống |
| **Repeat condition** | Khách đi xem xong thấy chưa ưng và vẫn chưa mua được nhà. Chừng nào chu kỳ tìm nhà chưa kết thúc, hành vi còn lý do lặp lại |

### Kết luận cadence

**Dạng hành vi:** theo dự án.

> Đối với **người mua nhà để ở lần đầu**, core action **gửi yêu cầu đặt lịch xem nhà** thường xuất hiện **2–6 lần trong một chu kỳ tìm nhà kéo dài 4–10 tuần** vì **mua nhà là quyết định lớn, khách cần đi xem vài căn để so sánh trước khi chốt, và mỗi buổi xem tốn nửa ngày nên không thể dồn dập**. Do đó, nhịp đo phù hợp là **theo tuần trong chu kỳ tìm nhà** ở cấp **người dùng**.

**Vì sao không đo daily:** Không ai đi xem nhà mỗi ngày. Ép DAU lên sản phẩm này sẽ tạo áp lực sai: đội sẽ tìm cách kéo khách mở app hằng ngày bằng thông báo, trong khi hành vi tạo giá trị thật lại thưa theo bản chất.

**Về câu hỏi frequency cao có nghĩa là value cao không:** Không, và với Nera là ngược lại. Nếu Nera hiểu đúng nhu cầu ngay từ đầu, khách cần **ít** lượt đi xem hơn để tìm được nhà. Chu kỳ tìm nhà **ngắn lại** mới là tín hiệu tốt, không phải số lượt tăng lên. Điều này được xử lý bằng leading indicator ở mục 03 thay vì đưa vào NSM.

*Ghi chú: khoảng 2–6 lần và 4–10 tuần là giả định từ đặc thù hành vi mua nhà, chưa có dữ liệu người dùng thật của Nera để xác nhận. Cần đo lại sau khi có cohort đầu tiên.*

---

## 03 — Metric System

### Activation metric

| Thành phần | Định nghĩa |
|---|---|
| **Start event** | `search_session_started` — khách gửi tin nhắn đầu tiên có chứa nhu cầu tìm nhà |
| **Activation event** | `booking_requested` lần đầu tiên của người dùng đó |
| **Time window** | 7 ngày kể từ start event |

**Vì sao không dùng "đăng nhập" hay "xem hết hướng dẫn":** Cả hai đều xảy ra trước khi khách chạm tới bất kỳ giá trị nào. Một người đăng nhập rồi bỏ đi vẫn được tính là activated, con số sẽ đẹp mà vô nghĩa.

**Vì sao 7 ngày:** Nếu sau một tuần trò chuyện mà khách vẫn không tìm được căn nào đáng đi xem, đó là thất bại của sản phẩm chứ không phải khách chưa sẵn sàng.

### Engagement metric

Chọn hai góc:

**Frequency** — số yêu cầu đặt lịch trên mỗi chu kỳ tìm nhà đang hoạt động.

**Depth** — tỉ lệ yêu cầu đặt lịch đi tới buổi xem hoàn tất. Đo mức độ "thật" của mỗi hành vi: yêu cầu được gửi ra rồi bỏ đó khác hẳn yêu cầu dẫn tới một buổi xem có người đến.

### North Star Metric

> **Số buổi xem nhà hoàn tất** *(unit of value)* **có khách đến đúng hẹn và sale xác nhận đã diễn ra** *(quality threshold)*, tính **trên mỗi chu kỳ tìm nhà đang hoạt động** *(frequency)*.

Ba thành phần đủ. Không dùng doanh thu vì Nera chưa có mô hình thu tiền, và không dùng lượt mở app vì mở app không phải giá trị.

### Leading indicators

| Chỉ số | Tính từ event nào | Vì sao tin nó dự báo được core action lặp lại |
|---|---|---|
| Tỉ lệ phiên có ít nhất một căn được lưu | `property_saved` / `search_session_started` | Lưu căn là bước ngay trước khi đặt lịch. Phiên không ai lưu gì nghĩa là gợi ý trượt hết, sẽ không có yêu cầu nào phát sinh |
| Tỉ lệ phiên đạt đủ tiêu chí trong 5 lượt trao đổi đầu | `criteria_captured` / `search_session_started` | Hồ sơ càng sớm đủ, truy vấn càng khớp. Đây là biến đội tác động trực tiếp được bằng cách cải thiện phần hỏi lại |
| Tỉ lệ căn bị loại trên tổng số căn hiển thị, theo từng vòng | `property_rejected` / `recommendation_shown` | Nếu vòng lặp ở mục 05 hoạt động, tỉ lệ này phải giảm dần qua các vòng. Đây chính là phép thử cho metric hypothesis |

### Counter-metrics

**Tỉ lệ lịch đã xác nhận nhưng không diễn ra.**
Tính bằng số `booking_confirmed` không có `viewing_completed` tương ứng trong vòng 48 giờ sau khung giờ hẹn, chia cho tổng `booking_confirmed`. Không cần event mới, suy ra từ hai event đã có.

Nếu số yêu cầu đặt lịch tăng mà tỉ lệ này cũng tăng, Nera đang đẩy khách đặt lịch bừa thay vì gợi ý đúng. Sale mất thời gian, sản phẩm mất uy tín.

**Tỉ lệ phản hồi mang nhãn `fallback`.**
Tính từ thuộc tính `ai_mode` gắn trên mỗi `recommendation_shown`. Hệ thống hiện đã trả về nhãn này trên mọi response nên dữ liệu có sẵn, không cần thêm gì.

Khi nhà cung cấp mô hình lỗi, hệ thống trả lời theo luật cứng. Chỉ số này tăng nghĩa là chất lượng hội thoại đang xuống dù các con số hành vi có thể chưa phản ánh ngay.

---

## 04 — Retention Definition

| Thành phần | Định nghĩa |
|---|---|
| **Unit** | Người dùng cá nhân (`user_id`) |
| **Cohort entry** | Tuần xảy ra `search_session_started` đầu tiên |
| **Return event** | `booking_requested` — core action lặp lại |
| **Window** | Theo tuần, đo W1 đến W8 kể từ tuần vào cohort |
| **Threshold** | Ít nhất một lần trong window |
| **Segment** | Chỉ tính người dùng đã activated, tức đã có ít nhất một `booking_requested` trong 7 ngày đầu. Tách thêm hai nhánh theo thuộc tính `resumed_from_memory` để so người có được nhắc lại nhu cầu cũ với người không |

**Khớp với cadence ở Phase 2:** Cadence kết luận hành vi theo dự án, chu kỳ 4–10 tuần. Nên đo theo tuần và kéo dài tới W8 để phủ hết một chu kỳ. Đo D1 hoặc D7 sẽ sai vì không ai đặt lịch xem nhà hai ngày liên tiếp.

**So với ba mốc:**

- **Natural cycle:** đường retention nên giữ mức có ý nghĩa tới W6–W8 rồi rơi. Rơi ở W8 là tín hiệu tốt, nghĩa là khách đã mua được nhà và chu kỳ kết thúc tự nhiên. Rơi ở W2 mới là vấn đề.
- **Cohort đúng segment:** tách riêng người mua để ở và người đầu tư. Hai nhóm có chu kỳ khác hẳn nhau, gộp lại thì đường cong vô nghĩa.
- **Benchmark category:** so với sản phẩm giao dịch giá trị lớn theo dự án như tuyển dụng hay du lịch, không so với mạng xã hội hay ứng dụng đọc tin.

**Lưu ý về việc rời bỏ:** Với Nera, khách ngừng dùng vì đã mua được nhà là **thành công**, không phải churn. Vì vậy retention phải đọc kèm lý do kết thúc chu kỳ, không đọc trơ con số.

---

## 05 — Product Loop

**Loại loop:** project loop.

### Hai chu kỳ

```
Chu kỳ 1
Natural trigger   Khách chưa tìm được nhà, buổi tối rảnh, mở Nera
Core action       Mô tả nhu cầu bằng lời, xem gợi ý kèm lý do, gửi yêu cầu đặt lịch một căn
Immediate value   Có lịch hẹn cụ thể với sale thật, không phải gọi điện dò từng nơi
Saved state       Hồ sơ nhu cầu (khu vực, ngân sách, số phòng, ưu tiên mềm) + danh sách căn đã lưu và đã loại

Chu kỳ 2
Next trigger      Đi xem xong thấy chưa ưng, vẫn cần nhà, quay lại Nera
Core action       Nera nhắc đúng nhu cầu cũ, loại các căn đã xem, đưa gợi ý mới sát hơn, gửi yêu cầu đặt lịch căn tiếp theo
Repeat value      Danh sách lần này sát hơn lần trước vì hệ thống biết thêm khách đã loại những gì
```

### Metric hypothesis

> Nếu loop này hoạt động, metric **số lượt xem gợi ý cần thiết để dẫn tới một yêu cầu đặt lịch** sẽ **giảm dần qua từng chu kỳ quay lại** trong **khoảng 3–6 tuần**, vì **hệ thống loại dần các căn khách đã từ chối và hồ sơ nhu cầu đầy dần lên sau mỗi vòng**.

### Reason to return nếu bỏ hết notification

Khách vẫn chưa có nhà để ở. Đó là lý do quay lại nằm ngoài sản phẩm, không do sản phẩm tạo ra. Nera chỉ cần làm cho việc quay lại đỡ tốn công hơn nơi khác, bằng cách nhớ nhu cầu cũ thay vì bắt khai lại.

Đây là điểm phân biệt loop thật với loop dựa vào trigger bên ngoài: bỏ hết thông báo đi, vòng lặp vẫn chạy.

---

## 06 — Tracking

### Bảng event

| Tên event | Ý nghĩa | Thời điểm ghi nhận | Metric sử dụng |
|---|---|---|---|
| `search_session_started` | Khách bắt đầu một phiên tìm nhà mới | Khi tin nhắn đầu tiên có chứa tiêu chí được xử lý xong và hồ sơ nhu cầu được khởi tạo | Start event của activation; cohort entry của retention; mẫu số của leading 1 và 2 |
| `criteria_captured` | Hệ thống trích xuất được tối thiểu khu vực và ngân sách | Khi hồ sơ nhu cầu được cập nhật thành công với cả hai trường | Leading indicator 2 |
| `recommendation_shown` | Danh sách căn gợi ý được hiển thị cho khách | Khi phản hồi chứa ít nhất một thẻ bất động sản render xong trên màn hình | Mẫu số của leading 3; counter-metric tỉ lệ `fallback` (qua thuộc tính `ai_mode`) |
| `property_saved` | Khách lưu một căn vào danh sách quan tâm | Khi bản ghi lưu được tạo thành công | Leading indicator 1 |
| `property_rejected` | Khách đánh dấu một căn không phù hợp | Khi bản ghi loại trừ được tạo thành công | Leading indicator 3; phép thử cho metric hypothesis ở mục 05 |
| `booking_requested` | **Core action.** Khách gửi yêu cầu đặt lịch xem một căn | Khi bản ghi yêu cầu được tạo với `property_id`, khung giờ, `sale_id`, và hệ thống trả về mã yêu cầu | Activation event; engagement frequency; return event của retention |
| `booking_confirmed` | Sale xác nhận lịch hẹn | Khi trạng thái yêu cầu chuyển từ chờ duyệt sang đã đặt | Engagement depth; bước trung gian của NSM; mẫu số của counter-metric 1 |
| `viewing_completed` | **Core value event.** Buổi xem nhà đã diễn ra | Khi sale đánh dấu buổi xem đã hoàn tất | North Star Metric; tử số của engagement depth và counter-metric 1 |

### Thuộc tính bắt buộc

Ba thuộc tính dưới đây cho phép tính thêm metric mà không phải thêm event, giữ bảng trong giới hạn 8 event.

| Event | Thuộc tính | Dùng để |
|---|---|---|
| `search_session_started` | `resumed_from_memory` (true/false) | Tách cohort retention theo việc khách có được nhắc lại nhu cầu cũ hay không |
| `recommendation_shown` | `ai_mode` (`llm_grounded` / `llm_direct` / `llm_intent` / `fallback`) | Tính counter-metric tỉ lệ `fallback`. Hệ thống hiện đã trả nhãn này trên mọi response |
| `recommendation_shown` | `round_index` (số vòng gợi ý trong cùng chu kỳ) | Đo tỉ lệ căn bị loại giảm dần qua từng vòng, tức phép thử cho metric hypothesis |

Mọi event đều mang `user_id`, `session_id` và `timestamp` theo múi giờ Asia/Ho_Chi_Minh. Loại trừ tài khoản nội bộ và tài khoản seed demo khỏi mọi phép tính.

### Tiêu chí nghiệm thu

**1. Event chỉ ghi khi hành vi thật sự hoàn tất.**
Với mỗi cặp `user_id` và `property_id`, hệ thống chỉ ghi `booking_requested` khi bản ghi yêu cầu được tạo thành công và mã yêu cầu đã trả về cho khách. Trường hợp khách bấm gửi nhưng khung giờ vừa bị người khác lấy, hoặc backend trả lỗi, thì không ghi event.

**2. Tải lại hoặc thử lại không ghi trùng.**
Với mỗi `request_id`, `booking_requested` và `booking_confirmed` chỉ được ghi đúng một lần cho mỗi lần chuyển trạng thái. Tải lại trang xác nhận, bấm nút quay lại, hay hệ thống tự thử lại đều không được sinh thêm event cho cùng lần chuyển đó.

**3. Không suy ra value từ thời gian trôi qua.**
`viewing_completed` chỉ được ghi khi sale chủ động đánh dấu buổi xem đã diễn ra. Không tự động ghi khi qua giờ hẹn, vì khách hoàn toàn có thể không đến.

### Ghi chú triển khai

Ba event `property_saved`, `property_rejected`, `booking_requested` tương ứng với các nút đã có sẵn trên giao diện hiện tại: *Lưu*, *Không phù hợp*, *Đặt lịch xem*. Cần kiểm tra lại `property_rejected` hiện có ghi bản ghi vào cơ sở dữ liệu hay chỉ ẩn thẻ trên giao diện.

Event `viewing_completed` **chưa có trong hệ thống**, cần bổ sung thao tác cho sale đánh dấu buổi xem đã diễn ra. Không có event này thì North Star và counter-metric 1 đều chưa tính được, chỉ đo tới `booking_confirmed`. Đây là việc cần làm trước khi thêm bất kỳ tính năng mới nào.

---

## Phase 5 — Tự soi lỗi

| Câu hỏi | Kết quả | Căn cứ |
|---|:--:|---|
| Core action không phải thao tác giao diện hay output hệ thống? | Đạt | `booking_requested` là cam kết thời gian thật của khách, không phải nút bấm hay kết quả AI sinh ra |
| Activation không phải "xem hết hướng dẫn" hay "đăng nhập"? | Đạt | Activation là `booking_requested` lần đầu trong 7 ngày, tức khách đã chạm value |
| Frequency không cao hơn nhu cầu thật? | Đạt | Cadence kết luận theo dự án, 2–6 lần trong 4–10 tuần. Không dùng DAU |
| Loop có reason to return ngoài notification? | Đạt | Khách vẫn chưa có nhà để ở. Lý do nằm ngoài sản phẩm, bỏ hết thông báo vòng lặp vẫn chạy |
| Retention không dùng chung một window cho mọi cadence? | Đạt | Window theo tuần, đo W1–W8, suy ra từ kết luận cadence ở mục 02. Không dùng D1/D7 |
| Mọi event đều map về một metric? | Đạt | Cả 8 event đều có cột "Metric sử dụng" trỏ về một chỉ số cụ thể ở mục 03 |
| Metric nào cũng có event để tính nó? | Đạt | Xem bảng đối chiếu dưới |

### Đối chiếu metric với nguồn dữ liệu

| Metric | Tính từ |
|---|---|
| Activation rate | `search_session_started` → `booking_requested` trong 7 ngày |
| Engagement frequency | Đếm `booking_requested` theo chu kỳ |
| Engagement depth | `viewing_completed` / `booking_requested` |
| North Star | Đếm `viewing_completed` theo chu kỳ đang hoạt động |
| Leading 1 | `property_saved` / `search_session_started` |
| Leading 2 | `criteria_captured` / `search_session_started` |
| Leading 3 | `property_rejected` / `recommendation_shown`, cắt theo `round_index` |
| Counter 1 | `booking_confirmed` không có `viewing_completed` trong 48 giờ |
| Counter 2 | Tỉ lệ `recommendation_shown` có `ai_mode = fallback` |
| Retention W1–W8 | `search_session_started` (vào cohort) → `booking_requested` (quay lại) |

Không có metric nào thiếu nguồn, không có event nào thừa.

---

## 07 — Revision

Core action và cadence giữ nguyên từ đầu. Hai thay đổi ở tầng metric và tracking, đều phát sinh khi chạy Phase 5.

**Thay đổi 1 — bỏ leading indicator không tính được.**
Bản đầu đặt leading indicator là "tỉ lệ khách quay lại mà không phải khai lại tiêu chí". Ý đúng nhưng không có event nào ghi lại việc đó, vi phạm quy tắc mọi metric phải có event để tính. Thay bằng "tỉ lệ căn bị loại trên tổng số căn hiển thị theo từng vòng", tính từ `property_rejected` và `recommendation_shown`. Chỉ số mới còn tốt hơn ở chỗ nó chính là phép thử trực tiếp cho metric hypothesis ở mục 05. Phần trí nhớ được giữ lại dưới dạng thuộc tính `resumed_from_memory` để tách cohort retention.

**Thay đổi 2 — bỏ event `property_rejected` khỏi diện "không map metric".**
Bản đầu ghi công dụng của event này là "chất lượng gợi ý", nghe hợp lý nhưng không trỏ về chỉ số cụ thể nào. Sau khi đổi leading indicator 3, event này có metric rõ ràng.

**Cách xử lý giới hạn 8 event.**
Ba metric còn thiếu nguồn ban đầu là tỉ lệ nhắc lại nhu cầu, tỉ lệ hủy lịch và tỉ lệ `fallback`. Nếu thêm ba event mới thì bảng lên 11, vượt giới hạn. Cách xử lý: hai trong ba dùng thuộc tính gắn trên event đã có (`resumed_from_memory`, `ai_mode`), cái còn lại suy ra từ việc `booking_confirmed` không có `viewing_completed` tương ứng. Bảng giữ đúng 8 event.

**Một điểm từng cân nhắc rồi loại:** chọn `property_saved` làm core action vì tần suất cao hơn. Loại vì lưu căn không tốn gì của khách nên là tín hiệu yếu, và có thể tăng chỉ vì khách phân vân chứ không phải vì gợi ý tốt. `booking_requested` bắt khách trả giá bằng thời gian thật, nên gần core value hơn.
