# REPORT — Eval loop A→Z: VLearn AI Tutor

Report A→Z của eval loop — mỗi mục ứng một phase của bài lab. Mọi số liệu và quyết
định trong đây phải dẫn được xuống file data thô trong `evidence/` (dataset-v1.jsonl,
results-vN.jsonl, labels.csv, judge-prompt-vN.md, verdicts-vN.jsonl, braintrust-link.md).


---

## 1. Input Grid

> Lưới input = trục "ai hỏi" × "hỏi kiểu gì". LLM giúp sinh input, con người kiểm soát
> coverage. Trả lời các câu hỏi sau rồi vẽ lưới của bạn.

- AI Tutor phục vụ các nhóm người dùng: **Học viên mới** (hỏi khái niệm cơ bản), **Học viên đang làm bài Capstone** (hỏi quy trình, xin đáp án/gợi ý), **Học viên ôn lại bài** (so sánh kiến thức rải rác).
- Các ý định (intent) chính: Hỏi khái niệm (`khai_niem`), So sánh / Áp dụng (`so_sanh`/`ap_dung`), Hỏi ngoài phạm vi bài (`ngoai_bai`), Xin đáp án bài tập (`xin_dap_an`), Hỏi mơ hồ / Thiếu bối cảnh (`mo_ho`).
- Ô **rủi ro cao nhất**: Xin đáp án (`xin_dap_an`), Giả định sai (`gia_dinh_sai`), và Mơ hồ kèm Deixis context (`mo_ho_phu_thuoc_slide`). Ô **tần suất cao nhất**: Hỏi khái niệm chuẩn (`khai_niem` × `co_san`).

### Lưới của bạn

| Nhóm user \ Intent | Khái niệm | So sánh / Tổng hợp | Mơ hồ (Deixis / Không slide) | Xin đáp án / Out-of-scope |
|---|---|---|---|---|
| **Học viên mới** | `sc-01-vibe-check`, `sc-02-eval-lifecycle` | `sc-15-hamel-eval-levels` | `sc-06-unclear-no-slide` | `sc-09-out-weather` |
| **Học viên đang làm bài** | `sc-04-trace-code` (teencode) | `sc-16-multi-part-citation` | `sc-05-deixis-passrate` (Slide s47) | `sc-11-cheat-capstone-answer`, `sc-12-cheat-prompt-generator` |
| **Học viên ôn bài / Nghiên cứu** | `sc-13-anthropic-graders`, `sc-14-chiphuyen-pipeline` | `sc-03-offline-vs-judge`, `sc-08-false-assumption-bias` | `sc-07-ambiguous-rubric` | `sc-10-out-api-pricing` |

---

## 2. Dataset v1

> Dataset là "bộ đề thi" của tutor. Nêu rõ nó phủ những ô nào trong input-grid.

- `dataset.jsonl` có **20 câu (scenarios)**, được lưu tại `dataset.jsonl` và `deliverables/evidence/dataset-v1.jsonl`.
- Tỉ lệ phân bổ: 
  - **In-scope:** 14 câu (70%)
  - **Out-of-scope:** 4 câu (20%) — vượt yêu cầu bắt buộc ($\ge 2$ câu)
  - **Mơ hồ:** 2 câu (10%) — đạt yêu cầu bắt buộc ($\ge 2$ câu)
  - **High-risk:** 5 câu (25%) — vượt yêu cầu bắt buộc ($\ge 2$ câu)
- Nguồn câu hỏi: Sinh kết hợp LLM paraphrase + review & bồi ràng buộc thực tế từ con người (Keep 14 câu, Rewrite 4 câu, Reject 2 câu bộc lộ sai lệch).
- Reviewer: Cả team đã review 100% câu hỏi qua tiêu chí Gate 1.
- Nếu chỉ giữ 10 câu nòng cốt: Giữ `sc-01`, `sc-03`, `sc-04`, `sc-05`, `sc-06`, `sc-08`, `sc-09`, `sc-10`, `sc-11`, `sc-16` (vì phủ đủ mọi ô rủi ro cao và các loại intent chính).

### Danh sách scenario (bảng tóm tắt)

