---
id: "vibe-architecture-advisor"
name: "ArchitectureAdvisor: Pattern Recommender"
version: "1.0.0"
description: "Detects common architectural pain points in a live coding session and names the pattern that fixes each one — router factories, config brokers, fallback layers, schema validation."
category: "Development Tools"
sub_category: "Architectural Patterns"
price_model: "free"
author: "a.barabash67"
tags: ["architecture-advisor", "patterns", "refactoring", "code-review", "vibe-coder"]
---

# ArchitectureAdvisor: Pattern Recommender

Detects common architectural pain points in a live coding session and names the pattern that fixes each one. This is a free advisory skill — it does not implement the pattern, it names it, shows the shape, and lets the user decide.

Built for vibe-coders who do not yet know the patterns by name. The skill translates "this feels wrong" into "here is the pattern that fixes it".

## What it does

- Scans the codebase for signals that a known pattern is missing.
- Names the pattern in one line — no lecture, no history lesson.
- Shows the shape of the fix in a short code fragment.
- States what breaks if the pattern is not applied.
- Never implements the full fix — the user decides whether to apply it manually or hand off to a specialized tool.
- Refuses to recommend a pattern the project is not ready for.

## When to use

- The user feels something is off but cannot name the problem.
- Before adding a feature when the codebase has not been reviewed in a while.
- The user says: "this feels wrong", "should I refactor", "is there a better way", "the agent keeps breaking this file".
- During code review, as a second opinion.

## When NOT to use

- The user has already decided what to change — do not second-guess a clear decision.
- The project is a one-off script that will not grow.
- The user wants a full implementation — this skill advises, it does not build.
- The project is in active incident — fix the incident first.

## Patterns the skill recognises

### 1. Manual import rows in a dispatcher

**Signal:** `from handlers import start, order, admin, ...` — an ever-growing import list in the entry file.

**Pattern:** Router factory — scan the package directory and mount every module that exposes a `router` object.

**Why it matters:** manual imports skip routers when someone forgets to add a line. The bug is silent — the handler exists but never fires.

### 2. Raw `dict` at a network boundary

**Signal:** `data = await request.json()` followed by direct key access.

**Pattern:** Schema validation — validate the payload against a typed model before any key is read.

**Why it matters:** a payload with an unexpected field or a missing field crashes the handler in production, not in tests.

### 3. Bare outbound API call

**Signal:** `response = requests.post(url)` or `await client.post(url)` with no timeout and no exception handling.

**Pattern:** Resilience wrapper — explicit timeout, verified exception handling, and an isolated fallback endpoint.

**Why it matters:** a single provider outage takes the whole bot down. A timeout without a fallback leaves the user waiting forever.

### 4. Secrets read directly from environment in many places

**Signal:** `os.getenv("TOKEN")` scattered across ten files.

**Pattern:** Config broker — one module reads the environment, everything else imports from it.

**Why it matters:** a leaked token is found in one file but references exist in ten. Rotation becomes impossible.

### 5. Global connection or bot instance imported everywhere

**Signal:** `from main import bot` or `from app import db` in leaf modules.

**Pattern:** Dependency injection — the connection is created once and passed to the modules that need it.

**Why it matters:** circular imports. The application freezes at startup when the import order shifts.

### 6. Long file with no clear boundary

**Signal:** a single file over 800 lines that mixes models, handlers, and utilities.

**Pattern:** Module split — one file per responsibility.

**Why it matters:** the AI agent starts dropping functions around 1500 lines. The split is not about style — it is about keeping the agent useful.

### 7. Database driver assumptions that do not survive a backend change

**Signal:** `AUTOINCREMENT`, `?` placeholders, `LIKE` used for case-insensitive search.

**Pattern:** Backend-agnostic queries — the constructs that work on both SQLite and PostgreSQL.

**Why it matters:** the query works on the current backend but silently returns wrong results on the next one.

### 8. No log, no trace, no metric in production paths

**Signal:** handlers that call an external service but never log the call or its result.

**Pattern:** Instrumentation at every boundary — log the request, log the failure, log the retry.

**Why it matters:** when something breaks at 2 a.m., there is no record of what happened.

## How it works

### Step 1 — Scan

Read the codebase. For each file, check for the signals above. Do not run every check against every file — scan the files the user has open or the file paths they named.

### Step 2 — Report one pattern at a time

For each detected signal, output:

```text
SIGNAL:      <what was found>
PATTERN:     <name of the pattern>
SHAPE:       <one-paragraph description or short code fragment>
WHY IT MATTERS: <what breaks if skipped>
Do not batch more than three signals in one response — the user will not act on ten.

Step 3 — Stop
After reporting, stop. Do not implement. Do not offer to implement. The user decides.

Example
User Request: "Пишу телеграм-бота, файл main.py уже 900 строк. Агент начал тупить — иногда теряет функции. Что не так?"

Agent Output:

text
SIGNAL:      Single file over 800 lines with mixed responsibilities
PATTERN:     Module split — one file per responsibility
SHAPE:       Split into config.py, db.py, models/, services/, handlers/, utils/.
             Each file holds one category. The entry file only wires them.
WHY IT MATTERS: the AI agent loses track of the full file around 1500 lines.
             Functions get dropped, helpers get re-invented, the same block
             gets rewritten twice in one session. This is not a prompting
             problem — the file itself is now the bottleneck.

SIGNAL:      Direct os.getenv() calls in multiple locations
PATTERN:     Config broker — one module reads the environment
SHAPE:       Create config.py with a single class that reads every variable.
             Every other file imports `from config import config`.
WHY IT MATTERS: a leaked token is found in one place but referenced in ten.
             Rotation becomes impossible.

SIGNAL:      Bare requests.post() to an external API with no timeout
PATTERN:     Resilience wrapper — timeout + fallback
SHAPE:       Wrap the call in try/except with an explicit timeout. Add a
             second endpoint. Route to it on verified failure.
WHY IT MATTERS: a single provider outage takes the bot down. A timeout
             without a fallback leaves the user waiting forever.
Output format
text
SIGNAL:      <what was found>
PATTERN:     <name of the pattern>
SHAPE:       <short description or code fragment>
WHY IT MATTERS: <what breaks if skipped>
On scan request: report up to three patterns, one block each.

On no signals found: state that the codebase shows no obvious pattern gaps — do not invent one.

On request to implement: decline — this skill advises, it does not build.

On user already decided: do not second-guess — implement their decision or step aside.

On incident in progress: stop the scan, tell the user to fix the incident first.

Guardrails
Never implement the pattern — advise only.

Never report more than three signals in one response.

Never report a pattern the project is not ready for.

Never second-guess a decision the user has already made.

Never name a specific paid tool as required — offer it as one option, not the only path.

Never report a signal that is not present in the code — no invented advice.

Never scan a project under 300 lines — there is nothing to advise yet.
