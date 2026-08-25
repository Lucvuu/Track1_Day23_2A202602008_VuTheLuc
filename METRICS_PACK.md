# Metrics Pack — Nera

Học viên: Vũ Thế Lực · MHV 2A202602008
Dự án: Nera — trợ lý AI tìm nhà và đặt lịch xem nhà (nerahome.space)

---

## 00 — Dự án, persona, core job

**Dự án:** Nera, trợ lý AI giúp người tìm nhà mô tả nhu cầu bằng lời nói tự nhiên, nhận gợi ý bất động sản có thật kèm lý do phù hợp, và đặt lịch đi xem với nhân viên sale.

**Persona:** Người mua nhà để ở lần đầu, 25–35 tuổi, đang đi làm tại Hà Nội, ngân sách 2–6 tỷ. Tìm nhà vào buổi tối và cuối tuần, không rành thuật ngữ bất động sản.

**Core job:** *"Tôi muốn tìm được vài căn đáng để bỏ một buổi đi xem thật, mà không phải đọc hàng trăm tin đăng và không phải kể lại nhu cầu từ đầu mỗi lần quay lại."*

**Use case chọn phân tích:** Từ lúc mô tả nhu cầu tới lúc có lịch hẹn xem nhà. Không phân tích phần quản lý của sale hay admin.

### Đơn vị đo: chu kỳ tìm nhà

Nhiều chỉ số dưới đây tính trên **chu kỳ tìm nhà** chứ không phải trên phiên. Một khách chat ba tối liên tiếp là ba phiên nhưng vẫn chỉ là một chu kỳ. Định nghĩa để đếm được:

| Mốc | Quy tắc |
|---|---|
| **Mở chu kỳ** | `search_session_started` xảy ra khi người dùng đó chưa có chu kỳ nào đang mở |
| **Đang hoạt động** | Có ít nhất một event bất kỳ trong 21 ngày gần nhất |
| **Đóng vì thành công** | `viewing_closed` mang `outcome = closed_deal`, tức sale ghi nhận khách đã chốt được nhà |
| **Đóng vì bỏ dở** | 30 ngày liên tiếp không có event nào |
| **Chu kỳ mới** | Phiên tìm nhà xảy ra sau khi chu kỳ trước đã đóng |

Hai kiểu đóng phải tách nhau khi đọc số. Đóng vì thành công nghĩa là khách đã có nhà, đó là kết quả tốt. Đóng vì bỏ dở mới là mất khách.

Ngưỡng 21 ngày và 30 ngày là giả định từ nhịp đi xem nhà, chưa có dữ liệu Nera để xác nhận, cần hiệu chỉnh sau cohort đầu tiên.

---

## 01 — Core Action Card

### Phân biệt bốn khái niệm

| Khái niệm | Câu trả lời cho Nera |
|---|---|
| Core job | Tìm được căn đáng bỏ công đi xem, không phải khai lại nhu cầu |
| Core action | Gửi yêu cầu đặt lịch xem một căn cụ thể |
| Core value | Có một buổi hẹn thật với sale thật, để đánh giá căn nhà bằng mắt mình |
| Core value event | `viewing_closed` mang `close_status = attended` — buổi xem đã diễn ra thật |

Core action và core value event **không trùng nhau**. Gửi yêu cầu là hành vi của khách; buổi xem hoàn tất mới là bằng chứng value đã xảy ra, và nó phụ thuộc vào sale đóng lịch hẹn. Vì bằng chứng value nằm trong tay người ngoài đội sản phẩm, mục 03 có thêm một chỉ số canh độ phủ đóng lịch, để không đọc nhầm sale quên ghi thành khách không đến.

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
| **Evidence of value** | Sale xác nhận lịch, rồi đóng lịch hẹn với `close_status = attended` |
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

**Dạng hành vi:** theo dự án. Chu kỳ tìm nhà được định nghĩa để đếm ở mục 00.

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

*Ghi chú: 7 ngày là giả định, cùng loại với khoảng 2–6 lần và 4–10 tuần ở mục 02. Nó khá căng với một quyết định 2–6 tỷ và có thể phải nới lên 14 ngày. Chốt lại sau khi đo phân bố thời gian tới lần đặt lịch đầu tiên của cohort đầu.*

### Engagement metric

Chọn hai góc:

**Frequency** — số yêu cầu đặt lịch trên mỗi chu kỳ tìm nhà đang hoạt động.

