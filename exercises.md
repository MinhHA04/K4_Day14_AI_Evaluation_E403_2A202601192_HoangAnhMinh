# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu trả lời chỉ hỏi lại để làm rõ hoặc nói “chưa đủ thông tin”, nên hầu như không có khẳng định cần đối chiếu với context. | Câu trả lời tự tạo thông tin về giá, bảo hành, thanh toán, tài khoản hoặc chính sách không có trong tài liệu. | Ưu tiên sửa ngay: buộc câu trả lời bám vào nguồn, thêm trích dẫn và fallback “không đủ thông tin/chuyển nhân viên”. |
| Answer Relevance | Câu hỏi của khách mơ hồ nên bot cần hỏi thêm một câu hoặc đưa cảnh báo cần thiết trước khi trả lời. | Khách hỏi rõ nhưng bot trả lời sai chủ đề hoặc không giải quyết nhu cầu chính. | Kiểm tra hiểu intent, viết lại prompt và bổ sung các câu hỏi cùng intent vào bộ test. |
| Context Recall | Trường hợp chỉ là chào hỏi/hỏi làm rõ, chưa cần lấy tài liệu để đưa ra câu trả lời thực tế. | Retriever bỏ sót điều kiện quan trọng như thời hạn đổi trả, ngoại lệ bảo hành hoặc bước xác minh tài khoản. | Sửa query/chunking, tăng phạm vi tìm kiếm và bổ sung tài liệu hoặc test còn thiếu. |
| Context Precision | Chunk đúng và đủ đã đứng đầu; vài chunk thừa phía sau chỉ làm tăng chi phí nhỏ và chưa ảnh hưởng câu trả lời. | Phần lớn context là nhiễu hoặc tài liệu đúng bị xếp sau, làm bot dùng nhầm chính sách/sản phẩm. | Cải thiện metadata filter và reranking; loại chunk trùng hoặc không liên quan. |
| Completeness | Câu hỏi hẹp chỉ cần câu trả lời ngắn; bot bỏ qua chi tiết tùy chọn nhưng vẫn trả lời đủ ý chính. | Bot thiếu bước bắt buộc, điều kiện, ngoại lệ, phí hoặc thời hạn khiến khách không thể thực hiện đúng. | Tách expected answer thành checklist các ý bắt buộc và yêu cầu bot kiểm tra đủ checklist trước khi trả lời. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Chuẩn bị nhiều câu hỏi, mỗi câu có hai đáp án cố định A và B, sau đó chấm trong hai conditions: **Condition 1:** đặt A trước B; **Condition 2:** đặt B trước A. Giữ nguyên nội dung, rubric, model và tham số; chỉ đổi vị trí, đồng thời ẩn tên model tạo đáp án. Chạy nhiều mẫu với thứ tự được phân ngẫu nhiên. Nếu đáp án đứng đầu có tỷ lệ thắng cao hơn rõ rệt, hoặc judge đổi đáp án thắng chỉ vì đảo thứ tự, judge có position bias. Có thể đối chiếu thêm với human labels để biết đáp án nào thực sự tốt hơn.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Rubric phải chấm theo **ý đúng bắt buộc**, không chấm theo độ dài. Mỗi tiêu chí nên có mô tả cụ thể, ví dụ: đúng chính sách, đủ bước, liên quan và rõ ràng; quy định rõ câu ngắn nhưng đủ ý vẫn được điểm tối đa. Không cộng điểm cho việc lặp lại hoặc thêm thông tin ngoài yêu cầu, đồng thời trừ điểm phần lan man, không liên quan hay không có bằng chứng. Có thể chấm từng tiêu chí riêng rồi mới tính tổng để một đáp án dài không tạo cảm giác “tốt hơn”.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Human labels là mốc tham chiếu giúp kiểm tra LLM judge có hiểu rubric giống con người hay không. So sánh hai bên giúp phát hiện judge quá dễ, quá nghiêm hoặc thiên vị vị trí/độ dài/phong cách của chính model. Từ đó ta điều chỉnh rubric, prompt và ngưỡng pass cho phù hợp. Nên dùng nhãn đồng thuận từ ít nhất hai người chấm và kiểm tra lại định kỳ vì dữ liệu, model và chính sách có thể thay đổi.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | ≥ 0.85 | Quan trọng nhất trong hỗ trợ khách hàng: thông tin bịa về giá, bảo hành hoặc tài khoản có thể gây thiệt hại, nên dùng ngưỡng nghiêm. |
| Answer Relevance | ≥ 0.75 | Cần trả lời đúng nhu cầu chính; vẫn cho phép một ít nội dung hỏi làm rõ hoặc cảnh báo cần thiết. |
| Completeness | ≥ 0.75 | Phải có đủ các bước và điều kiện quan trọng; chi tiết phụ có thể thiếu mà chưa cần chặn release. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* **Offline evaluation** dùng trước khi deploy hoặc khi thay model, prompt, retriever để chạy lại golden dataset và regression test một cách nhanh, an toàn, lặp lại được. **Online evaluation** dùng sau khi deploy để theo dõi dữ liệu thật, drift, latency, tỷ lệ fallback, phản hồi người dùng và A/B test; không gửi dữ liệu nhạy cảm vào hệ thống chấm nếu chưa được bảo vệ. **Human review** dùng cho trường hợp mơ hồ, điểm sát ngưỡng, khi các metric mâu thuẫn, hoặc nội dung rủi ro cao như thanh toán, quyền riêng tư và khiếu nại; đồng thời dùng định kỳ để tạo nhãn chuẩn và calibrate LLM judge. Quy trình hợp lý là: offline gate trước release → online monitoring sau release → human review cho mẫu rủi ro và các lỗi cần điều tra.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
