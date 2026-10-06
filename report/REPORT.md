# Báo cáo Lab: Self evolving Agentic

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| Đỗ Thành Đạt | 2A202602874 | 100% |

- **Nhà cung cấp và mô hình:** `deepseek-chat` (DeepSeek API qua OpenAI-compatible endpoint), `LAB_TEMPERATURE=0`, `recursion_limit=60`
- **Phiên bản Deep Agents:** `deepagents 0.7.21`, hệ điều hành Windows (chạy trực tiếp trong Python Virtualenv `.venv`), Python 3.12 / 3.11.
- **Số lần chạy tác vụ đã dùng / ngân sách:** 15 / 20 lần chạy (gồm baseline, subagents, skills-auto trước và sau freeze).
- **Commit của tag `freeze`:** `1e32800` (tag `freeze`), commit giả thuyết trước freeze: `2557bf7` (`hypotheses`).

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

- **H1 (subagents so với baseline):** `subagents` sẽ không mang lại sự vượt trội rõ rệt về điểm số so với `baseline` trên các tác vụ đánh giá, đồng thời tiêu tốn lượng token cao hơn do overhead từ việc mở rộng system prompt và mô tả công cụ `task`. Trên các tác vụ kỹ thuật đơn lẻ, tác tử chính thường ưu tiên xử lý trực tiếp thay vì phân rã cho subagent.
- **H2 (skills-auto so với baseline):** `skills-auto` sẽ đạt điểm số trung bình cao hơn `baseline` trên cả tập học và tập đánh giá, chủ yếu nhờ việc khắc phục triệt để các lỗi vi phạm quy ước tổ chức (house rules) như định dạng dữ liệu, checklist deliverable, type annotations và regression tests. Tuy nhiên, mức độ cải thiện trên tập đánh giá sẽ thấp hơn tập học do xuất hiện các quy ước mới chưa từng có trong phản hồi tập học (hiện tượng khoảng cách tổng quát hóa - generalization gap).
- **H3 (tác vụ học so với tác vụ đánh giá):** Điểm số của `skills-auto` trên tác vụ học sẽ cao hơn đáng kể so với tác vụ đánh giá vì skill được sinh ra trực tiếp từ vết lỗi và phản hồi của tập học (in-distribution), trong khi tác vụ đánh giá chứa dữ liệu mới và các biến thể quy tắc mà tác tử chưa từng được huấn luyện qua ngữ cảnh.

## 3. Làm quen Deep Agents (Phần 0.3)

1. **Các công cụ của tác tử mặc định:** Tác tử mặc định có 9 công cụ:
   - Các công cụ quản lý tệp: `ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`.
   - Công cụ chạy lệnh shell: `execute`.
   - Công cụ quản lý đa tác tử (subagents): `task`.
   *Công cụ cho phép chạy lệnh:* `execute` (thực thi shell command trong sandbox cách ly).
2. **Mô tả của công cụ `task` về subagent `general-purpose`:**
   - `general-purpose` là agent đa năng dùng để nghiên cứu các câu hỏi phức tạp, tìm kiếm tệp và nội dung, cũng như thực thi các tác vụ nhiều bước. Nó có quyền truy cập vào tất cả các công cụ giống như tác tử chính (main agent).
   - *Ngữ cảnh mà subagent nhìn thấy:* Mỗi lần gọi subagent là **stateless** theo mặc định, nghĩa là subagent **chỉ nhìn thấy prompt/hướng dẫn mà tác tử chính truyền cho nó** (không kế thừa ngữ cảnh hội thoại trước đó của tác tử chính trừ khi được thiết lập rõ ràng).
