# Báo cáo Lab: Self evolving Agentic

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| Đỗ Thành Đạt | 2A202602874 | 100% |

- Nhà cung cấp và mô hình (`LAB_MODEL`, không ghi khóa API), nhiệt độ (`LAB_TEMPERATURE`), `recursion_limit`: `deepseek-chat` (DeepSeek API qua OpenAI-compatible endpoint), `LAB_TEMPERATURE=0`, `recursion_limit=40`
- Phiên bản Deep Agents (`pip show deepagents`), hệ điều hành, chạy trực tiếp hay trong Docker: `deepagents 0.7.21`, Windows (chạy trực tiếp qua PowerShell/Python virtualenv), Python 3.11.
- Số lần chạy tác vụ đã dùng / ngân sách: Đang thực hiện
- Commit của tag `freeze`: (Chờ đóng băng ở Phần 4)

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

- H1 (subagents so với baseline): `subagents` sẽ không mang lại sự vượt trội rõ rệt về điểm số so với `baseline` trên các tác vụ đánh giá, đồng thời tiêu tốn lượng token cao hơn do overhead từ việc mở rộng system prompt và mô tả công cụ `task`. Trên các tác vụ kỹ thuật đơn lẻ, tác tử chính thường ưu tiên xử lý trực tiếp thay vì phân rã cho subagent.
- H2 (skills-auto so với baseline): `skills-auto` sẽ đạt điểm số trung bình cao hơn `baseline` trên cả tập học và tập đánh giá, chủ yếu nhờ việc khắc phục triệt để các lỗi vi phạm quy ước tổ chức (house rules) như định dạng dữ liệu, checklist deliverable, type annotations và regression tests. Tuy nhiên, mức độ cải thiện trên tập đánh giá sẽ thấp hơn tập học do xuất hiện các quy ước mới chưa từng có trong phản hồi tập học (hiện tượng khoảng cách tổng quát hóa - generalization gap).
- H3 (tác vụ học so với tác vụ đánh giá): Điểm số của `skills-auto` trên tác vụ học sẽ cao hơn đáng kể so với tác vụ đánh giá vì skill được sinh ra trực tiếp từ vết lỗi và phản hồi của tập học (in-distribution), trong khi tác vụ đánh giá chứa dữ liệu mới và các biến thể quy tắc mà tác tử chưa từng được huấn luyện qua ngữ cảnh.

## 3. Làm quen Deep Agents (Phần 0.3)

1. **Các công cụ của tác tử mặc định:** Tác tử mặc định có 9 công cụ:
   - Các công cụ quản lý tệp: `ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`.
   - Công cụ chạy lệnh shell: `execute`.
   - Công cụ quản lý đa tác tử (subagents): `task`.
   *Công cụ cho phép chạy lệnh:* `execute` (thực thi shell command trong sandbox).
2. **Mô tả của công cụ `task` về subagent `general-purpose`:**
   - `general-purpose` là agent đa năng dùng để nghiên cứu các câu hỏi phức tạp, tìm kiếm tệp và nội dung, cũng như thực thi các tác vụ nhiều bước. Nó có quyền truy cập vào tất cả các công cụ giống như tác tử chính (main agent).
   - *Ngữ cảnh mà subagent nhìn thấy:* Mỗi lần gọi subagent là **stateless** theo mặc định, nghĩa là subagent **chỉ nhìn thấy prompt/hướng dẫn mà tác tử chính truyền cho nó** (không nhìn thấy toàn bộ ngữ cảnh hội thoại trước đó của tác tử chính trừ khi được thiết lập kế thừa).
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

```text
(dán bảng ở đây)
```

## 8. Phân tích

1. So với `baseline`, điều kiện nào cải thiện điểm tác vụ **học**? Điều kiện nào cải thiện điểm tác vụ **đánh giá**? Có điều kiện nào cải thiện tác vụ học nhưng không cải thiện tác vụ đánh giá? Nếu có, đó là dấu hiệu gì?
2. Tách điểm thành check kỹ thuật và check quy ước (`rule_`). Skill do curator sinh giúp nhóm check nào? Check quy ước **mới** của tác vụ đánh giá có được skill giúp không, và vì sao?
3. Dựa vào vết và `skills_read`, giải thích một check mà skill giúp đạt và một check mà skill không giúp (skill chưa được đọc, đọc nhưng không làm theo, skill thiếu hoặc sai).
4. Chi phí: so sánh số token trung bình giữa các điều kiện. Điều kiện nào có hiệu quả tốt nhất theo điểm trên mỗi token? Đa tác tử có đáng chi phí trong thí nghiệm này không?
5. Có dấu hiệu rò rỉ dữ liệu hoặc quá khớp nào trong skill sinh ra không? Nhóm đã phòng tránh như thế nào?
6. Nhiễu: so sánh điểm tác vụ học của cùng bộ skill ở Phần 3.4 (đã sao lưu) và sau đóng băng. Chênh lệch bao nhiêu? Nó cho biết điều gì về độ tin cậy của các chênh lệch trong bảng ở mục 7?

## 9. Hạn chế và tính hợp lệ

1.
2.
3.

## 10. Kết luận

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
- Thử thách mở rộng (nếu có): hướng chọn, kết quả, nhận xét.
- Ghi chú khác:
