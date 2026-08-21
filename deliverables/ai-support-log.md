# AI Support Log — Nhóm: Huyền, Ánh, Tú Anh

> Ghi lại thực tế các bước nhóm mình dùng AI (ChatGPT/Claude/Gemini...) khi làm bài Capstone.
> Nhóm cam kết trung thực, con người kiểm soát 100% quyết định, nhãn vàng và phán quyết cuối cùng.

---

## 1. Bảng khai báo dùng AI ở từng bước

| #   | Bước                                  | AI giúp nhóm làm gì                                                                                        | Cách con người kiểm tra & chốt kết quả                                                                                                                         |
| --- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Phase 1: Sinh câu hỏi thử nghiệm**  | Nhóm đưa trước khung grid, nhờ AI gợi ý các kiểu diễn đạt ngắn/dài/viết tắt của học viên.                  | Nhóm ngồi soi từng câu (Keep/Rewrite/Reject). Bỏ 2 câu trùng ý, tự viết lại 4 câu sát thực tế hơn (`sc-04` teencode, `sc-05` trỏ slide, `sc-08` giả định sai). |
| 2   | **Phase 3: Gợi ý hàm code checks**    | Nhờ AI viết mẫu đoạn regex/string matching trong Python cho tiêu chí `quote_verbatim` và `followup_count`. | Chạy thử trực tiếp bằng file `code_checks.py`, tự kiểm tra lại từng dòng output xem code báo pass/fail có chuẩn xác không.                                     |
| 3   | **Phase 4: Gợi ý dàn ý Judge Prompt** | Nhờ AI lên khung cho `judge_prompt.md` và tóm tắt nhanh mấy case bất đồng giữa AI và người.                | Nhóm tự tay đọc 4 case lệch ở Vòng 1, tự viết thêm 3 ví dụ Near-Miss thực tế vào Prompt V2 để kéo độ đồng thuận từ 65% lên 85%.                                |
| 4   | **Phase 6: Soạn bản nháp Report**     | Nhờ AI lên khung trình bày báo cáo PM trong file `REPORT.md`.                                              | Nhóm tự chốt trước các ngưỡng điểm (Thresholds), tự đưa ra quyết định **HOLD (CHƯA SHIP)** chứ không nghe theo gợi ý "cho qua" của AI.                         |

---

## 2. Nhật ký tự rút kinh nghiệm

- **AI đã giúp tôi ở đâu?** AI giúp mình đặt câu hỏi theo kiểu gõ tắt, teencode vội vã của học viên rất nhanh, tiết kiệm thời gian nghĩ câu chữ.
- **AI sai, hời hợt hoặc làm mất coverage ở đâu?** AI hay sinh ra mấy câu hỏi quá tròn trịa, sạch sẽ và đủ thông tin. Nếu dùng nguyên văn câu AI tạo thì sẽ mất sạch các câu hỏi mơ hồ, cắt cụt thiếu context ngoài đời.
- **Tôi đã tự sửa hoặc quyết định lại điều gì?** Mình trực tiếp xóa bỏ 2 câu AI làm quá chi tiết, tự gõ lại câu `sc-06` ("Eval này ổn chưa ad?") cắt hết bối cảnh slide để ép Tutor phải tự biết hỏi lại học viên.

---

## 3. Cam kết tuân thủ 7 Luật Bài Lab Capstone

- [x] **Luật 1 (Human-first Coverage):** Con người tự chốt Dimensions & Combinations trước, AI chỉ diễn đạt lại câu chữ.
- [x] **Luật 2 (Human Baseline):** Nhãn người 100% do Huyền, Ánh, Tú Anh tự chấm độc lập (Đồng thuận 90%).
- [x] **Luật 3 (Unique Test Value):** Mỗi câu hỏi đại diện cho 1 tình huống rủi ro riêng biệt.
- [x] **Luật 4 (Pre-committed Thresholds):** Chốt ngưỡng điểm trước khi xem kết quả chạy thật.
- [x] **Luật 5 (No overall pass rate hiding):** Báo cáo đầy đủ từng slice (nhìn rõ câu mơ hồ bị 0% pass rate).
- [x] **Luật 6 (Minimal Calibration Changes):** Mỗi lần sửa prompt chỉ thêm ít ví dụ Near-Miss và ghi rõ lý do.
- [x] **Luật 7 (No unrecorded changes):** Lưu đầy đủ 9 file minh chứng thô trong `deliverables/evidence/`.