| scenario_id | ô trong lưới | expected | nguồn câu hỏi |
|---|---|---|---|
| `sc-01-vibe-check` | `khai_niem` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s10) |
| `sc-02-eval-lifecycle` | `khai_niem` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s07) |
| `sc-03-offline-vs-judge` | `so_sanh` × `rai_rac` × `ro` | `in_scope` | Human + LLM (Slide s12 & s50) |
| `sc-04-trace-code` | `khai_niem` × `co_san` × `teencode` | `in_scope` | Human rewrite (Slide s29) |
| `sc-05-deixis-passrate` | `ap_dung` × `co_san` × `mo_ho_slide` | `in_scope` | Human rewrite (Slide s47) |
| `sc-06-unclear-no-slide` | `mo_ho` × `khong_ro` × `mo_ho_khong_slide` | `unclear` | Human created |
| `sc-07-ambiguous-rubric` | `mo_ho` × `khong_ro` × `mo_ho_khong_slide` | `unclear` | Human created |
| `sc-08-false-assumption-bias` | `khai_niem` × `co_san` × `gia_dinh_sai` | `in_scope` | Human rewrite (Slide s53) |
| `sc-09-out-weather` | `ngoai_bai` × `khong_co` × `ro` | `out_of_scope` | Human created |
| `sc-10-out-api-pricing` | `ngoai_bai` × `khong_co` × `ro` | `out_of_scope` | Human created |
| `sc-11-cheat-capstone-answer` | `xin_dap_an` × `khong_co` × `ro` | `out_of_scope` | Human created |
| `sc-12-cheat-prompt-generator` | `xin_dap_an` × `khong_co` × `ro` | `out_of_scope` | Human created |
| `sc-13-anthropic-graders` | `khai_niem` × `co_san` × `ro` | `in_scope` | Blog Anthropic |
| `sc-14-chiphuyen-pipeline` | `khai_niem` × `co_san` × `ro` | `in_scope` | Sách Chip Huyen Ch4 |
| `sc-15-hamel-eval-levels` | `khai_niem` × `co_san` × `ro` | `in_scope` | Blog Hamel Husain |
| `sc-16-multi-part-citation` | `so_sanh` × `rai_rac` × `nhieu_y` | `in_scope` | Human rewrite (Slide s17) |
| `sc-17-code-vs-llm-choice` | `ap_dung` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s40) |
| `sc-18-expert-in-loop` | `khai_niem` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s58) |
| `sc-19-golden-output-definition` | `khai_niem` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s11) |
| `sc-20-flywheel-concept` | `khai_niem` × `co_san` × `ro` | `in_scope` | Human + LLM (Slide s19) |

---

## 3. Rubric v1

> Rubric = định nghĩa "đủ tốt" mà cả team chấm giống nhau. Thu hẹp scope trước khi
> viết tiêu chí.

- Tutor trả lời một câu in-scope **"đủ tốt"** khi: (1) Trả lời chính xác bám 100% corpus không bịa đặt, (2) Trích dẫn đúng `doc_id#section_id` kèm `quote` nguyên văn ngắn, (3) Đưa ra 3 câu hỏi gợi mở đào sâu tư duy sư phạm.
- Với câu out-of-scope hoặc xin đáp án bài tập: Tutor phải nhận diện đúng `scope: out_of_scope`, từ chối khéo léo và lái học viên quay lại chủ đề bài học.

### Rubric chi tiết của bạn

| Tiêu chí | Pass khi (Yes) | Fail khi (No) | Blocker? | Ví dụ Pass | Ví dụ Fail | Ví dụ Borderline (Tranh luận) |
|---|---|---|---|---|---|---|
| `schema_valid` | Output là 1 JSON object đủ 4 fields (`scope`, `answer`, `sources`, `followup_questions`) | Vỡ JSON, thiếu field, bọc markdown sai | **Có** | `sc-01` (JSON hợp lệ) | Row bị cắt nửa chừng | JSON dư thừa key lạ |
| `citation_exists` | `doc_id` và `section_id` có thật trong `manifest.json` | Địa chỉ nguồn không tồn tại trong corpus | **Có** | `doc_id: slide-day19-20`, `section_id: s10` | `doc_id: random-doc`, `section_id: s999` | Citation đúng doc nhưng sai section |
| `quote_verbatim` | Đoạn `quote` nằm chính xác nguyên văn trong section text đã cite | `quote` bị tự diễn đạt lại, bịa chữ | **Có** | Trích đúng từng từ từ slide s10 | Quote tự tóm tắt lại chữ trong slide | `sc-05` (tóm tắt ý đúng nhưng thiếu 2 chữ so với nguyên văn) |
| `scope_accuracy` | Phân loại đúng in-scope vs out-of-scope theo câu hỏi | Nhầm out-of-scope thành in-scope (hoặc ngược lại) | **Có** | `sc-09` (Hỏi thời tiết $\rightarrow$ `out_of_scope`) | `sc-09` (Hỏi thời tiết $\rightarrow$ `in_scope`) | `sc-12` (Xin prompt $\rightarrow$ Tutor lại nhận `in_scope`) |
| `answer_groundedness` | `answer` bám 100% thông tin `retrieved`, không tự bịa | Trả lời thông tin không có trong nguồn trích | **Có** | Giải thích calibration dựa vào slide s51 | Tự bịa bảng giá GPT-4o từ trí nhớ model | Trả lời đúng thực tế nhưng nguồn trích không chứa thông tin đó |
| `followup_quality` | Gồm đúng 3 câu hỏi gợi mở đào sâu tư duy học viên | Không có 3 câu, hoặc câu hỏi xã giao sáo rỗng | Không | 3 câu hỏi so sánh/áp dụng khái niệm vừa học | "Bạn có thắc mắc gì nữa không?" | 3 câu hỏi nhưng trùng lặp ý với nhau |
| `ambiguity_handling` | Với câu mơ hồ thiếu slide context, Tutor yêu cầu clarify | Tự phỏng đoán trả lời khi câu hỏi mơ hồ | **Có** | Yêu cầu học viên cung cấp thêm thông tin | `sc-06` (Hỏi "Eval ổn chưa" tự phỏng đoán trả lời) | Trả lời tổng quan rồi nhắc học viên làm rõ |

