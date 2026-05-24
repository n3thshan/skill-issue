---
name: law-anki-flashcards
description: >
  Full pipeline for turning law lecture notes into Anki flashcards. Trigger whenever
  a user uploads a law lecture note and wants flashcards or Anki cards from it.
  Extracts case names, researches each case, and creates formatted cards via the Anki MCP tool.
---

# Law → Anki Flashcard Pipeline

Three steps: extract → research → create cards.

---

## Step 1: Extract Case Names

From the uploaded note, extract every legal case name (e.g. "Nash v Inman").
Ignore statutes, author names, and doctrinal terms. Do not show the list unless asked.
Before searching, correct any apparent misspellings in case names (e.g. "madarsaibo" → "Madar Saibo").

---

## Step 2: Research Each Case

Find per case: (1) material facts — what happened, (2) holding/ruling — what the court decided and why.

**Tool priority** — move to the next only if the current returns nothing useful:
1. `mcpjungle:google__apify--google-search-scraper` — query: `[Case Name] case law ruling facts`
2. Native web search (last resort)

**Source merging:**
- Facts → web search primary; fill gaps with user's notes
- Holding → user's notes primary; fill gaps with web search; user wins on conflict
- Cases mentioned by name only in the notes (no facts or ruling provided) → generate the card entirely from web search

If material facts are not found online after both tools, do not create a card for that case — report it at the end with the corrected name if applicable.

---

## Step 3: Create Anki Cards

If the user hasn't specified a deck, ask. Add one card per case via the Anki MCP tool (`mcpjungle`).

**Front:**
```
[2–4 sentence factual scenario — third person, past tense]
<br><br>
<strong>[Legal question mirroring the exact issue the case resolved]</strong>
```

**Back:**
```
<strong>[Case Name]</strong>
<br>
[1–3 sentences: ruling + ratio]
```

Case name on back only. Use `<br><br>` after scenario, `<strong>` for question and case name.

**Party name rule:** Never use the parties' actual names (as they appear in the case name) on the front. Replace them with contextually appropriate generic roles — e.g. claimant, defendant, buyer, seller, landlord, tenant, employer, employee, promisor, promisee. The case name on the back is the only place party names may appear.

---

## Edge Cases

- Facts not found online → no card created; report at end with corrected name if applicable
- No cases in notes → tell the user
- Multiple cases per concept → one card each
