# AI Support Log — Track1 Day23

Học viên: Vũ Thế Lực · MHV 2A202602008

Công cụ dùng: Claude (Claude Code)

---

## Tôi đã dùng AI vào việc gì

**1. Đối chiếu core action với 5 tiêu chí tự kiểm.**
Tôi đưa hai phương án core action: `property_saved` (lưu căn) và `booking_requested` (đặt lịch xem), rồi nhờ AI chấm từng phương án theo 5 tiêu chí trong brief. Kết luận chọn `booking_requested` là của tôi, dựa trên lập luận: lưu căn không tốn gì của khách nên là tín hiệu yếu, còn đặt lịch bắt khách trả giá bằng nửa buổi thời gian thật.

**2. Kiểm tra tính nhất quán giữa các mục.**
Nhờ AI rà xem cadence kết luận ở mục 02 có khớp với window của retention ở mục 04 không, và mọi event ở mục 06 có map về metric nào ở mục 03 không. Phát hiện được một chỗ lệch: ban đầu tôi viết window retention theo tuần nhưng threshold lại để "ít nhất 2 lần", không hợp với hành vi thưa 2–6 lần trong 4–10 tuần. Đã sửa xuống 1 lần.

**3. Soát lại phần định nghĩa event.**
Nhờ AI kiểm tra từng event có ghi đúng thời điểm hành vi hoàn tất hay không. Phát hiện `viewing_completed` nếu ghi tự động khi qua giờ hẹn thì sai, vì khách có thể không đến. Đã sửa thành chỉ ghi khi sale chủ động đánh dấu.

**4. Diễn đạt.**
Nhờ viết lại một số câu cho gọn hơn, đặc biệt phần metric hypothesis và các tiêu chí nghiệm thu.

---

## Phần tôi tự làm, không nhờ AI

- Chọn persona và viết core job bằng lời người dùng.
- Quyết định phạm vi: chỉ phân tích luồng từ mô tả nhu cầu tới lịch hẹn, bỏ phần sale và admin.
- Kết luận cadence là theo dự án chứ không theo ngày hay tuần.
- Nhận định "với Nera, khách dùng ít lại mới là tốt" và quyết định không đưa tần suất vào North Star.
- Ghi chú phần nào của hệ thống chưa có event, dựa trên hiểu biết của tôi về mã nguồn Nera.

---

## Chỗ tôi không đồng ý với AI

AI đề xuất đưa "số lượt trò chuyện trung bình mỗi phiên" vào nhóm engagement. Tôi bỏ, vì với Nera hội thoại dài hơn thường nghĩa là hệ thống chưa hiểu đúng nhu cầu, phải hỏi đi hỏi lại. Đó là chỉ số dễ đọc ngược.

---

## Giả định chưa được kiểm chứng

Con số 2–6 lần đặt lịch trong chu kỳ 4–10 tuần là suy luận từ đặc thù hành vi mua nhà, chưa có dữ liệu người dùng thật của Nera. Cần đo lại khi có cohort đầu tiên đủ lớn.