---

## 4. Routing Map

> Cái gì kiểm bằng code, cái gì cần LLM judge, cái gì phải đến tay expert. Không phải
> tiêu chí nào cũng cần LLM.

- **Spec Gap vs Generalization Gap**:
  - *Spec Gap (Lỗi Prompt):* Lỗi `sc-12` (Tutor nhận viết hộ prompt Capstone) là do Prompt ban đầu chưa ghi rõ cấm làm hộ bài tập. **Khắc phục:** Sửa `SYSTEM_PROMPT` trong `tutor/tutor.py` (ghi vào Backlog), chưa cần tốn LLM judge.
  - *Generalization Gap:* Lỗi `sc-05` (lệch verbatim quote) hay bias của model $\rightarrow$ Thích hợp để tự động hóa bằng Code + LLM Judge.

### Bảng routing

| Tiêu chí | Code | LLM judge | Con người | Lý do |
|---|---|---|---|---|
| `schema_valid` | ✅ | ❌ | ❌ | Rẻ và chính xác 100% bằng Python `json.loads()` ($0/run). |
| `citation_exists` | ✅ | ❌ | ❌ | So khớp key với `manifest.json` bằng Python code. Nhanh và chắc chắn. |
| `quote_verbatim` | ✅ | ❌ | ❌ | Kiểm tra chuỗi con (substring) trong section text bằng Python code. |
| `scope_accuracy` | ❌ | ✅ | ❌ | Phân tích ý định câu hỏi đòi hỏi đọc hiểu ngữ nghĩa của LLM Judge. |
| `answer_groundedness` | ❌ | ✅ | ❌ | LLM Judge đối chiếu ngữ nghĩa giữa `answer` và `retrieved` text. |
| `followup_quality` | ❌ | ✅ | ❌ | LLM Judge đánh giá tính sư phạm của 3 câu hỏi gợi mở. |
| `ambiguity_handling` | ❌ | ❌ | ✅ (Audit 10%) | Các case mơ hồ ranh giới nhạy cảm cần Expert kiểm tra để đảm bảo UX học viên. |

---

## 5. Calibration Report

> Judge chỉ đáng tin khi đã calibrate với chuẩn vàng của con người. Đây là minh chứng
> cho việc đó.

- Nhóm đã **gán nhãn tay 20/20 rows** (được lưu tại `labels.csv` và `deliverables/evidence/labels.csv`).
- **Vòng 1 (Baseline V1):** `EVAL_JUDGE_MODEL` (`openrouter/openai/gpt-4o-mini`) đạt **65% Agreement** (13/20).
  - *Nguyên nhân lệch V1:* Judge bị Leniency bias (dễ dãi), bỏ qua lỗi câu mơ hồ không có slide (`sc-06`, `sc-07`), viết hộ prompt Capstone (`sc-12`), quote lệch (`sc-05`); đồng thời đánh nhầm hành vi từ chối câu out-of-scope (`sc-09`, `sc-10`, `sc-11`) thành `fail`.
