---
trigger: always_on
description: `hashline` is a file editing tool that uses content-hashed line anchors (`12:ab3f`) instead of fragile exact-text matching. It's designed for agent-driven editing where concurrent changes are expected and edit safety is critical.
---

## hashline — Hash-Anchored File Editing

`hashline` is a file editing tool that uses content-hashed line anchors (`12:ab3f`) instead of fragile exact-text matching. It's designed for agent-driven editing where concurrent changes are expected and edit safety is critical.

### Why It's Useful

- **Stable anchors:** Uses `line:hash` format that survives nearby edits—line numbers shift but hashes stay valid
- **Concurrent-safe:** Detects stale anchors when content changed; fails explicitly instead of guessing
- **Audit trail:** Optional `--receipt` and `--audit-log` for tracking edit history
- **No merge conflicts:** Each edit is independent; no patch files that conflict
- **Works with any text:** Language-agnostic; no parsing required

### The Anchor Format

Anchors are `line_number:content_hash` pairs like `42:a3f2`:

- **line_number**: 1-based line number (for human readability)
- **content_hash**: First 4+ chars of SHA-256 of line content (for stability)

Example output from `hashline read`:
```
  1:a1b2  fn main() {
  2:c3d4      println!("hello");
  3:e5f6  }
```

### Command Reference

**Reading:**
| Command | Purpose |
|---------|---------|
| `hashline read <file>` | Show file with line:hash anchors |
| `hashline read <file> --anchor 42:a3f2` | Show context around specific anchor |
| `hashline read <file> --context 10` | Set context lines (default: 5) |
| `hashline index <file>` | Show just anchors, no content |

**Editing:**
| Command | Purpose |
|---------|---------|
| `hashline edit <file> <anchor> <content>` | Replace line at anchor |
| `hashline edit <file> <start>..<end> <content>` | Replace line range |
| `hashline insert <file> <anchor> <content>` | Insert after anchor |
| `hashline insert <file> <anchor> <content> --before` | Insert before anchor |
| `hashline delete <file> <anchor>` | Delete line at anchor |
| `hashline delete <file> <start>..<end>` | Delete line range |

**Searching:**
| Command | Purpose |
|---------|---------|
| `hashline grep <file> <pattern>` | Search with anchor output |
| `hashline grep <file> <pattern> --case-insensitive` | Case-insensitive search |
| `hashline annotate <file> <query>` | Find and annotate matching lines |
| `hashline annotate <file> <regex> --regex` | Regex search |
| `hashline find-block <file> <anchor>` | Find enclosing block (brace/indent) |

**Utilities:**
| Command | Purpose |
|---------|---------|
| `hashline verify <file>` | Verify file integrity |
| `hashline stats <file>` | File statistics |
| `hashline patch <file> <patch>` | Apply patch: `-` stdin, `@path` file, or literal text |
| `hashline swap <file> <anchor1> <anchor2>` | Swap two lines |
| `hashline move <file> <anchor> <target-anchor>` | Move line to new position |
| `hashline indent <file> <anchor> <levels>` | Adjust indentation |

**Advanced:**
| Command | Purpose |
|---------|---------|
| `hashline from-diff <diff-file>` | Convert diff to anchor edits |
| `hashline merge-patches <file> <patch1> <patch2>` | Merge multiple patches |
| `hashline watch <file>` | Watch file for changes |
| `hashline explode <file>` | Split file into per-line files |
| `hashline implode <file>` | Reassemble from per-line files |

### Typical Agent Workflow

1. **Read file with anchors:**
   ```bash
   hashline read src/main.rs
   ```

2. **Find specific content:**
   ```bash
   hashline grep src/main.rs "fn process" --json
   ```

3. **Apply targeted edit:**
   ```bash
   hashline edit src/main.rs 42:a3f2 "fn process_data(input: &str) -> Result<()> {"
   ```

4. **Verify change:**
   ```bash
   hashline read src/main.rs --anchor 42:a3f2
   ```

5. **If anchor is stale, re-read and retry:**
   ```bash
   hashline read src/main.rs  # Get fresh anchors
   hashline edit src/main.rs 42:new_hash "..."
   ```

### Range Edits

Replace multiple lines with range syntax:

```bash
# Replace lines 10-15
hashline edit src/main.rs 10:a1b2..15:c3d4 "new content\nspanning\nmultiple lines"

# Delete lines 20-25
hashline delete src/main.rs 20:e5f6..25:g7h8
```

### Multi-Line Patches (USE STDIN — never create .patch files)

For multi-op patches, use `hashline patch` with stdin. **Never** write a `.patch` file to disk first:

```bash
# ✅ CORRECT — stdin via heredoc, no disk I/O
hashline patch src/main.rs - <<'EOF'
*** Begin Patch
SWAP 42:a3f2:
+fn process_data(input: &str) -> Result<()> {
+    todo!()
+}
SWAP 45:1a2b:
+    Ok(())
*** End Patch
EOF

# ❌ WRONG — creates a stale .patch file littering /tmp
cat > /tmp/something.patch <<'EOF'
...patch content...
EOF
hashline patch src/main.rs @/tmp/something.patch
```

**Why stdin wins:**
- No intermediate file on disk (no cleanup needed, no `/tmp` litter)
- 1 process spawn vs 1 spawn + 1 file write + 1 file read
- ~3x faster for typical patches
- Atomic — patch content is bound to the command, not a stale file

**`hashline patch <file> <patch>` argument modes:**
- `hashline patch file -` → read from stdin
- `hashline patch file @/path/to/file` → read from a file (only when patch is reused or pre-existing)
- `hashline patch file "literal text"` → use the argument as-is (no newlines in shell-safe form)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quangdang46/hashline](https://github.com/quangdang46/hashline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