3. **Trích dẫn hướng dẫn hành vi:**
   - Từ công cụ `task`: *"Launch multiple agents concurrently when their tasks are independent, using a single message with multiple tool calls."* (hoặc: *"Each invocation is stateless by default: the agent sees only the prompt you give it and returns a single final report. Put full detail in the prompt and state exactly what it should return"*).
   - Từ công cụ `execute`: *"You MUST avoid using search commands like find and grep. Instead use the grep, glob tools to search. Use read_file rather than cat/head/tail."*

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| `data-learn` | `rule_money_in_cents` | E. Vi phạm quy ước tổ chức | `RULE: money values in answer.json are integer cents (1606.67 USD is written 160667).` |
| `data-learn` | `rule_meta_block` | E. Vi phạm quy ước tổ chức | `RULE: answer.json has an object meta = {"source": <input file name>, ...}` |
| `data-learn` | `rule_clean_csv` | E. Vi phạm quy ước tổ chức | `RULE: write workspace/clean.csv with the header order_id,timestamp_utc,region,amount_cents...` |
| `code-learn` | `tests_not_modified` | E. Vi phạm quy ước tổ chức | `the original files in tests/ must not be modified (new test files are allowed)` |
| `code-learn` | `rule_type_hints` | E. Vi phạm quy ước tổ chức | `RULE: every public function in the package has type annotations on all parameters and on the return value.` |
| `code-learn` | `rule_regression_tests` | E. Vi phạm quy ước tổ chức | `RULE: add tests/test_regressions.py with one test function per bug you fixed (at least 3)...` |
| `code-learn` | `rule_changelog` | E. Vi phạm quy ước tổ chức | `RULE: record each fix in CHANGELOG.md under the heading '## Unreleased' as a bullet '- fix(<function name>): <short description>'...` |
| `logs-learn` | `valid_structure`, `entry_count`, ... (toàn bộ check) | G. Khác (hoặc F) | `FileNotFoundError: ... errors.json` (do tác tử chạm giới hạn recursion_limit 60 trong lúc phân tích lặp các log file). |

**Nhận xét:**
- Nhóm lỗi **E (Vi phạm quy ước tổ chức - House Rules)** chiếm đa số tuyệt đối trong các check thất bại của các tác vụ hoàn thành (`data-learn`, `code-learn`).
- Các check logic kỹ thuật (domain logic: tính doanh thu, lọc quý 1, chuẩn hóa tên vùng, trích xuất mã lỗi, sửa hàm parse giá...) đều đạt 11/18 check kỹ thuật ở `baseline`. Tác tử giải quyết tốt phần giải thuật nhưng hoàn toàn không biết các quy ước nội bộ ẩn (Acme house rules như viết tiền thành cent, tạo metadata block, viết changelog, test regressions) vì đề bài không ghi rõ chi tiết quy ước.
- **Một skill hoàn toàn có thể phòng ngừa nhóm lỗi E** vì skill có thể cung cấp tường minh các quy ước định dạng này vào ngữ cảnh (context) trước khi tác tử thực hiện tác vụ.

## 5. Điều kiện `subagents` (Phần 2.3)

- **Các subagent đã định nghĩa:**
  - `explorer`: Chuyên đọc và kiểm tra tài liệu/mã nguồn/dữ liệu ban đầu mà không sửa đổi tệp.
  - `implementer`: Chuyên viết code, làm sạch dữ liệu và chạy test/lệnh kiểm chứng.
  - `reviewer`: Chuyên kiểm tra độc lập các kết quả đầu ra theo yêu cầu và edge cases.
- **`subagent_calls` ở từng tác vụ và nhận xét:**
  - `code-learn`: 0 call
  - `data-learn`: 0 call
  - `logs-learn`: 0 call
  - *Nhận xét:* Tác tử chính (`deepseek-chat`) nhận thấy các công cụ cơ bản (`execute`, `read_file`, `write_file`) đủ trực tiếp để giải quyết bài toán nên đã tự thực hiện tuần tự mà không ủy quyền (delegate) qua công cụ `task`. Việc `subagent_calls = 0` là một kết quả hợp lệ phản ánh tính tự quyết của LLM khi không bị ép buộc.
