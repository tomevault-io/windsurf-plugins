---
trigger: always_on
description: Hướng dẫn chung cho AI agent khi làm việc trong một repository code.
---

# AGENTS.md

Hướng dẫn chung cho AI agent khi làm việc trong một repository code.

## Mục tiêu

- đọc trước khi sửa
- không đoán mò
- thay đổi tối thiểu
- verify trước khi kết luận
- báo rõ phần đã làm, đã kiểm tra, và phần còn chưa chắc chắn

Repository instruction, system instruction, developer instruction, và user instruction luôn có độ ưu tiên cao hơn file này.

## Quy ước ngôn ngữ

- Luôn giao tiếp với người dùng bằng tiếng Việt có dấu, trừ khi người dùng yêu cầu ngôn ngữ khác.
- Khi viết nội dung hiển thị cho người dùng, helper text, label, placeholder, thông báo lỗi, mô tả tính năng hoặc tài liệu nội bộ: ưu tiên tiếng Việt có dấu, rõ ràng, tự nhiên.
- Chỉ dùng không dấu khi thật sự bắt buộc cho dữ liệu kỹ thuật như slug, tên file, key cấu hình, tên biến, command, hoặc chuỗi hệ thống không hỗ trợ dấu.

## Quy tắc bắt buộc trước khi làm code

### 1. Bắt buộc đọc skill trước khi làm tác vụ code không tầm thường

Mọi tác vụ code không tầm thường đều phải đọc và áp dụng skill `buff-nao-gpt` trước khi tiếp tục.

Phạm vi áp dụng bao gồm:

- đọc và phân tích code
- tìm file liên quan
- debug
- review
- sửa bug
- thêm tính năng
- refactor
- thay đổi API, validation, upload, admin action, deploy flow, data flow, hoặc logic nghiệp vụ

Trình tự này là bắt buộc:

1. đọc file skill `buff-nao-gpt`
2. xác nhận trong câu đầu tiên của phần làm việc rằng skill đang được áp dụng
3. rồi mới được đọc file code, sửa code, chạy verify

Nếu chưa đọc skill thì:

- không được suy luận từ trí nhớ
- không được sửa code
- không được review code
- không được đề xuất bản vá như thể đã nắm rõ hệ thống

Nếu không tìm thấy skill hoặc không đọc được skill:

- phải dừng ngay
- phải báo đây là blocker
- không được tự ý dùng workflow thay thế

Yêu cầu về thông điệp đầu tiên cho tác vụ code:

- phải nói rõ đang áp dụng `$buff-nao-gpt`
- phải xuất hiện trước mọi phân tích kỹ thuật đáng kể

Nếu vi phạm mục này thì toàn bộ phần làm việc sau đó được xem là không tuân thủ instruction của repository.

### 2. Quy trình làm việc chuẩn cho tác vụ code

Làm theo thứ tự:

1. đọc skill bắt buộc
2. nói rõ đang áp dụng skill
3. đọc file liên quan
4. xác định đúng đường lỗi hoặc mục tiêu thay đổi
5. chọn thay đổi nhỏ nhất nhưng an toàn
6. sửa code
7. verify bằng cách sát lỗi nhất có thể
8. chỉ kết luận sau khi đã verify

## Quy tắc sửa code

- ưu tiên sửa nguyên nhân gốc thay vì vá bề mặt
- mọi dòng thay đổi phải bám trực tiếp vào yêu cầu
- không mở rộng phạm vi nếu không cần thiết
- không refactor phần bên cạnh chỉ vì thấy chưa đẹp
- không đổi tên, đổi style, hoặc dọn dẹp ngoài phạm vi
- không thay đổi public contract nếu người dùng không yêu cầu
- không thêm dependency mới nếu chưa thật sự cần

## Quy tắc verify

- luôn verify trước khi nói là đã xong
- ưu tiên cách kiểm tra nhỏ và sát nhất:
- test lỗi cụ thể
- test file liên quan
- lint hoặc typecheck phần bị chạm tới
- build nếu thay đổi ảnh hưởng UI, asset, hoặc hành vi public
- nếu không thể verify thì phải nói rõ phần nào chưa verify

## Quy tắc đồng bộ runtime

- Nếu thay đổi chỉ có hiệu lực sau khi `khởi động lại dự án`, `restart dev server`, `chạy migration`, `xóa cache`, hoặc `reload bundle`, thì bắt buộc phải nhắc rõ người dùng thực hiện bước đó.
- Không được giả định người dùng đã tự restart hoặc môi trường đã tự đồng bộ.
- Khi nghi ngờ lỗi đến từ việc code và runtime chưa khớp nhau, phải ưu tiên kiểm tra khả năng này trước khi suy luận sâu hơn.
- Nếu vừa sửa code vừa biết chắc cần restart để nhìn thấy kết quả, phải nói rõ ngay trong phần kết luận để tránh đoán mù và hiểu nhầm.

Nếu verify thất bại, phải phân loại rõ:

- do thay đổi vừa tạo ra
- lỗi có sẵn
- lỗi không liên quan
- chưa đủ dữ liệu để kết luận

## Quy tắc an toàn

- không tự ý chạy hành động phá hủy dữ liệu
- không tự ý reset, revert, hoặc overwrite thay đổi của người dùng
- không commit secret, token, password, file môi trường, file runtime, backup, cache, hoặc log
- với tác vụ liên quan auth, upload, admin action, external integration, hoặc dữ liệu nhạy cảm: luôn kiểm tra phía server, validation, và rủi ro rò rỉ dữ liệu

## Quy tắc kiểm tra UI

- ưu tiên đọc source, CSS, cấu trúc component, và logic render trước khi mở browser automation
- không kiểm tra mù bằng browser khi source đã trả lời được câu hỏi
- chỉ dùng browser automation khi thật sự cần quan sát hành vi runtime mà terminal không xác minh được


## Quy tắc làm UI theo chuẩn production

Áp dụng bắt buộc cho mọi tác vụ liên quan giao diện, layout, CSS, component, responsive, dashboard, form, landing page, admin page, bảng dữ liệu, modal, toast, navigation, hoặc trải nghiệm người dùng.

### 1. Nguyên tắc trả lời khi làm UI

- Không giải thích dài dòng về lý thuyết thiết kế nếu người dùng không hỏi.
- Không viết nhiều phương án lan man. Chọn một phương án tốt nhất, phù hợp nhất với codebase hiện tại, rồi thực hiện.
- Không báo cáo bằng các bảng dài nếu không cần thiết.
- Không mô tả lại những việc hiển nhiên như “đã căn chỉnh giao diện cho đẹp hơn” nếu không nêu được thay đổi cụ thể.
- Khi kết thúc, chỉ báo cáo ngắn theo format `Changed / Verified / Notes`.
- Nếu cần nêu lý do thiết kế, viết tối đa 3 gạch đầu dòng, tập trung vào tác động thực tế: dễ nhìn hơn, ít rối hơn, responsive tốt hơn, đúng luồng người dùng hơn.

### 2. Chuẩn giao diện mong muốn


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thuanphanmem/khoa-ung-dung-android-tu-xa](https://github.com/thuanphanmem/khoa-ung-dung-android-tu-xa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
