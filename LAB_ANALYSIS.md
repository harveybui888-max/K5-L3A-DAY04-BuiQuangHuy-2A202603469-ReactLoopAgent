# Ghi chú phân tích eval

## Cấu hình

- Bộ eval chính: 22 case, tất cả đều được chấm; không có case `unscored`.
- Agent và judge: `openrouter` / `openai/gpt-4o-mini`.
- `max_steps`: 8 trong cả hai lần chạy.
- Điểm judge là ước lượng. Vì model được gọi lại ở từng lần chạy, khác biệt giữa hai lần không chứng minh mức cải thiện ổn định.

## So sánh từng case

| Case | Trước | Sau | Chênh lệch |
|---|---:|---:|---:|
| E1 | 55 | 55 | 0 |
| E2 | 95 | 85 | -10 |
| E3 | 80 | 90 | +10 |
| E4 | 70 | 95 | +25 |
| E5 | 80 | 75 | -5 |
| E6 | 70 | 75 | +5 |
| A1 | 75 | 65 | -10 |
| A2 | 100 | 100 | 0 |
| A3 | 75 | 70 | -5 |
| A4 | 100 | 100 | 0 |
| A5 | 15 | 55 | +40 |
| A6 | 75 | 75 | 0 |
| A7 | 75 | 75 | 0 |
| A8 | 100 | 100 | 0 |
| H1 | 60 | 60 | 0 |
| H2 | 100 | 100 | 0 |
| H3 | 100 | 80 | -20 |
| H4 | 100 | 100 | 0 |
| H5 | 100 | 100 | 0 |
| H6 | 60 | 60 | 0 |
| H7 | 70 | 75 | +5 |
| H8 | 75 | 75 | 0 |
| **Trung bình** | **78.64** | **80.23** | **+1.59** |

Điểm tăng ở A5, E3, E4, E6 và H7; giảm ở E2, E5, A1, A3 và H3. Các case còn lại giữ nguyên. Vì vậy, điểm trung bình tăng nhưng kết quả không cải thiện đồng đều.

## Phân tích lỗi và thay đổi

- **Case chọn: A5**, yêu cầu danh sách đầy đủ sự cố GitHub Actions trong năm 2025.
- Trước thay đổi, câu trả lời nói “không có sự cố nào liên quan đến GitHub Actions trong năm 2025”. Trace chỉ có kết quả tìm chuỗi trong 50 incident gần nhất; tool cũng cảnh báo không tìm thấy không chứng minh không có incident. Câu trả lời đã khẳng định vượt quá bằng chứng.
- Đã sửa system prompt trong [agent.py](./agent.py) để nêu giới hạn 50 incident của snapshot và yêu cầu không kết luận về một giai đoạn cũ hơn nếu dữ liệu không bao phủ giai đoạn đó. Thêm test cho hướng dẫn này trong [tests/test_validation.py](./tests/test_validation.py).
- A5 tăng từ **15 lên 55**. Câu trả lời sau nói không thể cung cấp danh sách đầy đủ và nêu giới hạn của 50 incident, nhưng vẫn có câu “Dữ liệu hiện có không cho thấy bất kỳ sự cố nào từ năm 2025.” Câu này còn có thể bị hiểu thành kết luận phủ định; cần diễn đạt dứt khoát hơn là **không đủ dữ liệu để xác nhận có hay không có incident trong năm 2025**.

## Chi phí, trace và giới hạn

Theo usage được provider ghi trong hai file eval:

| Lần chạy | Model calls | Tool calls | Token agent (input/output) | Token judge (input/output) | Chi phí provider báo cáo |
|---|---:|---:|---:|---:|---:|
| Trước | 53 | 51 | 42,004 / 10,904 | 40,605 / 3,101 | ~$0.020410 |
| Sau | 49 | 33 | 34,855 / 9,005 | 31,634 / 3,228 | ~$0.017016 |

Chi phí là giá trị provider báo cáo, có thể thay đổi theo giá model và không phải điểm lab. Các trace cho thấy lần chạy sau ít model/tool calls hơn, nhưng cũng có một số điểm giảm; một lần chạy mỗi phiên bản không đủ để kết luận cải thiện ổn định. Judge dùng cùng model với agent nên có thể thiên lệch. Kết quả eval không thay thế việc đọc câu trả lời, trace và đối chiếu với bằng chứng.

## Kết quả

- Unit tests offline: 38 test bài chính và 3 test bonus đều qua sau thay đổi.
- Eval chính: 22/22 case được chấm ở cả hai lần; điểm **78.64 → 80.23/100**.
- Bonus chưa chạy; không tính điểm bonus.
- File kết quả: [eval_before.json](./eval_before.json), [eval_after.json](./eval_after.json).