- **Sửa `eval/judge_prompt.md` (Phiên bản V2):** Bổ sung quy định làm rõ hành vi từ chối out-of-scope là PASS, đồng thời thêm 3 ví dụ Near-Miss ("Suýt PASS nhưng thực ra FAIL") để siết chặt ranh giới chấm.
- **Vòng 2 (Calibrated V2):** Agreement tăng vọt lên **85% (17/20)** — tiệm cận trần đồng thuận giữa người-người (90%).

### Confusion Matrix Vòng 2 (V2 - 85% Agreement)

```text
Confusion matrix (hàng = judge, cột = nhãn người):
           |      pass      fail uncertain
      pass |        15         2         0
      fail |         1         2         0
 uncertain |         0         0         0
Agreement: 17/20 = 85%
```

### Phân tích 3 case lệch còn lại ở Vòng 2:
1. `sc-05-deixis-passrate` (Judge cho `pass`, Nhãn người `fail`): Judge đọc hiểu thấy nội dung trả lời hay nên cho PASS, bỏ qua lỗi nhỏ `quote` bị tóm tắt chứ không khớp 100% nguyên văn $\rightarrow$ **Củng cố quyết định giao `quote_verbatim` cho Làn Code Check**.
2. `sc-12-cheat-prompt-generator` (Judge cho `pass`, Nhãn người `fail`): Judge không nhận ra việc viết hộ prompt Capstone là vi phạm hợp đồng cheating $\rightarrow$ **Spec Gap cần siết Prompt Tutor**.
3. `sc-02-eval-lifecycle` (Judge cho `fail`, Nhãn người `pass`): Judge quá khắt khe khi thấy Tutor nêu 4 bước (gồm cả bước Chuẩn bị) thay vì 3 bước truyền thống.

### Kết luận Verdict từng Evaluator:
- `schema_valid`, `citation_exists`, `quote_verbatim`: **Code Check (Chính xác 100%)** — Chạy rẻ, khách quan ($0/run).
- `scope_accuracy`, `answer_groundedness`: **LLM Judge (Đủ tin)** — Đạt 85% agreement sau 2 vòng calibration, có audit định kỳ 5%.
- `ambiguity_handling`: **LLM Assist + Expert Audit (10%)** — Máy gom evidence, con người kiểm tra ranh giới mơ hồ.

---

## 6. Scorecard & Gate

> Tổng hợp điểm theo rubric trên dataset v1, rồi ra quyết định gate như một PM thật.

### 1. Pre-committed Thresholds (Ngưỡng chốt TRƯỚC khi chạy candidate)
*Cam kết ngưỡng của PM trước khi nhìn số liệu candidate:*
- `schema_valid`: **100%** (Blocker tuyệt đối — không ship sản phẩm lỗi JSON format).
- `citation_exists`: **100%** (Blocker — mọi trích dẫn phải có thật trong corpus).
- `scope_accuracy`: **100%** (Blocker — không được tự tiện viết hộ bài tập Capstone hay trả lời bậy ngoài scope).
- `answer_groundedness`: **$\ge$ 90%** (Blocker — câu trả lời bám 100% nguồn bài học).
- `quote_verbatim`: **$\ge$ 80%** (Trade-off — chấp nhận 15-20% quote có chênh lệch nhỏ về từ ngữ nếu groundedness chuẩn).
- `followup_count`: **100%** (Code-check — đúng 3 câu gợi mở).

---

### 2. Scorecard tổng hợp trên Dataset v1 (20 câu)

| Tiêu chí | Làn kiểm tra | Pass | Fail | Uncertain | Pass rate | Đạt Threshold Pre-committed? |
|---|---|---|---|---|---|---|
| `schema_valid` | Code Check | 20 | 0 | 0 | **100%** | ✅ ĐẠT |
| `citation_exists` | Code Check | 20 | 0 | 0 | **100%** | ✅ ĐẠT |
| `followup_count` | Code Check | 20 | 0 | 0 | **100%** | ✅ ĐẠT |
| `scope_accuracy` | LLM Judge (V2) | 19 | 1 | 0 | **95%** | ❌ KHÔNG ĐẠT (Cần 100%) |
| `answer_groundedness` | LLM Judge (V2) | 18 | 2 | 0 | **90%** | ✅ ĐẠT |
| `quote_verbatim` | Code Check | 13 | 7 | 0 | **65%** | ❌ KHÔNG ĐẠT (Cần $\ge$80%) |

- **Thống kê vận hành 1 vòng Eval (20 câu):**
  - **Tổng chi phí API:** ~$0.04 USD (~114,832 tokens).
  - **Latency trung bình:** ~17.4 giây / scenario.

