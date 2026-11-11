# Ghi chú phân tích · Day 04

Họ và tên: nguyễn văn huy
MSSV: 2A202603432

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: Agent liệt kê năm incident Actions gần đây từ năm 2026, rồi nói không thể khẳng định không có incident năm 2025.
- Điểm trong eval baseline hiện tại: 85/100 (judge; tham khảo).

## 2. Bằng chứng trong trace

Trace `get_status` cho `provider=github`, `component=GitHub Actions` ghi nhận:

```text
captured_at: 2026-11-09T03:50:47.065020+00:00
matching_incidents_in_last_50: 11
note: Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.
Các incident trả về có updated_at trong tháng 10–11/2026; trace không cung cấp dữ liệu năm 2025.
```

Nguyên nhân có thể xảy ra: câu trả lời dành nhiều chỗ liệt kê incident gần đây dù không trả lời được giai đoạn năm 2025, dễ tạo ấn tượng rằng dữ liệu gần đây có thể thay thế bằng chứng lịch sử.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Làm rõ `SYSTEM_PROMPT` trong `agent.py`: chỉ dùng mốc thời gian có trong tool, nêu giới hạn snapshot/50 incident, không suy luận cho giai đoạn ngoài dữ liệu, và nói rõ provider chưa xác minh được khi tool lỗi. Thêm test kiểm tra các chỉ dẫn được đưa vào system message.
- Vì sao chọn thay đổi này: Phản hồi judge của A5 nêu nguy cơ câu trả lời gây hiểu nhầm về năm 2025; đây là thay đổi prompt nhỏ, không sửa dataset hay grading.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 80.00 (22/22 case) | 84.09 (22/22 case) |
| Điểm case đã chọn (A5) | 85 | 75 |
| Agent model calls | 49 | 48 |
| Tổng token (input + output; agent và judge) | 74,177 | 76,249 |
| Cấu hình | OpenRouter / `openai/gpt-4o-mini` | OpenRouter / `openai/gpt-4o-mini` |

- Kết quả lần eval sau: Điểm tổng tăng từ 80.00 lên 84.09; một lần chạy cho mỗi phiên bản không chứng minh tính ổn định.
- Lỗi còn lại hoặc mới xuất hiện: H2 được 50 và H1/H6 được 60 ở cả hai lần; A5 giảm từ 85 xuống 75. Điểm judge là ước lượng và câu trả lời có thể dao động giữa các lần chạy.
- Giới hạn so sánh: Hai file hiện có cùng cấu hình nhưng được tạo bằng code hiện tại; không còn artifact của phiên bản agent trước cải tiến để đo tác động nhân quả của thay đổi prompt. Chi phí tiền tệ không có trong báo cáo usage.

## 5. Giới hạn

Điểm judge là ước lượng, có thể thiên lệch hoặc lỗi định dạng. Snapshot core được chụp ngày 09/11/2026 và tool chỉ tìm trong tối đa 50 incident gần nhất; nó không thể xác nhận lịch sử ngoài dữ liệu đó hoặc trạng thái live. Mỗi báo cáo hiện chấm đủ 22 case và cùng cấu hình OpenRouter/GPT-4o-mini. Tuy vậy, `eval_before.json` được chạy lại sau khi thay đổi agent đã có trong workspace, nên không đại diện cho code trước cải tiến; cần lưu baseline trước khi thay đổi trong lần thực hành sau để so sánh nhân quả hợp lệ.

## 6. Demo CP7

- Trace nhiều bước chạy thành công với snapshot: `get_status` (step 1) → `get_incident_details` (step 2) → câu trả lời cuối (step 3), có URL incident Cloudflare.
- Trace lỗi giả lập: `get_status` trả `TimeoutError`; agent hoàn tất ở step 2 bằng cách nói rõ không xác minh được trạng thái, không suy luận outage.
- Còn thao tác thủ công: xác nhận tên repo theo quy tắc trong `SUBMISSION.md` và nộp link repo lên LMS.

## 7. Thử thách bonus

- Kết quả `eval_bonus.json`: 10/10 case được chấm; 97.90/100, tương đương 14.69/15 điểm bonus tham khảo. Cấu hình: Starimg / `gemini-3.8-flash`, tối đa 16 bước.
- Agent gọi đủ status tools được yêu cầu, lấy thêm các trang incident khi cần, thử lại lỗi 429 có giới hạn và sửa tham số sau `INVALID_ARGUMENTS`. Tên các provider được hỏi nhưng chưa kiểm tra sẽ được yêu cầu gọi trước khi hoàn tất.
- 160 mục `distractors` được loại khỏi observation gửi cho model để giảm context; bản kết quả đầy đủ vẫn giữ trong trace phục vụ kiểm tra. Test xác nhận cả hai hành vi.
- Đối chiếu B10: judge chấm 84 và nhận xét trạng thái Vercel chưa được xác minh; tuy nhiên trace có lần retry Vercel thành công với `indicator: none`, và câu trả lời nêu `All Systems Operational`. Cần xem đây là bất đồng của judge, không phải kết luận chắc chắn có hallucination. B8 được 95.
- Hạn chế còn lại: điểm judge chỉ là tham khảo; chi phí tiền tệ không có trong báo cáo usage. Snapshot không đại diện cho trạng thái live.
