# Doc Lookup — Shared Research Procedure

All modes (Inbox Probe, Email Q&A, Action Review) follow this procedure for every technical or product claim. **Always complete Step 1 before running any web search.**

---

## Step 1 — Local docs

### 1a. Try project_knowledge_search first

If the `project_knowledge_search` tool is available (claude.ai Project with knowledge), call it with the topic and use the results. Extract the source URL from chunk metadata (`> Source: https://...`) → this becomes `doc_url` for Step 2. Then skip to **Step 2**.

If `project_knowledge_search` is unavailable, continue with **1b–1e** below.

### 1b. Check for available docs

Run this to find path-loaded files (Claude Desktop / Cowork environments):

```bash
find / -name "product_a_docs_part*.md" -o -name "product_b_docs_part*.md" \
       -o -name "platform_docs.md" -o -name "product_c_docs_part*.md" \
       -o -name "*-overview.md" -o -name "sla.md" -o -name "terms.md" \
       2>/dev/null | grep -v proc | head -30
```

Also check for user direct uploads:
```bash
ls /mnt/user-data/uploads/ 2>/dev/null
```

> ⚙️ **SETUP:** Update the filenames in the `find` command above to match the names of your
> documentation files. Consistent filenames across setups make this command reliable.

**If `find` returns nothing and no user uploads exist** → no local docs available. Proceed directly to **Step 2**.

**If files are found** → continue with 1c–1e below.

### 1c. Route by topic

> ⚙️ **SETUP:** Replace the topic → filename mappings below with your own product documentation
> structure. Add or remove rows to match your file layout.

| Topic | Files |
|---|---|
| Product A — campaigns, messaging, segments, personalization | `product_a_docs_part1.md` … `product_a_docs_partN.md` |
| Product B — CDP, audiences, events, user data | `product_b_docs_part1.md` … `product_b_docs_partN.md` |
| Platform Console, add-on modules | `platform_docs.md`, `platform-overview.md` |
| Product C — chat, messaging | `product_c_docs_part1.md` … `product_c_docs_partN.md` |
| Product overviews | `product-a-overview.md`, `product-b-overview.md` |
| SLA, support policy, response times | `sla.md` |
| Operations, contract terms | `terms.md` |

For cross-product questions (e.g. CDP segmentation + campaign), search **multiple files**.

### 1d. Two-Pass Extraction — by file type

File sizes and section structures differ. Use the right extraction method per file:

**Type A — Small files (under 250 lines): read entirely, no grep needed**
- `sla.md`, `terms.md`, `*-overview.md`

```bash
cat /path/to/sla.md
```

**Type B — Large files with small sections (avg 7–8 lines/section): Two-pass section-aware**
- `product_a_docs_part*.md`, `product_b_docs_part*.md`, `product_c_docs_part*.md`

```bash
# Pass 1: keyword + synonyms → get matching line numbers
grep -in "KEYWORD1\|KEYWORD2\|SYNONYM" /path/to/file.md

# Pass 2: for each hit line, extract the full ## section it belongs to
awk -v target=LINE_NUMBER '
  /^## / { section_start=NR; section_content="" }
  { section_content = section_content "\n" $0 }
  NR==target { found=1 }
  found && NR>target && /^## / { print section_content; exit }
  END { if(found) print section_content }
' /path/to/file.md
```

Repeat Pass 2 for each distinct hit line. This returns the full semantic section (~7–20 lines) rather than a single matching line.

**Type C — Large files with large sections (avg 30–50 lines/section): Two-pass section-aware (same as Type B)**
- `platform_docs.md`

Same two-pass approach as Type B.

### 1e. Keyword Expansion

Before grepping, expand the core concept into synonyms and related terms to avoid missing content phrased differently. Bundle all variants into a single `grep -iE` call.

Think broadly: include the English term, any abbreviations, action-oriented phrasings, and closely related technical concepts. For example, "push notification delivery" might expand to `send|deliver|trigger|push|notification|message|token|endpoint|payload`.

The goal is to capture all the ways the documentation might describe the same concept — not to match a fixed list.

### 1f. Extract from the result

- Record: filename + full section content (not just the matching line)
- Extract the **source URL from chunk metadata** (the `> Source: https://...` line at the top of the section) → this becomes `doc_url`, used as the anchor for Step 2
- Do **not** extract incidental links or URLs from body text
- If no source URL in metadata → `doc_url: null`; use the product base URLs from `profile.md` for Step 2

---

## Step 2 — Web search augmentation (always runs after Step 1)

This is **not a fallback** — it runs regardless of whether Step 1 found something.

- If `doc_url` was found in Step 1 → use it as the basis for a targeted web search
- If `doc_url` is null → use the official base URL from `profile.md` for the relevant product
- Purpose: confirm the doc content is current, surface recent changes or additional detail

---

## Step 3 — State the gap

If neither Step 1 nor Step 2 covers the point, say so explicitly. Do not invent behavior. Recommend escalation if needed.

---

## References schema

Every reference entry (card `references` field or prose citation) follows this schema:

```json
{
  "source": "rag_indexed",
  "file": "product_a_docs_part2.md",
  "doc_url": "https://docs.example.com/product-a/push/setup",
  "url": null,
  "title": "Product A Push Setup — Dynamic Content",
  "snippet": "Push notifications require device token registration before campaign activation."
}
```

`source` values: `"rag_indexed"` · `"path_loaded"` · `"user_upload"` · `"web"`

A single claim typically yields two entries — one local doc, one web. Include both.

## Citation format

| Source | Format |
|---|---|
| Local doc, no `doc_url` | `📄 [filename, page N] — snippet` |
| Local doc, with `doc_url` | `📄 [filename, page N] — snippet` + `→ [linked text](doc_url)` |
| Web | `🌐 [page title](url) — snippet` |

If a web result contradicts or extends the local doc, flag it explicitly in the response.
