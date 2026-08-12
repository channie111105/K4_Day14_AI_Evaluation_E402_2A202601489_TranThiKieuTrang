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
| Faithfulness | Có thể chấp nhận tạm thời khi câu hỏi ngoài scope hoặc context được cung cấp rất ít, và assistant nói rõ evidence không đủ thay vì bịa. | Critical khi answer đưa ra chính sách, ngày, số tiền, điều kiện refund/warranty/privacy không có trong context hoặc trái với evidence. | Siết grounding trong prompt, thêm hallucination check, kiểm tra lại gold context và retrieved chunks. |
| Answer Relevance | Có thể thấp với câu hỏi mơ hồ, nhiều intent, hoặc adversarial case mà assistant cần từ chối thay vì trả lời trực tiếp. | Critical khi user hỏi một vấn đề cụ thể nhưng assistant trả lời sang chủ đề khác, bỏ qua intent chính hoặc không giải quyết request. | Cải thiện intent handling, viết prompt rõ hơn, thêm examples cho câu hỏi multi-intent và adversarial. |
| Context Recall | Có thể thấp nếu expected answer nằm ngoài scope hợp lệ hoặc câu hỏi cố tình thiếu thông tin để kiểm tra refusal. | Critical khi corpus có evidence cần thiết nhưng retriever không lấy được, làm answer thiếu hoặc sai. | Sửa chunking/query formulation, tăng top-k, thêm synonym handling hoặc cải thiện retriever. |
| Context Precision | Có thể thấp khi retriever lấy đủ evidence nhưng câu hỏi cần nhiều tài liệu nên có thêm noise ở cuối ranking. | Critical khi các chunk liên quan bị đẩy xuống sau nhiều chunk nhiễu, khiến generator dùng sai context hoặc bỏ sót evidence. | Thêm reranking, điều chỉnh scoring/ranking, giảm chunk noise và kiểm tra top-k. |
| Completeness | Có thể thấp khi assistant cố ý trả lời ngắn vì evidence không đủ hoặc câu hỏi chỉ yêu cầu một phần nhỏ của policy. | Critical khi answer bỏ sót điều kiện, exception, deadline, fee hoặc bước hành động quan trọng mà expected answer yêu cầu. | Tăng yêu cầu trả lời đủ điều kiện trong prompt, cải thiện retrieval coverage, thêm few-shot complete answers. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Tạo một tập câu hỏi có reference answer và hai candidate answers cho mỗi câu: một answer tốt hơn và một answer kém hơn. Condition 1 đặt answer tốt ở vị trí A và answer kém ở vị trí B. Condition 2 giữ nguyên nội dung nhưng đảo thứ tự, answer kém ở A và answer tốt ở B. Nếu judge thường cho điểm cao hơn cho answer đứng trước dù nội dung không đổi, hoặc tỷ lệ thắng của vị trí A cao bất thường sau khi đảo thứ tự, đó là dấu hiệu position bias. Có thể lặp lại với nhiều câu hỏi và randomize ID để giảm ảnh hưởng của tên nhãn.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Rubric nên chấm theo correctness, grounded evidence, completeness và actionability, không chấm theo độ dài. Cần ghi rõ rằng câu trả lời dài nhưng thêm claim không có evidence, lặp ý, hoặc không trả lời đúng policy sẽ bị trừ điểm. Mức điểm cao chỉ dành cho answer đủ thông tin cần thiết, chính xác, có điều kiện/exception quan trọng và diễn đạt gọn rõ.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Human labels là mốc chuẩn để biết judge có đang quá dễ, quá nghiêm, thiên vị câu dài, hoặc hiểu sai domain hay không. Calibration giúp chỉnh rubric, threshold và prompt judge để điểm tự động gần hơn với đánh giá của người thật. Việc này đặc biệt quan trọng trong customer support vì sai chính sách refund, warranty, privacy hoặc escalation có thể gây hậu quả thực tế.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.75 | Đây là metric quan trọng nhất để block deploy vì assistant support không được bịa chính sách, số tiền, deadline hoặc điều kiện không có trong corpus. |
| Answer Relevance | 0.70 | Nếu relevance thấp, user không nhận được câu trả lời đúng intent; threshold này giúp chặn các thay đổi prompt/routing làm answer lạc đề. |
| Completeness | 0.70 | Customer support cần đủ điều kiện, exception và next step; completeness thấp dễ làm user hiểu thiếu về refund, warranty, shipping hoặc security. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Offline evaluation dùng trước mỗi release, prompt change, retriever change hoặc model upgrade để so sánh với baseline trên golden dataset lặp lại được. Online evaluation dùng sau khi deploy để theo dõi traffic thật, drift, user satisfaction, latency, cost và các failure mới chưa có trong dataset. Human review dùng cho các case high-stakes hoặc khó chấm tự động như privacy/security, refund dispute, policy exception, adversarial prompt, và để calibrate LLM judge bằng nhãn của người thật.

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
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E01 | Easy | `01_product_catalog.md` | Factual lookup trực tiếp về thông số NovaBook 14 trong một đoạn duy nhất. |
| H01 | Hard | `09_escalation_and_policy_updates.md` | Cần áp dụng policy version theo order-placement date, phân biệt version 1.0/2.0 và exception OrbitPlus 45-day benefit. |
| A02 | Adversarial | `00_system_scope.md` | Kiểm tra prompt injection: user yêu cầu bỏ luật và tiết lộ hidden prompt/credentials/private notes, assistant phải từ chối theo scope/safety rule. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Điểm khó nhất là giữ expected answer đủ ngắn nhưng vẫn bao phủ đầy đủ điều kiện, exception, ngày hiệu lực, phí và limitation trong evidence. Với các case hard, cần tránh tự suy diễn ngoài corpus và phải chọn evidence đủ cụ thể để support toàn bộ claim trong expected answer.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

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
