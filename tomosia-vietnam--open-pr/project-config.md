---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo là gì

Claude Code **plugin** tên `open-pr` — 2 slash command: `/open-pr:review <PR_URL>` review PR GitHub
đa stack (Rails, Vue, React, Python, Node.js, Lambda, PHP, Laravel, WordPress, Shell, Makefile), tự
học convention riêng theo từng repo được review, post kết quả (summary + inline line-by-line) trực
tiếp lên PR qua `gh api`; và `/open-pr:fix <PR_URL>` dev-facing, đọc đúng finding
`/open-pr:review` đã để lại, tự fix code đúng convention dự án, commit/push có kiểm soát, reply lại
PR.

Không có build/lint/test — toàn bộ plugin là markdown (command + template nội dung) và 1 file JSON
cấu hình. Không có runtime code riêng của repo này để chạy/test độc lập; cách "chạy thử" là cài
plugin vào Claude Code rồi gọi `/open-pr:review <PR_URL>` thật trên 1 repo khác.

## Cấu trúc

Sản phẩm (những gì `${CLAUDE_PLUGIN_ROOT}` trỏ tới lúc runtime) nằm trong `src/` — `src/` CHÍNH LÀ
plugin root thật (có `.claude-plugin/plugin.json` riêng), không phải repo root. Nhờ vậy lúc
`/plugin install`, Claude Code chỉ copy đúng `src/` vào plugin cache — README, CLAUDE.md,
backlogs/, scripts/ ở repo root (phục vụ phát triển repo này) không lọt vào máy user cài plugin.

```
.claude-plugin/marketplace.json  Marketplace tự host (source: "./src") — chỉ copy `src/` vào
                               plugin cache lúc install, không phải cả repo root
src/.claude-plugin/plugin.json   Metadata plugin (name: "open-pr", trỏ commands: "./commands/" — path
                               tính từ root MỚI là src/, không phải repo root)
scripts/reinstall.sh          Script dev: uninstall/re-add marketplace/install lại (đọc tên qua
                               2 manifest trên, không đụng nội dung src/)
CLAUDE.md                     File này
backlogs/*.md                 Task breakdown lịch sử khi build plugin lần đầu (tạm, sẽ xoá sau
                               khi xong dự án — không phải doc vận hành runtime)
.gitignore                    Dev repo, không liên quan runtime plugin

src/commands/review.md        Slash command DUY NHẤT /open-pr:review — **thin orchestrator**: mindset +
                               xương quy trình + invariant cứng (giọng imperative ngắn). Không nhồi
                               chú thích "vì sao / đã bug thật" (chúng nằm ở file này, mục D dưới).
                               Chi tiết detect-stack → `src/stack-detection.md`; setup →
                               `src/setup-flow.md`; nhánh theo PR → `src/cases/` (hard gate). CRITICAL
                               + allowed-tools: chỉ review/comment (+ 1 review submodule khi case áp
                               dụng); `gh pr view/diff/checkout/checks`, `gh api` (scope theo path
                               cụ thể — reviews/comments/replies/reactions/files/user/graphql, không
                               còn `gh api:*` chung), `git init`,
                               `git -C notebooks/review:*` (chỉ `add`/`commit`, không subcommand
                               khác), `git fetch`,
                               `git worktree add notebooks/review/*/worktrees/*`,
                               `cd notebooks/review/*/worktrees/* && gh pr checkout`,
                               `git -C notebooks/review/*/worktrees/* submodule update`, `cp`,
                               `mkdir`, `Agent`, `Read`, `Grep`, `Write`, `Edit` — không
                               `gh pr close/merge`, không `git push/branch -D/reset --hard`, không
                               `git branch`/`git checkout` trần
src/commands/fix.md   Slash command THỨ HAI /open-pr:fix <PR_URL> — dev-facing, SỬA
                               CODE THẬT tại pwd hiện tại (KHÔNG qua worktree, khác review.md).
                               Đọc finding review.md để lại trên 1 PR, tự quyết fix/decline theo
                               severity, commit/push có kiểm soát, reply lại đúng thread/issue. Verify
                               remote+branch+branch-bảo-vệ ở đầu lệnh, DỪNG NGAY nếu sai. Bootstrap
                               setting riêng `fix-meta.json` (sibling `meta.json`, không
                               chung field). allowed-tools: `gh pr view`, `gh api` scope path cụ thể
                               (comments/reviews/graphql/user GET, POST đúng reply LINE-level +
                               comment OVERVIEW-level, KHÔNG POST reviews), `git remote`,
                               `git branch --show-current`, `git add/commit/push` (không `-A`, không
                               `--amend`, không `--force`), `Read`, `Grep`, `Write`, `Edit`, `Agent`
                               — không `gh pr checkout`, không `git worktree`, không
                               `gh pr close/merge/reopen`, không `git branch -D/reset --hard`
src/stack-detection.md        KHÔNG phải slash command. Bảng mapping đuôi file/path → stack +
                               overlay rule; `review.md` Bước 2 đọc bằng Read
src/setup-flow.md             KHÔNG phải slash command (có ý — xem bên dưới). `review.md` chỉ đọc
                               file này bằng tool Read khi repo CHƯA thiết lập xong, để không tốn

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TOMOSIA-VIETNAM/open-pr](https://github.com/TOMOSIA-VIETNAM/open-pr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