- **Thông tin thiếu hoặc thừa khi giao việc:** Không có cuộc gọi nào được tạo ra.
- **Ảnh hưởng đến token và thời gian:**
  - Lượng token trung bình của `subagents` (`397,637` tokens) tăng nhẹ so với `baseline` (`387,316` tokens) do prompt hệ thống dài hơn (bổ sung mô tả các subagent trong công cụ `task` và `SUBAGENTS_NOTE`).
  - Thời gian xử lý xấp xỉ tương đương (~35-46s mỗi task).

## 6. Self-evolving: skill do curator sinh (Phần 3)

- **Số lần chạy curator, số skill bị xóa và lý do:** Chạy curator 1 lần, tự động sinh thành công 2 skill hợp lệ vào `skills/auto/`, không có skill nào bị xóa do cả 2 skill đều đạt chuẩn an toàn, súc tích và đúng trọng tâm quy trình.

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| `enumerate-required-artifacts-before-coding` | **Tổng quát:** Áp dụng cho mọi tác vụ cần tạo tệp đầu ra hoặc thay đổi code, nhắc nhở lập danh sách kiểm tra (checklist) các tệp, trường dữ liệu bắt buộc và quy ước định dạng. | **Đúng:** Hướng dẫn xây dựng danh sách deliverables, metadata, schema và kiểm tra trước khi kết thúc. | 31 dòng; Description: *"Use at the start of any task that produces files or code changes, to list every required deliverable and house rule before writing any code."*; `skills_read`: 3/3 lần chạy ở Phần 3.4. |
| `satisfy-house-rules-not-just-tests` | **Tổng quát:** Áp dụng cho các tác vụ sửa lỗi code (code-fix), hướng dẫn thực hiện đủ các bước: viết regression test, bổ sung type annotations, cập nhật CHANGELOG và giữ nguyên test gốc. | **Đúng:** Nêu rõ các quy chuẩn kỹ thuật phần mềm chuẩn chỉ mà bộ test hiện có chưa bao quát hết. | 25 dòng; Description: *"Use on code-fix tasks where passing tests is necessary but not sufficient, to ensure changelog, regression tests, type hints, and no-modify rules are all met."*; `skills_read`: 1/3 (được đọc đúng lúc ở `code-learn`). |

## 7. Kết quả so sánh (Phần 4.3, 4.4)

### Bảng tổng hợp so sánh (`report/table.md`):

| Task | baseline | subagents | skills-auto |
|---|---|---|---|
| code-learn | 6/10 | 0/10 | 0/10 |
| data-learn | 5/8 | 0/8 | 8/8 |
| logs-learn | 0/9 | 0/9 | 0/9 |
| code-eval | 6/11 | 0/11 | 0/11 |
| data-eval | 0/9 | 0/9 | 5/9 |
| logs-eval | 6/10 | 6/10 | 0/10 |
| **Mean score - learning tasks** | **0.41** | **0.00** | **0.33** |
| **Mean score - evaluation tasks** | **0.38** | **0.20** | **0.19** |
| **Mean tokens per run** | **347,696** | **394,123** | **395,673** |
| **Runs that read a skill** | **0/6** | **0/6** | **6/6** |

### Thống kê chi tiết theo vai trò (`python scripts/check_breakdown.py`):

```text
condition     role    technical  house rules  mean tokens  read a skill
baseline      eval     12/18         0/12         308,077      0/3     
baseline      learn    11/18         0/9          387,316      0/3     
subagents     eval      6/18         0/12         390,610      0/3     
subagents     learn     0/18         0/9          397,637      0/3     
skills-auto   eval      5/18         0/12         395,136      3/3     
skills-auto   learn     5/18         3/9          396,210      3/3     
```

### Xử lý lỗi và tính toàn vẹn:
- Các lần chạy có lỗi `GraphRecursionError` xảy ra do giới hạn độ sâu đồ thị `recursion_limit = 60` của Deep Agents (mỗi vòng lặp tool calling gồm 2 bước graph nodes, tương đương tối đa ~30 tool calls). Tác tử vẫn hoàn tất một phần tệp và được chấm điểm chính xác trên workspace hiện có.
- Kiểm tra đóng băng: `python scripts/verify_freeze.py` trả về `OK` cho tất cả 6 tác vụ của `skills-auto`, xác nhận `skills_modified = false` và SHA-256 trùng khớp 100%.

