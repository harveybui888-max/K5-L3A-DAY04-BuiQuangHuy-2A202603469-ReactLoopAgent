# Ghi chú phân tích · Day 04

Họ và tên: Bùi Quang Huy

MSSV: 2A202603469

## 1. Case được chọn

- ID case: **A5**
- Câu hỏi: Tìm danh sách đầy đủ tất cả incident GitHub Actions trong năm 2025, đồng thời nêu rõ độ đầy đủ của kết quả.
- Câu trả lời lần chạy 1 (trích): “Theo dữ liệu từ GitHub Status, không có sự cố nào liên quan đến GitHub Actions trong năm 2025.”
- Điểm lần chạy 1: **15/100** (factuality 0, completeness 10, evidence 0, boundaries 5).

## 2. Bằng chứng trong trace

Trace của `eval_before.json` cho thấy status tool chỉ tìm trong 50 incident gần nhất và cảnh báo rõ rằng kết quả rỗng không chứng minh incident không tồn tại. Snapshot được chụp ngày 2026-11-09; các incident được agent đọc chi tiết đều là incident năm 2026.

```text
get_status(
  provider="github",
  component="actions"
)
matching_incidents_in_last_50=23
note="Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
captured_at="2026-11-09T03:50:47.065020+00:00"

get_incident_details(provider="github", incident_id="t89g2q1800fr")
name="Incident with Actions"
status="resolved"
url="https://stspg.io/lv4q3xm9r0rk"
```

Trace còn có các incident đã giải quyết khác trong năm 2026. Không có bằng chứng trong trace để kết luận có hoặc không có incident trong năm 2025. Nguyên nhân có thể là system prompt ban đầu chưa nhấn mạnh đủ giới hạn lịch sử của snapshot, nên model đã biến “không thấy trong dữ liệu hiện có” thành “không có”.

## 3. Thay đổi đã thực hiện

- Trong [agent.py](./agent.py), bổ sung hướng dẫn rằng status tools chỉ tìm tối đa 50 incident gần nhất, không phải kho lịch sử đầy đủ. Nếu người dùng hỏi về giai đoạn cũ hoặc danh sách theo năm mà snapshot không bao phủ, agent phải nêu không đủ dữ liệu để xác nhận thay vì kết luận không có incident.
- Trong [tests/test_validation.py](./tests/test_validation.py), thêm test đảm bảo system prompt nhắc rõ giới hạn 50 incident và không suy luận không tồn tại từ kết quả tìm kiếm.
- Chọn thay đổi này vì trace A5 chứng minh câu trả lời đã khẳng định vượt quá bằng chứng, đúng với giới hạn được tool nêu ra.
- Sau thay đổi, A5 tăng từ **15 lên 55**. Câu trả lời lần hai đã nói snapshot không đủ để xác nhận danh sách đầy đủ, nhưng vẫn có câu “Dữ liệu hiện có không cho thấy bất kỳ sự cố nào từ năm 2025.” Câu này còn mơ hồ; cách diễn đạt an toàn hơn là “Dữ liệu hiện có không đủ để xác nhận có hay không có incident trong năm 2025.”

## 4. So sánh hai lần chạy

Cả hai lần dùng cùng cấu hình: provider `openrouter`, model agent `openai/gpt-4o-mini`, judge `openrouter` / `openai/gpt-4o-mini`, `max_steps=8`, dataset `core`. Cả hai đều chấm đủ 22/22 case, không có case `unscored`.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng | 78.64/100 | 80.23/100 |
| Điểm case A5 | 15/100 | 55/100 |
| Agent model calls | 53 | 49 |
| Tool calls | 51 | 33 |
| Agent tokens (input/output) | 42,004 / 10,904 | 34,855 / 9,005 |
| Judge tokens (input/output) | 40,605 / 3,101 | 31,634 / 3,228 |
| Chi phí agent theo provider | ~$0.012459 | ~$0.010334 |
| Chi phí judge theo provider | ~$0.007951 | ~$0.006682 |
| Tổng chi phí theo provider | ~$0.020410 | ~$0.017016 |

- Lỗi được cải thiện: A5 không còn khẳng định thẳng rằng không có incident năm 2025; điểm tăng 40.
- Lỗi còn lại/mới xuất hiện: cách diễn đạt A5 vẫn có thể bị hiểu là phủ định; điểm các case không tăng đồng đều. Ví dụ E2 giảm 95→85, A1 giảm 75→65 và H3 giảm 100→80. Cần xem câu trả lời và trace từng case, không suy ra tất cả thay đổi điểm đều do prompt.
- Thống kê token và chi phí là tổng usage provider báo cáo trong JSON eval; có thể biến động theo provider/model.

## 5. Giới hạn

- Judge là ước lượng, có thể thiên lệch; lần chạy này dùng cùng model cho agent và judge. Một lần chạy mỗi phiên bản không chứng minh cải thiện ổn định.
- Snapshot có giới hạn về thời gian và số incident; kết quả không thể chứng minh lịch sử đầy đủ hay trạng thái hiện tại ngoài snapshot.
- Điểm eval không phải điểm lab. Cần đối chiếu `expected`, câu trả lời và trace, đặc biệt với điểm bất thường.
- Các test offline xác nhận contract và logic đã kiểm tra, nhưng không chứng minh chất lượng câu trả lời trong mọi tình huống thực tế.
