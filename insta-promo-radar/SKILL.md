---
name: insta-promo-radar
description: >
  Find and surface active promotions, deals, discounts, and limited-time offers
  posted on a brand's Instagram account. Use this skill whenever a user asks
  about deals, promos, offers, or discounts from a brand on Instagram — even if
  they phrase it casually like "any deals at Domino's?", "what's Zara offering
  right now?", "check Nike's IG for sales", or "compare promotions across these
  brands". Also trigger when a user asks to monitor or track what a brand is
  currently promoting on social media, or wants to know if a brand has any active
  Instagram campaigns. Always use this skill — don't attempt to answer from memory,
  since promotions change constantly and require live data.
---

# Instagram Promo Radar

This skill finds active promotions and deals posted on a brand's Instagram.
It covers the full pipeline: handle verification → scraping → filtering → presenting results.

---

## Step 1 — Identify the target brand(s)

Extract every brand or entity the user wants to check. They may name one or several
(e.g., "compare Domino's and Pizza Hut promotions").

If the user already supplies a handle (e.g. `@nike`, `instagram.com/nike`), use it directly —
web verification is still recommended but optional.

---

## Step 2 — Verify the official Instagram handle

For each brand without a confirmed handle, use the Tavily web search tool:

```
<Brand Name> official Instagram handle
```

From the results, extract the `@handle` or `instagram.com/<handle>`.
Prefer signals from:
- The brand's official website (most reliable)
- Verified press/news articles
- Social media directories

**If multiple handles appear** (e.g. regional accounts), pick the one with the
strongest authority signal. If you genuinely can't tell, ask the user before scraping.

**If no handle can be found**, stop and ask the user to supply it rather than guessing.

---

## Step 3 — Scrape the account(s)

For each verified handle, call:

```
apify__coderx--instagram-profile-scraper-bio-posts(usernames=["<handle>"])
```

If the user asked about multiple brands, pass all verified handles in one call
(e.g. `usernames=["dominos", "pizzahut"]`) to minimise round trips.

---

## Step 4 — Filter for promotional content

Review the `latestPosts` array from the scrape output. Each post has:
- `url` — direct link to the post
- `caption` — the post's text
- `timestamp` — publication date
- Other metadata (likes, comments, etc.)

A post counts as **promotional** if the caption or context suggests:
- Discounts or percentage-off offers ("20% off", "BOGO", "half price")
- Limited-time deals ("this weekend only", "ends Sunday", "flash sale")
- Promo codes or vouchers ("use code SUMMER25")
- Bundle or combo offers ("2 for £20", "meal deal")
- Giveaways or contests with a purchase element
- New product launches paired with an introductory offer
- Free delivery or bonus items ("free fries with any order")

**Not promotional:** general brand storytelling, recipe content, lifestyle imagery,
or new product announcements without any offer attached.

---

## Step 5 — Present the results

### If promotional posts are found

For each matching post, use this format:

> **[Brand] — [Short deal description] 🏷️** (*[Date, e.g. May 28, 2026]*)
> [One-sentence summary of the offer in your own words]
> 🔗 [full post URL]

Example:
> **Domino's — 2-for-1 Tuesdays 🏷️** *(May 27, 2026)*
> Every Tuesday, any two medium pizzas for the price of one — no code needed.
> 🔗 https://www.instagram.com/p/ABC123xyz

If comparing multiple brands, group results under a heading per brand, then add
a brief **Summary** at the end noting which brand has the most active or compelling offers.

### If no promotional posts are found

Say so clearly, then give a one-paragraph summary of what the account *is*
posting about recently (so the user isn't left empty-handed). Example:

> No promotional posts were found in [Brand]'s recent Instagram activity. Their
> latest posts focus on [general theme, e.g. "behind-the-scenes content and
> seasonal recipes"]. Worth checking back later or visiting their website directly
> for current offers.

---

## Edge cases

| Situation | Action |
|-----------|--------|
| User provides handle directly | Skip web search (or do a quick sanity check) |
| Ambiguous handle (e.g. brand has regional accounts) | Pick strongest signal; note the choice to the user |
| No handle found via search | Ask user before calling the scraper |
| Scraper returns no posts | Report it and suggest the account may be private or inactive |
| Post is in a foreign language | Translate the caption before deciding if it's promotional |
| Multiple brands requested | Batch into one scraper call; present results per brand |
