---
name: nms-story-hunt
description: NMS morning story discovery. Finds trending marketing stories, verifies freshness at source level, scores by NMS criteria, outputs ranked queue for today's issue.
schedule: "30 5 * * 1-5"
---

You are running the morning story hunt for Notice Me Senpai.

Load brand config from /config/brand.json.
Check today's day of week to determine format: {{ brand.tracks.track1.format_rotation[TODAY_DOW] }}

## STEP 1 — DISCOVER STORIES

Search for marketing news using these sources in order:

**Web search queries to run:**
- "internet marketing news {{ TODAY_DATE }}"
- "SEO update {{ TODAY_DATE }}"
- "Meta ads update {{ THIS_WEEK }}"
- "Google ads {{ THIS_WEEK }}"
- "email marketing {{ THIS_WEEK }}"
- "social media marketing news {{ TODAY_DATE }}"

**RSS feeds to pull:**
- Marketing Brew
- Search Engine Journal
- Ahrefs Blog
- The Drum
- AdAge
- Social Media Today
- Marketing Dive

**Reddit hot posts (last 24hrs):**
- r/marketing
- r/SEO
- r/PPC
- r/entrepreneur

Collect all unique stories. Target: 15-20 raw candidates before filtering.

---

## STEP 2 — TWO-LAYER FRESHNESS CHECK

For EVERY candidate story, run both layers. Fail either = discard immediately.

**Layer 1 — Page recency:**
Was the page you found it on updated in the last 72 hours?
If no → DISCARD

**Layer 2 — Origin verification (CRITICAL):**
Web search the story headline. Find the EARLIEST coverage.
What date did this story FIRST appear anywhere?
If original publication > 72 hours ago → DISCARD as STALE

**Sources that CANNOT be trusted for freshness dating:**
- SocialBee (running changelog — page date ≠ story date)
- Swipe Insight (same — aggregator)
- Any "X updates this month" roundup page
- Weekly digest newsletters
- Stories that appear because someone tweeted an old article

You MUST find: original outlet + original publication date.
Example: "Meta announced X" → find the original Meta blog post or first trade press coverage → check THAT date.

Log discarded stories as STALE in story_queue with reason.

---

## STEP 3 — SCORE REMAINING STORIES

For each story that passed freshness, score on three dimensions (1-10 each):

**Practitioner Relevance (40% weight)**
How directly does this affect someone running campaigns TODAY?
10 = immediate budget/strategy impact
5 = interesting but not urgent
1 = industry gossip with no actionable angle

**Urgency (40% weight)**  
Does the reader need to act on this THIS WEEK specifically?
10 = deadline or window closing (e.g. new ad format with low CPMs NOW)
5 = matters but no time pressure
1 = evergreen, no urgency

**Shareability (20% weight)**
Would a reader forward this to their team?
10 = makes them look smart for sharing
5 = useful but not remarkable
1 = they'd read it and move on

Calculate weighted score. Rank all stories.

---

## STEP 4 — ANGLE CHECK

For the top 8 stories by score, run the angle check:

Search for existing coverage of this story. Check top 5 results.
Ask: What take has NOT been written yet?
- What would a skeptical 12-year practitioner push back on?
- What implication is everyone missing?
- What's the conventional wisdom here, and is it wrong?
- What specific number or benchmark does this change?

If no strong non-obvious angle exists → mark as NO_ANGLE, skip to next story.

A story with no angle is just a recap. NMS does not run recaps.

---

## STEP 5 — FINAL ADMISSION

Stories must clear ALL three gates:
1. ✅ Freshness verified (Layer 1 + Layer 2)
2. ✅ Weighted score ≥ 6.0
3. ✅ Strong non-obvious angle identified

Select top 5 admitted stories. Rank by score.
Story #1 = Big Story in today's issue.
Stories #2-4 = Quick Hits candidates.
Story #5 = backup if any of the above get cut in writing.

---

## STEP 6 — SEO OPPORTUNITY SCAN (TRACK 2)

While reviewing stories, flag any topic that is ALSO a keyword opportunity:
- Is this trending topic something people search for regularly?
- Does it map to a content gap in our pillar cluster structure?

Write flagged topics to seo_opportunity_queue in SQLite with:
- topic
- suggested_keyword
- cluster_match (which existing pillar it would support)

---

## OUTPUT

Write to SQLite story_queue table:
- All admitted stories with scores, angles, topics
- All discarded stories with discard_reason (STALE | LOW_SCORE | NO_ANGLE)

Write to SQLite seo_opportunity_queue:
- Any flagged SEO opportunities

Print summary to terminal:
```
NMS STORY HUNT — {{ TODAY_DATE }}
Format today: {{ FORMAT }}
Stories found: [N]
Stale discarded: [N]
Low score discarded: [N]
No angle discarded: [N]
ADMITTED: [N]

TOP STORY: [headline] — [one-line angle]
QUICK HITS: [3 headlines]
SEO OPPORTUNITIES: [N flagged]
```
