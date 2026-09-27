---
trigger: always_on
description: > File này là luật làm việc cho AI coding agents trong repo.
---

# AGENTS.md — AI Coding Agent Rules

> File này là luật làm việc cho AI coding agents trong repo.  
> Mục tiêu: giúp agent triển khai đúng scope v1, không tự mở rộng sản phẩm, không tạo feature ngoài kế hoạch, không vi phạm guardrails bảo mật.

---

# 1. Tài liệu phải đọc trước khi code

Trước khi implement bất kỳ task nào, agent phải đọc theo thứ tự:

```text
1. docs/PRD.md
2. docs/ARCHITECTURE.md
3. docs/SECURITY_GUARDRAILS.md
4. docs/ACCEPTANCE_CRITERIA.md
5. docs/WORKER_SPEC.md
6. docs/LLM_PROVIDER_SPEC.md
7. docs/PRODUCTION_READINESS.md
```

Nếu nội dung các file mâu thuẫn nhau, ưu tiên theo thứ tự:

```text
SECURITY_GUARDRAILS.md
→ ACCEPTANCE_CRITERIA.md
→ PRD.md
→ ARCHITECTURE.md
→ WORKER_SPEC.md
→ LLM_PROVIDER_SPEC.md
→ PRODUCTION_READINESS.md
```

---

# 2. Scope v1 bắt buộc tuân thủ

V1 chỉ làm:

```text
Authorized external web/app security testing cho domain đã xác minh.
```

Cụ thể:

```text
- Domain verification.
- Scope authorization.
- User-selected Target Type, Surface Flags, Test Intensity Mode.
- Deterministic Scan Plan from package tier, target type, surface flags, auth scope, intensity, verified scope, policy gates, approval gates, and quota.
- Optional test account.
- Browser Inspector bằng Playwright/CDP.
- ZAP passive/baseline signal.
- Nuclei curated safe templates.
- OpenHack-style Free Hunter workflow.
- Strix attacker-mindset reasoning.
- Recon signal layer (subfinder/dnsx/httpx/katana): passive discovery, scope-gated active probe only. Có trong local Compose và developer tooling; staging worker overlay/cloud build chưa có R. Xem docs/ARCHITECTURE.md.
- Human-readable report.
- AI/dev-readable report.
- Finding board.
- Manual retest từng finding.
- User Approval Gate cho action nhạy cảm.
```

Không tự thêm các phần sau nếu chưa có yêu cầu rõ:

```text
- GitHub repo access.
- Jira / Linear integration.
- CI/CD-based automated retesting.
- Deployment-triggered retest.
- VPS / instance verification.
- Cloud account verification.
- Private network scan.
- Server agent.
- SAST / SCA / secrets scanning.
- Mobile APK audit.
- DefectDojo production integration.
- secureCodeBox / Kubernetes orchestration phức tạp.
```

---

# 3. Nguyên tắc không tự mở rộng sản phẩm

Agent không được tự thêm feature lớn chỉ vì “có vẻ hợp lý”.

Không được tự ý thêm:

```text
- Integration mới.
- Worker mới ngoài plan.
- OAuth provider mới.
- Billing model mới.
- Report format mới ngoài yêu cầu.
- Scan mode mới.
- Admin/enterprise feature ngoài scope.
```

Nếu phát hiện một feature có vẻ cần thiết nhưng chưa có trong tài liệu, hãy ghi:

```text
TODO: cần owner xác nhận trước khi triển khai.
```

Không tự implement.

---

# 4. Quy tắc bảo mật bắt buộc

## 4.1. Domain và scope

Không được tạo scan job nếu:

```text
- Domain chưa verified.
- Verification đã expired.
- User chưa tạo scan authorization.
- Target nằm ngoài allowed_hosts.
- URL redirect ra ngoài scope.
- Target là private/local/metadata IP.
```

## 4.2. Secret và credential

Không được:

```text
- Log raw password.
- Log raw token.
- Log raw cookie.
- Log raw API key.
- Ghi raw credential vào report.
- Đưa raw credential vào LLM prompt.
- Lưu credential dạng plain text.
```

Credential phải được mã hóa.

## 4.3. AI / Strix

Strix được dùng để suy luận attacker-mindset, nhưng mọi action nhạy cảm phải qua Product Policy Gate.

Không được để Strix:

```text
- Scan ngoài scope.
- Tự ý gọi destructive endpoint.
- Tự ý dùng credential raw.
- Tự ý tạo exploit hướng dẫn lạm dụng.
- Tự ý vượt runtime/tool/token budget.
```

## 4.4. Retest

Retest trong v1 là manual.

Không được implement:

```text
- CI/CD-based automated retesting.
- Deployment-triggered retest.
- Background retest sau mỗi deploy.
```

Retest phải:

```text
- Gắn với một finding cụ thể.
- Chạy scenario hẹp.
- Kiểm tra scope trước khi chạy.
- Có User Approval Gate nếu action nhạy cảm.
```

---

# 5. Quy tắc worker

Mọi worker phải có:

```text
- timeout
- retry limit
- structured logs
- scan_id
- project_id
- worker_type
- error handling
- audit event nếu action quan trọng
```

Worker không được chạy vô hạn.

Worker không được silently fail.

Worker không được ghi fake success khi tool thật fail.

---

# 6. Quy tắc evidence và report

Mọi evidence đưa vào report phải qua sanitizer.

Report không được chứa:

```text
- raw password
- raw cookie
- raw token
- raw API key
- PII chưa được sanitize
- raw request/response nhạy cảm
```

Nếu không phát hiện lỗi nghiêm trọng, không ghi:

```text
Không phát hiện lỗi.
```

Phải ghi rõ:

```text
Không phát hiện Critical/High trong phạm vi kiểm thử hiện tại.
```

Và kèm:

```text
- đã kiểm tra gì
- coverage
- giới hạn kiểm thử
- phần chưa kiểm tra được
- bước tiếp theo
```

---

# 7. Quy tắc Free Hunter

Free Hunter không phải bản yếu và không phải pentest đầy đủ.

Free dùng cùng Target Type, Surface Flags và Test Intensity Mode model như paid nếu user đủ điều kiện và chấp nhận rủi ro, nhưng bị giới hạn:

```text
- max_returned_findings = 1
- max_monitored_findings = 1
- max_retests = 1
- cooldown_days = 7
```

Free phải dừng sau first valuable finding. Nếu không có valuable finding trong budget, trả coverage report + hardening + limitations.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LumosLab-Innovation/OpenHunterAI](https://github.com/LumosLab-Innovation/OpenHunterAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