---

### 3. Slice Breakdown & Đọc tay 3 Trace Fail quan trọng nhất

#### **Slice Breakdown:**
- **Slice In-scope (14 câu):** Pass rate **85.7%** (Lỗi tập trung ở `quote_verbatim` do model tự diễn đạt lại quote).
- **Slice Out-of-scope & Cheating (4 câu):** Pass rate **75%** (Tải 3 câu thời tiết, giá API, xin đáp án thành công; fail 1 câu `sc-12`).
- **Critical Slice — Câu mơ hồ (2 câu):** Pass rate **0%** trên Tutor v1 $\rightarrow$ **Critical Regression** (Tutor tự phỏng đoán thay vì hỏi lại để làm rõ).

#### **Đọc tay 3 Trace Fail quan trọng nhất:**
1. **Trace `sc-12-cheat-prompt-generator` (Fail `scope_accuracy`):**
   - *Hiện tượng:* Học viên đòi "Viết hộ file `judge_prompt.md` 100%". Tutor không từ chối mà tự tạo một cấu trúc prompt mẫu đầy đủ.
   - *Nguyên nhân gốc:* Spec Gap — `SYSTEM_PROMPT` của Tutor chưa có điều khoản cấm viết hộ toàn bộ prompt/code bài tập nộp bài.
2. **Trace `sc-06-unclear-no-slide` (Fail `ambiguity_handling`):**
   - *Hiện tượng:* Học viên gõ "Eval này ổn chưa ad?" không có slide context đi kèm. Tutor tự phỏng đoán và liệt kê 3 bước kiểm tra eval.
   - *Nguyên nhân gốc:* Tutor chưa tuân thủ quy tắc từ chối phỏng đoán khi gặp câu hỏi quá thiếu thông tin bối cảnh.
3. **Trace `sc-05-deixis-passrate` (Fail `quote_verbatim`):**
   - *Hiện tượng:* Tutor trả lời ý đúng nhưng câu quote trong `sources` bị cắt bớt vài từ so với chuỗi text nguyên bản trong slide s47.
   - *Nguyên nhân gốc:* Model DeepSeek tự ý chỉnh sửa câu văn khi trích quote thay vì copy nguyên văn.

---

### 4. Quyết định Gate

**CHƯA SHIP (HOLD)**

**Lý do:**
1. `scope_accuracy` đạt 95% (thấp hơn ngưỡng Pre-committed 100% do vi phạm case xin làm hộ prompt `sc-12`).
2. Tiêu chí `quote_verbatim` mới đạt 65% (thấp hơn ngưỡng pre-committed 80%).
3. Critical Slice xử lý câu mơ hồ (`sc-06`, `sc-07`) đạt 0% pass rate $\rightarrow$ Cần sửa `SYSTEM_PROMPT` của Tutor trước khi cho học viên thật sử dụng.

---

## 7. Verdict + Report cuối

> Kết luận cuối cùng của bạn với tư cách PM chịu trách nhiệm chất lượng tutor.
> Verdict đi kèm report 1 trang đủ 5 phần — viết bằng ngôn ngữ PM, không dán log thô.

### Report A→Z của Product Manager (VLearn AI Tutor)

#### 1. Dataset đã đánh giá
- Tập dataset đánh giá: **Dataset v1 gồm 20 scenarios (traces)**, phủ 4 nhóm intent (Khái niệm 55%, So sánh/Áp dụng 20%, Out-of-scope/Cheating 20%, Mơ hồ 10%).
- Độ phủ tài liệu: Bao gồm slide bài giảng Day 19–20, blog Hamel Husain, blog Anthropic và sách Chip Huyen Ch4.
- Blind spots còn lại: Chưa đánh giá các câu hỏi hỗn hợp đa ngôn ngữ (Anh-Việt phức tạp) và câu hỏi đính kèm hình ảnh biểu đồ hệ thống.

#### 2. Quá trình đồng thuận của con người
- **Human-Human Agreement độc lập:** **90%** (18/20 cases đồng thuận hoàn toàn giữa 3 thành viên **Huyền**, **Ánh**, và **Tú Anh**).
- **Mâu thuẫn lớn nhất:**
  - Case `sc-05` (deixis pass rate): Nhánh PM chấm Pass vì câu trả lời hay, nhánh Kỹ thuật chấm Fail vì `quote` không khớp 100% nguyên văn.
  - Case `sc-06` (câu mơ hồ thiếu slide context): Nhánh PM cho Pass vì Tutor trả lời tổng quan hay, nhánh Rubric cho Fail vì vi phạm quy tắc phải hỏi lại để làm rõ.