**Depth** — tỉ lệ yêu cầu đặt lịch đi tới một buổi xem có khách đến, tức `viewing_closed` mang `close_status = attended`. Đo mức độ "thật" của mỗi hành vi: yêu cầu được gửi ra rồi bỏ đó khác hẳn yêu cầu dẫn tới một buổi xem có người đến.

### North Star Metric

> **Số buổi xem nhà hoàn tất** *(unit of value)* **được sale đóng với `close_status = attended`, tức khách có mặt** *(quality threshold)*, tính **trên mỗi chu kỳ tìm nhà đang hoạt động theo định nghĩa ở mục 00** *(frequency)*.

Ba thành phần đủ. Không dùng doanh thu vì Nera chưa có mô hình thu tiền, và không dùng lượt mở app vì mở app không phải giá trị.

### Leading indicators

| Chỉ số | Tính từ event nào | Vì sao tin nó dự báo được core action lặp lại |
|---|---|---|
| Tỉ lệ phiên có ít nhất một căn được lưu | `property_saved` / `search_session_started` | Lưu căn là bước ngay trước khi đặt lịch. Phiên không ai lưu gì nghĩa là gợi ý trượt hết, sẽ không có yêu cầu nào phát sinh |
| Tỉ lệ phiên đạt đủ tiêu chí trong 5 lượt trao đổi đầu | `criteria_captured` có `turn_index <= 5` / `search_session_started` | Hồ sơ càng sớm đủ, truy vấn càng khớp. Đây là biến đội tác động trực tiếp được bằng cách cải thiện phần hỏi lại |
| Tỉ lệ căn bị loại trên tổng số căn hiển thị, theo từng vòng | `property_rejected` / `recommendation_shown`, chỉ tính các vòng có ít nhất một `property_saved` hoặc `property_rejected` | Nếu vòng lặp ở mục 05 hoạt động, tỉ lệ này phải giảm dần qua các vòng. Đây chính là phép thử cho metric hypothesis |

**Bẫy đọc ngược của leading 3.** Tỉ lệ căn bị loại giảm có hai nguyên nhân: gợi ý sát hơn thật, hoặc khách chán không bấm *Không phù hợp* nữa. Cả hai đều làm tử số giảm và nhìn giống hệt nhau. Vì vậy mẫu số chỉ lấy các vòng khách còn phản hồi, và chỉ số này luôn đọc kèm leading 1. Nếu tỉ lệ loại giảm mà tỉ lệ lưu căn cũng giảm thì đó là khách bỏ đi, không phải gợi ý tốt lên.

### Counter-metrics

**Counter 1 — tỉ lệ lịch đã xác nhận nhưng khách không đến.**
Số `viewing_closed` mang `close_status = no_show` chia cho tổng số `viewing_closed` mang `attended` hoặc `no_show`. Lịch khách hủy trước giờ hẹn (`close_status = cancelled`) nằm ngoài cả tử số lẫn mẫu số, vì hủy sớm không làm sale mất buổi.

Nếu số yêu cầu đặt lịch tăng mà tỉ lệ này cũng tăng, Nera đang đẩy khách đặt lịch bừa thay vì gợi ý đúng. Sale mất thời gian, sản phẩm mất uy tín.

**Counter 2 — độ phủ đóng lịch hẹn.**
Số `booking_confirmed` đã qua khung giờ hẹn quá 48 giờ mà không có `viewing_closed` nào tương ứng, chia cho tổng `booking_confirmed` đã qua giờ hẹn. Đây là chỉ số chất lượng dữ liệu, không phải chỉ số sản phẩm.

Chỉ số này tồn tại vì bằng chứng value của Nera nằm trong tay sale chứ không nằm trong hành vi của khách. Sale quên đóng lịch sẽ làm North Star tụt và counter 1 nhìn như tăng, dẫn tới kết luận sai là khách đặt bừa trong khi thật ra chỉ là thiếu dữ liệu. Chừng nào chỉ số này còn cao, mọi con số phía dưới đều phải đọc dè chừng.

**Counter 3 — tỉ lệ phản hồi mang nhãn `fallback`.**
Tính từ thuộc tính `ai_mode` gắn trên mỗi `recommendation_shown`. Hệ thống hiện đã trả về nhãn này trên mọi response nên dữ liệu có sẵn, không cần thêm gì.

Khi nhà cung cấp mô hình lỗi, hệ thống trả lời theo luật cứng. Chỉ số này tăng nghĩa là chất lượng hội thoại đang xuống dù các con số hành vi có thể chưa phản ánh ngay.

