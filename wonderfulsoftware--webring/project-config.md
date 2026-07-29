---
trigger: always_on
description: - **Install**: `yarn` (uses yarn@1.22.19)
---

# CLAUDE.md - Agent Guide for webring

## Commands
- **Install**: `yarn` (uses yarn@1.22.19)
- **Run tests**: `npx playwright test` (in automation/e2e directory)
- **Run single test**: `npx playwright test tests/file.spec.ts`
- **Run specific test**: `npx playwright test -g "test name pattern"`

## Pull Request Review Playbook

### Mergeability Criteria (ALL three required before approve + squash-merge)

A PR is only ready to merge once **all** of the following hold:

1. **Automated validation comment is present** — the github-actions bot has posted its
   "PR validation result" comment on the PR.
   - If it is missing, add the `review` label to trigger it: `gh pr edit <PR_NUMBER> --add-label "review"`.
   - If the validation workflow fails on a transient GitHub Actions cache error
     (`BlobNotFound` / "failed to copy" from `productionresultssa11.blob.core.windows.net`),
     it is a poisoned buildkit cache, not a PR problem. A plain re-run won't fix it —
     clear the caches first (`gh cache delete --all`), then re-run the failed workflow
     (`gh run rerun <RUN_ID> --failed`). Confirm with the operator before deleting caches.
2. **Agent has checked and verified the site** — the agent confirms the HTML element
   structure, the backlink, the site requirements (personal/non-profit, own domain, real
   content, icon visible), cross-checking against the validation comment and its screenshot.
3. **Human operator has done a manual review** — the agent gives the operator the site URL
   and the operator checks it manually while the agent runs its own checks. Never approve
   or merge without the operator's explicit confirmation.

Once all three are satisfied and the operator confirms, approve and squash-merge
(see "If site meets all requirements" below).

### Initial PR Review Checklist
1. **Add review label** when starting review: `gh pr edit <PR_NUMBER> --add-label "review"`
   - IMPORTANT: After adding the label, immediately provide the website URL to the user for manual verification
   - Then yield back to the user to wait for checks to complete
   - Ask the user to notify you when checks are ready to proceed
2. **Check PR validation results** from the automated comment by github-actions bot
3. **Validate HTML element structure**:
   - Verify the PR adds a single `<li>` element at the end of the list
   - Check that `data-lang` attribute is either "en" or "th"
   - Confirm `id` attribute matches the domain name (without subdomain if present)
   - Verify `data-owner` attribute matches the PR author's GitHub username
   - Ensure link text matches the `id` attribute
4. **Verify webring link exists** on the site (backlink check should be ✅ in validation results)
5. **Check site requirements** against README criteria:
   - IMPORTANT: Provide the site URL to the user for manual verification
   - Wait for user confirmation before proceeding to approval
   - Personal, non-profit website (portfolio, blog, digital garden)
   - Has actual content (no "coming soon" pages)
   - Uses own domain (not github.io, netlify.app, etc.)
   - Icon is visible on both desktop and mobile

#### PR Actions by Situation

**IMPORTANT: Always use heredoc for properly formatted comments**

For all PR comments, use this format to preserve proper formatting:
```bash
gh pr comment <PR_NUMBER> --body "$(cat <<'EOF'
@<USERNAME> Your comment text here with proper formatting.

This preserves line breaks and markdown formatting properly.

> Even blockquotes work correctly.

EOF
)"
```

##### 1. If HTML element is invalid

```bash
# Comment about invalid HTML element
gh pr comment <PR_NUMBER> --body "$(cat <<'EOF'
@<USERNAME> Thanks for your interest in joining the webring. I noticed some issues with your HTML element:
- [specific issue with data-lang/id/data-owner]

Please update your PR to fix these issues according to the guidelines.
EOF
)"
```

##### 2. If site meets all requirements

```bash
# Approve and merge PR
gh pr review <PR_NUMBER> --approve --body "@<USERNAME> Thanks, and welcome to the webring!"
gh pr merge <PR_NUMBER> --squash
```

IMPORTANT: Always use the `--squash` option when merging PRs.

##### 3. If site is missing content

```bash
# Comment requesting more content
gh pr comment <PR_NUMBER> --body "$(cat <<'EOF'
@<USERNAME> Thanks for your interest in joining the webring! I noticed your website currently appears to mainly link to other platforms without having much original content directly on the site itself.

According to our requirements in the [README](https://github.com/wonderfulsoftware/webring?tab=readme-ov-file#%E0%B8%A3%E0%B9%88%E0%B8%A7%E0%B8%A1%E0%B8%A7%E0%B8%87), websites in the webring should:

> มีการเผยแพร่ผลงานบนเว็บไซต์ (ไม่รับเว็บที่เป็นหน้า Coming soon, Under construction หรือมีแค่ลิงค์ไปยังโซเชียลเน็ตเวิร์ค)

This means we're looking for sites that publish content directly on the website itself, rather than just linking to content on other platforms.

Would you be able to add some original content to your site? Once you've added some content, please let me know so we can take another look. Thanks!
EOF
)"
```

##### 4. If using public domain (not allowed)

```bash
# Comment about domain requirement
gh pr comment <PR_NUMBER> --body "$(cat <<'EOF'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wonderfulsoftware/webring](https://github.com/wonderfulsoftware/webring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