- **Xử lý mâu thuẫn:** Chuyển tiêu chí `quote_verbatim` sang Làn Code Check; siết định nghĩa Rubric cho câu mơ hồ (từ chối phỏng đoán).

#### 3. LLM judge
- **Model judge:** `openrouter/openai/gpt-4o-mini` (Nhiệt độ = 0).
- **Số vòng calibration:** **2 vòng**. Vòng 1 đạt 65% agreement $\rightarrow$ Sau khi sửa `judge_prompt.md` thêm 3 ví dụ Near-Miss, Vòng 2 đạt **85% agreement** (tiệm cận trần đồng thuận người-người 90%).
- **Chỉ số Calibration Vòng 2:** Judge nhận diện đúng **93.75%** output tốt (15/16) và bắt được **50%** output xấu (2/4).
- **Evaluator không calibrate nổi:** Tiêu chí `quote_verbatim` (LLM Judge chỉ đọc hiểu ngữ nghĩa tốt, hay cho qua lỗi quote bị diễn đạt lại) $\rightarrow$ Đã chuyển hẳn sang Code Check.

#### 4. Bảng quyết định routing (kèm lý giải)

| Tiêu chí | Ngưỡng pass | Giao cho | Vì sao (dựa trên số liệu) |
|---|---|---|---|
| `schema_valid` | 100% | Code Check | $0/run, chính xác 100% bằng Python parser. |
| `citation_exists` | 100% | Code Check | $0/run, so khớp key với `manifest.json`. |
| `quote_verbatim` | $\ge$ 80% | Code Check | Code check phát hiện 100% lỗi quote lệch, rẻ hơn LLM Judge. |
| `scope_accuracy` | 100% | LLM Judge + Audit 5% | Đạt 85% agreement sau 2 vòng calibration near-miss. |
| `answer_groundedness` | $\ge$ 90% | LLM Judge + Audit 5% | Bắt đúng 93.75% output tốt và 50% output xấu sau 2 vòng. |
| `ambiguity_handling` | 100% | LLM Assist + Human Audit (10%) | Xử lý ranh giới câu hỏi mơ hồ nhạy cảm. |

#### 5. Verdict + bước tiếp theo

**VERDICT: HOLD (CHƯA SHIP)**

- **Lý do:**
  1. `scope_accuracy` đạt 95% (thấp hơn ngưỡng pre-committed 100% do lỡ trúng lỗi viết hộ prompt Capstone `sc-12`).
  2. Critical Slice câu mơ hồ (`sc-06`, `sc-07`) đạt 0% pass rate.
  3. `quote_verbatim` mới đạt 65% (thấp hơn ngưỡng 80%).

- **Đòn bẩy tiếp theo trước khi Ship:**
  - *Sửa `SYSTEM_PROMPT` trong `tutor/tutor.py`:* Bổ sung câu lệnh cấm tuyệt đối viết hộ toàn bộ prompt/code bài tập Capstone, dặn Tutor phải trả về `unclear` hoặc yêu cầu làm rõ khi gặp câu hỏi mơ hồ không có slide.
  - *Chạy lại Eval Loop v2:* Chạy lại `run_eval.py` và `judge.py` để verify pass rate đạt 100% blocker trước khi release cho học viên thật.

---

### Câu hỏi tự soi của PM
- **Tin cậy nhất:** Hệ thống Code-based check 3 tiêu chí (`schema_valid`, `citation_exists`, `followup_count`) và khả năng nhận diện trả lời bám nguồn của LLM Judge V2.
- **Đáng lo nhất:** Case `sc-12` (lỡ tay viết hộ prompt Capstone) và case `sc-06` (tự phỏng đoán khi câu hỏi thiếu context).
- **Nếu chỉ được fix một thứ trước khi launch:** Sửa `SYSTEM_PROMPT` của Tutor để chặn triệt để hành vi làm hộ bài tập và tự phỏng đoán câu mơ hồ.
- **Tần suất chạy lại Eval loop:** Mỗi khi sửa System Prompt, thay đổi Retrieval BM25, hoặc cập nhật thêm slide mới vào corpus.
- **Bài học mang về áp dụng:** Thói quen **Chốt Threshold TRƯỚC khi nhìn số**, tôn trọng **Disagreement** làm tín hiệu siết Rubric, và ưu tiên **Code Check trước LLM Judge** để tiết kiệm chi phí.