---

## 04 — Retention Definition

| Thành phần | Định nghĩa |
|---|---|
| **Unit** | Người dùng cá nhân (`user_id`) |
| **Cohort entry** | Tuần xảy ra `booking_requested` đầu tiên, tức tuần activation |
| **Return event** | `booking_requested` tiếp theo, không tính lần đã dùng để activation |
| **Window** | Theo tuần, đo W1 đến W8 kể từ tuần vào cohort |
| **Threshold** | Ít nhất một lần trong window |
| **Segment** | Chỉ tính người dùng đã activated, tức đã có ít nhất một `booking_requested` trong 7 ngày đầu. Tách thêm hai nhánh theo thuộc tính `resumed_from_memory` để so người có được nhắc lại nhu cầu cũ với người không |

**Vì sao lấy tuần activation làm mốc vào cohort chứ không lấy phiên đầu tiên:** cohort này đã lọc chỉ còn người activated. Nếu vào cohort từ phiên đầu mà return event vẫn là `booking_requested`, thì chính lần đặt lịch dùng để activation sẽ bị đếm lại thành retention của W1, và đường cong đẹp lên một cách giả tạo ngay ở điểm đầu.

**Khớp với cadence ở Phase 2:** Cadence kết luận hành vi theo dự án, chu kỳ 4–10 tuần. Nên đo theo tuần và kéo dài tới W8 để phủ hết một chu kỳ. Đo D1 hoặc D7 sẽ sai vì không ai đặt lịch xem nhà hai ngày liên tiếp.

**So với ba mốc:**

- **Natural cycle:** đường retention nên giữ mức có ý nghĩa tới W6–W8 rồi rơi. Rơi ở W8 là tín hiệu tốt, nghĩa là khách đã mua được nhà và chu kỳ kết thúc tự nhiên. Rơi ở W2 mới là vấn đề.
- **Cohort đúng segment:** tách riêng người mua để ở và người đầu tư. Hai nhóm có chu kỳ khác hẳn nhau, gộp lại thì đường cong vô nghĩa.
- **Benchmark category:** so với sản phẩm giao dịch giá trị lớn theo dự án như tuyển dụng hay du lịch, không so với mạng xã hội hay ứng dụng đọc tin.

**Lưu ý về việc rời bỏ:** Với Nera, khách ngừng dùng vì đã mua được nhà là **thành công**, không phải churn. Vì vậy retention phải đọc kèm lý do kết thúc chu kỳ, không đọc trơ con số. Lý do đó lấy từ thuộc tính `outcome` trên `viewing_closed`: chu kỳ đóng với `closed_deal` được tách khỏi đường cong, phần rơi còn lại mới là mất khách thật.

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

> Nếu loop này hoạt động, metric **leading indicator 3 — tỉ lệ căn bị loại trên tổng số căn hiển thị** (mục 03) sẽ **giảm dần theo `round_index`, tức vòng gợi ý sau thấp hơn vòng trước** trong **khoảng 3–6 tuần của một chu kỳ tìm nhà**, vì **hệ thống loại dần các căn khách đã từ chối và hồ sơ nhu cầu đầy dần lên sau mỗi vòng**.

Chỉ số này trỏ thẳng về một metric đã định nghĩa ở Phase 3, tính từ `property_rejected` chia cho `recommendation_shown`, cắt theo thuộc tính `round_index`, và chỉ lấy các vòng khách còn phản hồi theo đúng bẫy đọc ngược đã nêu ở mục 03.

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
| `recommendation_shown` | Danh sách căn gợi ý được hiển thị cho khách | Khi phản hồi chứa ít nhất một thẻ bất động sản render xong trên màn hình | Mẫu số của leading 3; counter 3 tỉ lệ `fallback` (qua thuộc tính `ai_mode`) |
| `property_saved` | Khách lưu một căn vào danh sách quan tâm | Khi bản ghi lưu được tạo thành công | Leading indicator 1 |
| `property_rejected` | Khách đánh dấu một căn không phù hợp | Khi bản ghi loại trừ được tạo thành công | Leading indicator 3; phép thử cho metric hypothesis ở mục 05 |
| `booking_requested` | **Core action.** Khách gửi yêu cầu đặt lịch xem một căn | Khi bản ghi yêu cầu được tạo với `property_id`, khung giờ, `sale_id`, và hệ thống trả về mã yêu cầu | Activation event; engagement frequency; return event của retention |
| `booking_confirmed` | Sale xác nhận lịch hẹn | Khi trạng thái yêu cầu chuyển từ chờ duyệt sang đã đặt | Bước trung gian của NSM; mẫu số của counter 2 |
| `viewing_closed` | **Core value event** khi `close_status = attended`. Sale đóng lịch hẹn và ghi lại chuyện gì đã xảy ra | Khi sale chủ động đóng lịch hẹn và chọn một trong ba trạng thái: khách đến, khách không đến, khách hủy trước giờ | North Star Metric; tử số của engagement depth; counter 1; quy tắc đóng chu kỳ ở mục 00 |