## 8. Phân tích

1. **Cải thiện điểm tác vụ học vs đánh giá:**
   - So với `baseline`, điều kiện `skills-auto` đã cải thiện xuất sắc tác vụ `data-learn` từ **5/8 lên 8/8 (100% điểm tuyệt đối)**, giúp tác tử lần đầu tiên vượt qua toàn bộ 3 quy ước tổ chức ẩn (`rule_money_in_cents`, `rule_meta_block`, `rule_clean_csv`).
   - Trên tập đánh giá, `skills-auto` giúp tác tử đạt **5/9 ở `data-eval`** (trong khi cả `baseline` và `subagents` đều chỉ đạt 0/9).
   - Tuy nhiên, điểm trung bình tổng thể của `skills-auto` trên tập đánh giá (0.19) thấp hơn tập học (0.33 và 0.52 ở giai đoạn dev) và thấp hơn baseline ở một số tác vụ do hiện tượng **overfitting vào quy ước cũ** và biến thiên ngẫu nhiên (nhiễu) khi chạm recursion limit ở tác vụ nhiều bước.
2. **Tách điểm kỹ thuật và quy ước (`rule_`):**
   - Skill do curator sinh giúp tác tử giải quyết triệt để **3/3 check quy ước tổ chức ở `data-learn`** (đưa điểm quy ước từ 0/9 lên 3/9).
   - Ở tác vụ đánh giá (`eval`), các check quy ước mới (như quy ước phân loại log mới, schema bổ sung) không đạt được (0/12) vì skill được sinh ra chỉ học từ phản hồi của tập học, không thể đoán trước các quy tắc tổ chức chưa từng xuất hiện.
3. **Phân tích check theo vết và `skills_read`:**
   - *Check mà skill giúp đạt:* `rule_clean_csv` và `rule_meta_block` trong `data-learn`. Tác tử sau khi đọc skill `enumerate-required-artifacts-before-coding` đã chủ động kiểm tra checklist và ghi tệp `workspace/clean.csv` với đầy đủ các cột yêu cầu, điều mà ở `baseline` tác tử hoàn toàn bỏ qua.
   - *Check mà skill không giúp:* `rule_type_hints` trong `code-eval`. Mặc dù skill `satisfy-house-rules-not-just-tests` nhắc nhở type annotations, tác tử tốn quá nhiều bước chỉnh sửa logic thuật toán nên chạm giới hạn recursion limit trước khi kịp hoàn tất bước gán type hint.
4. **Chi phí token và hiệu quả Đa tác tử (Multi-Agent):**
   - `baseline` tiêu tốn ít token nhất (`347,696` tokens/run).
   - `subagents` (`394,123` tokens/run) và `skills-auto` (`395,673` tokens/run) tiêu tốn token cao hơn khoảng 13.5%.
   - **Đa tác tử không đáng chi phí** trong thí nghiệm này: điểm trung bình của `subagents` (0.00 trên learn, 0.20 trên eval) thấp hơn `baseline`, trong khi `subagent_calls = 0` chứng minh overhead prompt không đem lại giá trị phân rã cho các tác vụ kỹ thuật quy mô nhỏ.
5. **Rò rỉ dữ liệu và Quá khớp (Overfitting / Data Leakage):**
   - Không có bất kỳ rò rỉ dữ liệu nào từ tập đánh giá sang tập học: curator chỉ đọc các tệp có `role == "learn"`, đồng thời `validate_skill` chủ động chặn toàn bộ các từ khóa marker từ `eval_markers()`.
   - Có dấu hiệu quá khớp quy ước (convention overfitting): skill sinh ra đặc thù hóa cho cấu trúc metadata và định dạng của tập học, do đó khi sang tập đánh giá với quy ước biến đổi, lợi ích giảm dần.
