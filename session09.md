# Bài 9: RAG Grounding dập tắt ảo giác hợp đồng thu mua

## So sánh nhanh

| | Hỏi LLM không kèm tài liệu | RAG Grounding (dán kèm tài liệu) |
|---|---|---|
| Nguồn câu trả lời | Kiến thức chung học từ Internet | Văn bản hợp đồng được dán vào prompt |
| Rủi ro | Cao: AI tự suy đoán | Thấp: AI chỉ trích xuất và diễn giải |
| Khi thiếu dữ liệu | Vẫn trả lời, thường bịa | Trả lời "Không có dữ liệu trong tài liệu" |

## Phần 1: Phân tích nguyên nhân ảo giác

1. **AI dự đoán từ tiếp theo có xác suất cao nhất, không tra cứu sự thật.** Mô hình không có kho dữ liệu về hợp đồng MartX. Nó chỉ sinh ra câu trả lời nghe hợp lý nhất.
2. **Dữ liệu huấn luyện chỉ chứa quy định chung của Internet.** Các mức phạt như "20% giá trị đơn" hay "đình chỉ 30 ngày" là mô típ phổ biến trong nhiều hợp đồng khác. AI ghép chúng lại như thể đó là quy định của MartX.
3. **Câu hỏi có cụm "Theo quy định MartX" nhưng không có tài liệu nào kèm theo.** AI không được huấn luyện để nói "tôi không biết", nên nó lấp khoảng trống bằng con số có vẻ hợp lý.
4. **Giọng văn tự tin không phản ánh độ chính xác.** AI viết trôi chảy cả khi nội dung sai. Đó là lý do nhân viên tin và phạt nhầm.

**Kết quả:** Hợp đồng thực tế ghi "phạt 5% giá trị lô bị dập nát và đổi hàng trong 24h", còn AI nói "phạt 20% tổng đơn và đình chỉ 30 ngày". Sai cả mức phạt, cơ sở tính phạt (lô hay cả đơn) và chế tài, dẫn tới tranh chấp.

## Phần 2: Prompt RAG Grounding hoàn chỉnh

```
Bạn là trợ lý pháp chế cho bộ phận Thu mua MartX. Chỉ trả lời dựa trên
văn bản hợp đồng dưới đây.

### 1. VĂN BẢN HỢP ĐỒNG
"""
Điều khoản chất lượng hàng hóa: Trường hợp rau giao bị dập nát trên 10%,
nhà cung cấp bị phạt 5% giá trị lô hàng bị dập nát và phải đổi hàng
trong vòng 24 giờ.
"""

### 2. CÂU HỎI NGHIỆP VỤ
Nhà cung cấp giao rau bị dập nát trên 10% thì bị phạt bao nhiêu?

### 3. KHÓA AN TOÀN (NEGATIVE CONSTRAINT)
- Chỉ dùng thông tin có trong văn bản ở mục 1.
- Nếu thông tin không có trong văn bản, tuyệt đối trả lời đúng câu
  "Không có dữ liệu trong tài liệu", không tự suy đoán, không dùng kiến
  thức bên ngoài.
- Khi trả lời, trích nguyên văn điều khoản làm căn cứ.
```

**Kết quả kỳ vọng:**
> Phạt 5% giá trị lô hàng bị dập nát và phải đổi hàng trong 24 giờ.
> Căn cứ: "nhà cung cấp bị phạt 5% giá trị lô hàng bị dập nát và phải đổi hàng trong vòng 24 giờ".

**Kiểm tra khóa an toàn:** Đổi câu hỏi thành *"Nhà cung cấp có bị đình chỉ nhập hàng 30 ngày không?"* thì AI phải trả lời đúng một câu: **"Không có dữ liệu trong tài liệu"**. Hợp đồng không đề cập đình chỉ nên AI không được tự thêm.
