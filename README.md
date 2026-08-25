# Track1_Day23_2A202602008_VuTheLuc

**Họ tên:** Vũ Thế Lực
**MHV:** 2A202602008

**Dự án chọn làm:** Nera — trợ lý AI tìm nhà và đặt lịch xem nhà qua hội thoại tự nhiên. Sản phẩm của Team 046 LTD, đang chạy công khai tại https://www.nerahome.space/

**Use case phân tích:** từ lúc khách mô tả nhu cầu bằng lời tới lúc có lịch hẹn xem nhà. Không phân tích phần quản lý của sale và admin.

**Link tệp Metrics Pack:** `[DÁN LINK GOOGLE DOC ĐÃ CẤP QUYỀN XEM]`

---

## Điều tôi mang về áp dụng cho dự án thật

**Nera đang không có cách nào biết mình có hữu ích hay không.**

Trước bài lab, cách cả nhóm nói về tiến độ là "chat chạy được", "đặt lịch chạy được", "có trí nhớ". Toàn là mô tả tính năng. Không ai trả lời được câu: dấu hiệu nào cho thấy một người dùng thật đã nhận được giá trị.

Ba thứ tôi mang về:

**Thứ nhất, phân biệt được output hệ thống với value của người dùng.** Nera trả về ba căn kèm lý do phù hợp — đó là output. Khách bỏ nửa buổi đi xem một trong ba căn đó mới là bằng chứng họ tin. Suốt hai tuần build, nhóm đo mình bằng cái thứ nhất.

**Thứ hai, nhịp đo phải đi từ bản chất hành vi.** Tôi từng định đề xuất đo người dùng hoạt động hằng ngày cho Nera vì đó là chỉ số ai cũng dùng. Sai. Không ai đi xem nhà mỗi ngày. Ép chỉ số đó lên sẽ dẫn nhóm đi xây thông báo đẩy để kéo khách mở app, trong khi việc cần làm là gợi ý đúng hơn để khách tìm được nhà nhanh hơn.

**Thứ ba, với Nera thì dùng ít lại mới là tốt.** Nếu hệ thống hiểu đúng nhu cầu ngay từ đầu, khách cần ít lượt đi xem hơn và chu kỳ tìm nhà ngắn lại. Một sản phẩm mà thành công đồng nghĩa với việc người dùng rời đi sớm hơn thì không thể đo bằng retention thuần. Đây là điều tôi chưa từng nghĩ tới trước hôm nay, và nó đổi cách tôi đọc mọi con số của dự án.

**Việc làm tiếp:** hệ thống hiện chưa có event `viewing_completed`, tức là chưa đo được North Star. Sale phải có thao tác đánh dấu buổi xem đã diễn ra. Đây là thứ tôi sẽ đề xuất bổ sung ở sprint sau, trước khi thêm bất kỳ tính năng mới nào.