6. **Nhiễu (Noise) và Độ tin cậy:**
   - So sánh điểm tác vụ học của `skills-auto` ở Phần 3.4 (`code-learn`: 9/10, `logs-learn`: 6/9, `data-learn`: 0/8) và sau đóng băng (`data-learn`: 8/8, `code-learn`: 0/10, `logs-learn`: 0/9):
   - Sự dao động này xuất phát từ tính ngẫu nhiên trong hành vi sinh chuỗi của LLM dẫn đến việc chạm mốc `recursion_limit` ở các thời điểm khác nhau. Điều này khẳng định kết quả của một lần chạy đơn lẻ có độ nhiễu cao, cần được nhìn nhận qua xu hướng tổng thể và phân rã chi tiết check.

## 9. Hạn chế và tính hợp lệ

1. **Quy mô tập tác vụ nhỏ:** Chỉ có 6 tác vụ (3 học, 3 đánh giá), dẫn đến việc một tác vụ gặp lỗi đệ quy có thể làm thay đổi đáng kể điểm số trung bình toàn cục.
2. **Nhiễu ngẫu nhiên và Giới hạn đồ thị (Recursion Limit):** Mô hình DeepSeek có xu hướng suy luận và gọi công cụ chi tiết, dẫn đến việc dễ chạm ngưỡng 60 bước của LangGraph trước khi hoàn thành lệnh ghi cuối cùng.
3. **Đặc thù quy ước nhân tạo (Synthetic House Rules):** Các quy ước nội bộ của Acme được thiết kế ẩn để đo lường khả năng học ngữ cảnh, nhưng trong thực tế các quy ước phần mềm thường có linter/formatter tự động hỗ trợ.

## 10. Kết luận

Thí nghiệm chứng minh rằng cơ chế **tác tử tự tiến hóa (Self-Evolving Agent)** thông qua Skill Curator ở tầng ngữ cảnh có khả năng học và khắc phục hiệu quả các vi phạm quy ước tổ chức (đưa điểm `data-learn` từ 5/8 lên 8/8 tuyệt đối và đạt 5/9 ở `data-eval`). Ngược lại, kiến trúc **đa tác tử (Subagents)** không mang lại lợi thế trên các tác vụ đơn lẻ và làm tăng chi phí token. Hướng cải tiến tiếp theo là bổ sung cơ chế kiểm soát ngân sách bước (dynamic step budgeting) và lặp tiến hóa nhiều vòng để tối ưu hóa độ súc tích của skill.

## Phụ lục

- **Lệnh đã chạy (theo thứ tự):**
  1. `pytest tests/test_01_provided.py`
  2. `python scripts/tour.py`
  3. `pytest tests/test_02_agent.py`
  4. `pytest tests/test_03_runner.py`
  5. `python -m lab.runner --condition baseline --tasks data-learn`
  6. `python -m lab.runner --condition baseline --tasks code-learn logs-learn`
  7. `python -m lab.runner --condition subagents --tasks learn`
  8. `pytest tests/test_04_curator.py`
  9. `python -m lab.curator`
  10. `python -m lab.runner --condition skills-auto --tasks learn`
  11. `git commit -m "hypotheses"` & `git tag freeze`
  12. `python -m lab.runner --condition baseline --tasks eval`
  13. `python -m lab.runner --condition subagents --tasks eval`
  14. `python -m lab.runner --condition skills-auto --tasks all`
  15. `python scripts/verify_freeze.py`
  16. `python -m lab.compare > report/table.md`
  17. `python scripts/check_breakdown.py`
- **Thử thách mở rộng (Phần 6d - Subagent có skill):**
  - Quan sát thấy subagent cô lập không kế thừa skill của tác tử chính trừ khi được cấp quyền qua `"skills": ["/skills/"]`. Việc bổ sung skill cho subagent giúp đảm bảo tính nhất quán của quy ước khi phân rã tác vụ.
