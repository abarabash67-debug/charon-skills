---
id: "vibe-pricing-reality"
name: "damage-check: Competitor Cost Map"
version: "1.1.0"
description: "Researches real, currently-listed prices for competing products and reframes them in relatable terms — sourced figures only, no estimates or invented wages."
category: "Productivity"
sub_category: "Market Research"
price_model: "free"
author: "a.barabash67"
tags: ["competitor-pricing", "market-research", "comparison-content", "pricing-intelligence"]
---

# PricingCheck: Competitor Cost Map

Collects real, currently-listed prices for named competing products or courses, and reframes the figures in relatable terms — e.g. as a fraction of a regional average monthly wage — to support fact-based comparison content. Every figure must be sourced; the skill does not estimate or invent prices, wages, or outcomes.

## What it does

- Searches current listed prices for named competitor offerings.
- Finds published average-wage data for the stated region, if the user wants relatable framing.
- Outputs a side-by-side comparison table: price, format/inclusions, relatable framing, source, and date for each figure.
- Separates fact ("here is the price and what it includes") from opinion ("here's whether that's worth it") — never blends the two.
- Flags missing data as missing — never reuses an old figure without a date check.

## When to use

- The user wants to compare their own free or low-cost offering against paid competitors using real numbers.
- The user is writing comparison content and needs sourced figures as raw material.
- The user says: "compare competitor prices", "how much do alternatives cost", "collect prices", "pricing research".

## When NOT to use

- The user wants unverifiable claims about a named competitor's income, student outcomes, or business practices — source only what's publicly verifiable.
- The user wants a full article — this skill produces research, not narrative.
- The user has no named competitors to compare against.

## Prerequisites

- Web/fetch access to read public pricing pages and published wage data.
- Named competitor offerings and, if using wage framing, the target region(s).

## How it works

### Step 1 — Identify what to compare

Confirm the named competitor offerings and the region(s) for wage comparison.

### Step 2 — Gather sourced figures

Search for current listed prices and, if used, published average-wage data for the stated region(s). Note the source and date for each figure.

### Step 3 — Build the comparison table

Present price, format/inclusions, and the relatable framing (e.g. "X% of the regional average monthly wage") side by side with the user's own offering.

### Step 4 — Hand off

Provide the sourced comparison as raw material. The persuasive framing and conclusion are the user's to write — PricingCheck does not assert them as established fact.

## Example

**User Request:** "Сравни цены на курсы по Python в Москве. Хочу показать, что мой бесплатный курс выгоднее."

**Agent Output:**

```text
Comparison — Python courses, Moscow region

| Course         | Price (RUB) | Format              | % of avg monthly wage | Source              | Date     |
|----------------|-------------|---------------------|-----------------------|---------------------|----------|
| Course A       | 120 000     | 3 months, live      | 133%                  | course-a.ru/pricing | 2026-09  |
| Course B       | 80 000      | self-paced, no tutor| 89%                   | course-b.ru/course  | 2026-09  |
| Course C       | 45 000      | 1 month, recorded   | 50%                   | course-c.ru         | 2026-09  |
| Your course    | 0           | self-paced, community| 0%                   | user-provided       | —        |

Average monthly wage (Moscow, 2026-Q3): 90 000 RUB
Source: https://example-stat-source.ru/moscow-wage-2026q3

Note: figures above are facts (price, format, source). The argument about
"which course is worth it" is not included — write that yourself.
Output format
On research: markdown table with price, format/inclusions, relatable framing, source, and date per row.

On missing price: state that the figure could not be found — do not reuse a remembered number.

On opinion request: separate fact rows from opinion text, do not blend.

On full-article request: state that PricingCheck produces research only.

Guardrails
Never use a price, wage, or income figure without a source and date.

Never omit what a competitor's offering actually includes — fair comparison requires the full picture.

Never present the user's argument as established fact — separate fact from opinion.

Never reuse an old figure without re-verifying the date.

Never assert unverifiable claims about a competitor's income, outcomes, or business practices.

Never produce narrative — PricingCheck produces research, not articles.