### Thuộc tính bắt buộc

Các thuộc tính dưới đây cho phép tính thêm metric mà không phải thêm event, giữ bảng trong giới hạn 8 event.

| Event | Thuộc tính | Dùng để |
|---|---|---|
| `search_session_started` | `resumed_from_memory` (true/false) | Tách cohort retention theo việc khách có được nhắc lại nhu cầu cũ hay không |
| `criteria_captured` | `turn_index` (lượt trao đổi thứ mấy trong phiên) | Tính leading indicator 2, vốn định nghĩa là đủ tiêu chí trong 5 lượt đầu. Không có thuộc tính này thì vế "5 lượt" không tính được |
| `recommendation_shown` | `ai_mode` (`llm_grounded` / `llm_direct` / `llm_intent` / `fallback`) | Tính counter 3 tỉ lệ `fallback`. Hệ thống hiện đã trả nhãn này trên mọi response |
| `recommendation_shown` | `round_index` (số vòng gợi ý trong cùng chu kỳ) | Đo tỉ lệ căn bị loại giảm dần qua từng vòng, tức phép thử cho metric hypothesis |
| `viewing_closed` | `close_status` (`attended` / `no_show` / `cancelled`) | Tách ba chuyện khác hẳn nhau mà một event "buổi xem đã diễn ra" sẽ gộp làm một: khách đến, khách không đến, khách hủy trước giờ |
| `viewing_closed` | `outcome` (`still_searching` / `closed_deal`), chỉ ghi khi `close_status = attended` | Đóng chu kỳ tìm nhà ở mục 00 và tách phần rơi thành công khỏi đường retention |

Mọi event đều mang `user_id`, `session_id`, `cycle_id` và `timestamp` theo múi giờ Asia/Ho_Chi_Minh. `cycle_id` do backend gán theo quy tắc mở và đóng chu kỳ ở mục 00, nhờ đó mọi chỉ số tính "trên mỗi chu kỳ" đều group được bằng một trường có sẵn thay vì suy ngược từ dấu thời gian mỗi lần chạy truy vấn. Loại trừ tài khoản nội bộ và tài khoản seed demo khỏi mọi phép tính.

### Tiêu chí nghiệm thu

**1. Event chỉ ghi khi hành vi thật sự hoàn tất.**
Với mỗi cặp `user_id` và `property_id`, hệ thống chỉ ghi `booking_requested` khi bản ghi yêu cầu được tạo thành công và mã yêu cầu đã trả về cho khách. Trường hợp khách bấm gửi nhưng khung giờ vừa bị người khác lấy, hoặc backend trả lỗi, thì không ghi event.

**2. Tải lại hoặc thử lại không ghi trùng.**
Với mỗi `request_id`, `booking_requested` và `booking_confirmed` chỉ được ghi đúng một lần cho mỗi lần chuyển trạng thái. Tải lại trang xác nhận, bấm nút quay lại, hay hệ thống tự thử lại đều không được sinh thêm event cho cùng lần chuyển đó.

**3. Không suy ra value từ thời gian trôi qua.**
`viewing_closed` chỉ được ghi khi sale chủ động đóng lịch hẹn và chọn `close_status`. Không tự động ghi khi qua giờ hẹn, vì khách hoàn toàn có thể không đến.

**4. Thao tác đóng lịch phải nằm trên đường đi sẵn có của sale, không được là việc làm thêm.**
Tiêu chí 3 đặt bằng chứng value vào tay sale, nên việc đóng lịch phải gắn vào bước sale vốn đã phải làm là ghi kết quả buổi xem trước khi chuyển trạng thái lead. Lịch hẹn chưa đóng thì lead không đóng được. Nếu để đóng lịch thành một nút rời không ảnh hưởng gì tới công việc của sale, dữ liệu North Star sẽ mục dần, counter 2 sẽ leo, và mọi chỉ số phía dưới mất giá trị.

