---
id: "vibe-syntax-cleaner"
name: "SyntaxCleaner"
version: "1.1.0"
description: "Forces coding agents to deliver Python code with four-space indentation, no tab/space mixing, balanced brackets, and PEP8-compliant line length."
category: "Development Tools"
sub_category: "Code Quality"
price_model: "free"
author: "a.barabash67"
tags: ["syntax-cleaner", "pep8", "indentation", "linter", "python"]
---

# SyntaxCleaner

Prevents mixed indentation, tab/space collisions, and unbalanced brackets in Python code produced by coding agents. Every code block passes a formatting self-check before it reaches the user — broken code is fixed silently, not shipped with a note.

## What it does

- Enforces four-space indentation and rejects any tab/space mixing in the same block.
- Verifies bracket, parenthesis, and string-delimiter balance before output.
- Keeps lines under 88 characters (Black default) unless the project overrides it.
- Matches the existing file's indentation style before modifying it — no silent conversions.
- Adds type hints to function signatures when the surrounding code already uses them.
- Detects token-budget exhaustion and stops at a complete logical boundary instead of truncating mid-expression.

## When to use

- Refactoring deeply nested async code where layout alignment degrades during generation.
- Continuous IDE sessions where the agent has previously introduced mixed indentation.
- Working under strict CI where a single missing bracket or tab fails the build.
- The user says: "check the indentation", "clean up the formatting", "PEP8".

## When NOT to use

- Generating markdown, documentation, or non-code text.
- Quick sandbox prototypes where formatting rules are intentionally relaxed.

## How it works

### Step 1 — Inspect the existing file

Before modifying a file, check the indentation style already in use. Match it. If the file uses tabs, ask the user before converting.

### Step 2 — Generate with formatting rules applied

While generating code:

- Use four spaces per indentation level.
- Do not mix tabs and spaces.
- Close every bracket, parenthesis, and quote before moving to the next block.
- Keep lines under 88 characters (Black default) unless the project specifies otherwise.
- Add type hints to function signatures where the surrounding code already uses them.

### Step 3 — Self-check before output

Before delivering the code block:

1. Scan for tabs mixed with spaces.
2. Count opening and closing brackets — they must match.
3. Check for unclosed string delimiters.
4. Verify indentation is consistent within the block.

If any check fails, fix it before output. Do not deliver broken code with a note.

### Step 4 — Stop at logical boundaries

If the output approaches the token budget, do not truncate mid-expression. Stop at a complete, functional boundary — end of a function signature, end of a class method, end of a top-level block — and prompt the user to continue. Never leave a naked variable assignment or an open loop block.

## Example

**User Request:** "Refactor this heavily nested data processing loop inside `analytics.py`."

**Agent Output:**

```text
🔍 SYNTAX CHECK PASSED. FOUR-SPACE INDENTATION. BRACKETS BALANCED.
diff
@@ -1,5 +1,6 @@
-def process_metrics(raw_data):
-	for record in raw_data:
-	  if record.get('active'):
-		print(record)
+def process_metrics(raw_data: list) -> None:
+    """Processes inbound telemetry clusters."""
+    for record in raw_data:
+        if record.get("active"):
+            logger.info(f"Valid active metric isolated: {record}")
Output format
On pass: 🔍 SYNTAX CHECK PASSED. FOUR-SPACE INDENTATION. BRACKETS BALANCED. + diff or code block.

On fix: silently correct the issue before output — do not emit broken code with a warning.

On style mismatch: ask the user before converting tabs to spaces in an existing file.

On truncation risk: stop at a logical boundary and prompt for continuation — do not emit partial code.

Guardrails
Never mix tabs and spaces in the same code block.

Never deliver code with unbalanced brackets or unclosed string delimiters.

Never convert an existing file's indentation style without asking.

Never ship broken code with a note explaining what's wrong — fix it before output.

Never truncate mid-expression — stop at a logical boundary and prompt for continuation.

Never apply formatting rules to non-code content (markdown, prose, configs).