### Ghi chú triển khai

Ba event `property_saved`, `property_rejected`, `booking_requested` tương ứng với các nút đã có sẵn trên giao diện hiện tại: *Lưu*, *Không phù hợp*, *Đặt lịch xem*. Cần kiểm tra lại `property_rejected` hiện có ghi bản ghi vào cơ sở dữ liệu hay chỉ ẩn thẻ trên giao diện.

Event `viewing_closed` **chưa có trong hệ thống**, cần bổ sung màn hình cho sale đóng lịch hẹn kèm `close_status` và `outcome`. Không có event này thì North Star, engagement depth, counter 1 và quy tắc đóng chu kỳ ở mục 00 đều chưa tính được, chỉ đo tới `booking_confirmed`. Đây là việc cần làm trước khi thêm bất kỳ tính năng mới nào.

Trường `cycle_id` cũng chưa có. Backend đang định danh theo phiên chứ không theo chu kỳ, nên trước mắt mọi chỉ số "trên mỗi chu kỳ" phải suy từ dấu thời gian theo quy tắc ở mục 00. Gán `cycle_id` ngay từ đầu rẻ hơn nhiều so với suy ngược sau này.

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
| Metric nào cũng có event **và thuộc tính** để tính nó? | Đạt sau khi sửa | Vòng tự soi đầu bỏ sót: leading 2 định nghĩa "đủ tiêu chí trong 5 lượt đầu" nhưng `criteria_captured` không mang số lượt, nên vế "5 lượt" không tính được. Đã thêm thuộc tính `turn_index` |
| Mọi đơn vị đếm trong metric đều có định nghĩa? | Đạt sau khi sửa | "Chu kỳ tìm nhà đang hoạt động" là mẫu số của NSM và engagement, nhưng ban đầu không mục nào nói nó mở khi nào, đóng khi nào, khác phiên ra sao. Đã bổ sung ở mục 00 kèm trường `cycle_id` |
| Có chỉ số nào đọc ngược được mà chưa đặt chốt chặn? | Đạt sau khi sửa | Leading 3 giảm cũng có thể vì khách bỏ ngang. Đã siết mẫu số về các vòng còn phản hồi và buộc đọc kèm leading 1 |
| Chỉ số phụ thuộc thao tác của người ngoài đội có được canh không? | Đạt sau khi sửa | North Star phụ thuộc sale đóng lịch. Đã thêm counter 2 đo độ phủ đóng lịch, và tiêu chí nghiệm thu 4 buộc thao tác này nằm trên đường đi sẵn có của sale |
| Retention có đếm trùng với activation không? | Đạt sau khi sửa | Cohort entry cũ là phiên đầu tiên trong khi return event chính là event activation, nên W1 đếm lại lần đặt lịch đã dùng để activation. Đã chuyển mốc vào cohort sang tuần activation |

### Đối chiếu metric với nguồn dữ liệu

| Metric | Tính từ |
|---|---|
| Activation rate | `search_session_started` → `booking_requested` trong 7 ngày |
| Engagement frequency | Đếm `booking_requested`, group theo `cycle_id` |
| Engagement depth | `viewing_closed` có `close_status = attended` / `booking_requested` |
| North Star | Đếm `viewing_closed` có `close_status = attended`, group theo `cycle_id` đang hoạt động |
| Leading 1 | `property_saved` / `search_session_started` |
| Leading 2 | `criteria_captured` có `turn_index <= 5` / `search_session_started` |
| Leading 3 | `property_rejected` / `recommendation_shown`, cắt theo `round_index`, chỉ lấy vòng còn phản hồi |
| Counter 1 | `close_status = no_show` / (`attended` + `no_show`) |
| Counter 2 | `booking_confirmed` đã qua giờ hẹn 48 giờ mà không có `viewing_closed` / tổng `booking_confirmed` đã qua giờ hẹn |
| Counter 3 | Tỉ lệ `recommendation_shown` có `ai_mode = fallback` |
| Retention W1–W8 | `booking_requested` đầu tiên (vào cohort) → `booking_requested` tiếp theo (quay lại) |
| Đóng chu kỳ | `viewing_closed` có `outcome = closed_deal`, hoặc 30 ngày không event |

Không có metric nào thiếu nguồn, không có event nào thừa. Bảng vẫn đúng 8 event: mọi thứ bổ sung ở vòng sửa này đều là thuộc tính gắn lên event đã có.

---

## 07 — Revision

Core action và cadence giữ nguyên từ đầu. Các thay đổi đều nằm ở tầng metric và tracking: hai cái đầu phát sinh khi chạy Phase 5, bốn cái sau phát sinh khi soi lại lần hai.

**Thay đổi 1 — bỏ leading indicator không tính được.**
Bản đầu đặt leading indicator là "tỉ lệ khách quay lại mà không phải khai lại tiêu chí". Ý đúng nhưng không có event nào ghi lại việc đó, vi phạm quy tắc mọi metric phải có event để tính. Thay bằng "tỉ lệ căn bị loại trên tổng số căn hiển thị theo từng vòng", tính từ `property_rejected` và `recommendation_shown`. Chỉ số mới còn tốt hơn ở chỗ nó chính là phép thử trực tiếp cho metric hypothesis ở mục 05. Phần trí nhớ được giữ lại dưới dạng thuộc tính `resumed_from_memory` để tách cohort retention.

**Thay đổi 2 — bỏ event `property_rejected` khỏi diện "không map metric".**
Bản đầu ghi công dụng của event này là "chất lượng gợi ý", nghe hợp lý nhưng không trỏ về chỉ số cụ thể nào. Sau khi đổi leading indicator 3, event này có metric rõ ràng.

**Thay đổi 3 — định nghĩa chu kỳ tìm nhà thành đơn vị đếm được.**
North Star và engagement frequency đều tính "trên mỗi chu kỳ tìm nhà đang hoạt động", nhưng bản đầu không mục nào nói chu kỳ mở khi nào, đóng khi nào, hay khác phiên ra sao. Mẫu số không có định nghĩa thì chỉ số không đếm được, dù event có đủ. Đã bổ sung quy tắc mở, đóng và ngưỡng hoạt động ở mục 00, kèm trường `cycle_id` gắn trên mọi event.

**Thay đổi 4 — `viewing_completed` thành `viewing_closed` kèm `close_status`.**
Bản đầu chỉ ghi event khi buổi xem đã diễn ra. Hệ quả là ba tình huống khác hẳn nhau đều hiện ra giống nhau, cùng là "không có event": khách không đến, khách hủy trước giờ, và sale quên ghi. Counter-metric no-show vì thế đo lẫn cả lỗi dữ liệu lẫn hành vi khách. Đã đổi thành sale đóng lịch hẹn với `close_status` ba trạng thái. North Star chỉ đếm `attended` nên định nghĩa core value không đổi, chỉ đo được sạch hơn.

**Thay đổi 5 — thêm counter đo độ phủ đóng lịch.**
Đi kèm thay đổi 4. Bằng chứng value của Nera nằm trong tay sale chứ không nằm trong hành vi khách, nên phải có chỉ số canh chính việc ghi nhận. Kèm theo là tiêu chí nghiệm thu 4, buộc thao tác đóng lịch nằm trên đường đi sẵn có của sale thay vì là một nút rời ai nhớ thì bấm.

**Thay đổi 6 — vá hai chỗ vòng tự soi đầu bỏ sót.**
Leading 2 định nghĩa "đủ tiêu chí trong 5 lượt đầu" nhưng `criteria_captured` không mang số lượt, đã thêm `turn_index`. Retention lấy phiên đầu tiên làm mốc vào cohort trong khi return event chính là event activation, khiến W1 đếm lại lần đặt lịch đã dùng để activation; đã chuyển mốc vào cohort sang tuần activation.

**Cách xử lý giới hạn 8 event.**
Ba metric còn thiếu nguồn ban đầu là tỉ lệ nhắc lại nhu cầu, tỉ lệ hủy lịch và tỉ lệ `fallback`. Nếu thêm ba event mới thì bảng lên 11, vượt giới hạn. Cách xử lý: hai trong ba dùng thuộc tính gắn trên event đã có (`resumed_from_memory`, `ai_mode`), cái còn lại nằm trong `close_status` của `viewing_closed`. Bốn thay đổi sau cũng theo đúng luật đó, mọi thứ thêm vào đều là thuộc tính. Bảng giữ đúng 8 event.

**Một điểm từng cân nhắc rồi loại:** chọn `property_saved` làm core action vì tần suất cao hơn. Loại vì lưu căn không tốn gì của khách nên là tín hiệu yếu, và có thể tăng chỉ vì khách phân vân chứ không phải vì gợi ý tốt. `booking_requested` bắt khách trả giá bằng thời gian thật, nên gần core value hơn.
