---
name: nms-learning-loop
description: Nightly review of all NMS performance signals across both tracks. Updates tomorrow's content strategy, story scoring weights, and affiliate priorities. Sends weekly digest on Sundays.
schedule: "30 23 * * 1-5"
---

You are running the Notice Me Senpai nightly learning loop.

Pull all today's performance data from SQLite.
Analyze across both tracks.
Update strategy configs for tomorrow.

---

## STEP 1 — PULL TODAY'S DATA

```sql
-- Track 1: Newsletter performance
SELECT open_rate, click_rate, new_subscribers, unsubscribes, forwards,
       email_subject_winner, google_news_pickup, format
FROM issues WHERE issue_date = DATE('now');

-- Track 2: Pillar post performance (early signals)
SELECT title, focus_keyword, url, word_count
FROM posts WHERE published_date = DATE('now') AND track = 'track2';

-- Tool brief performance
SELECT tool_name, clicks, conversions, revenue, selection_tier, relevance_match
FROM tool_brief_log WHERE issue_date = DATE('now');

-- Story queue: what was used vs skipped
SELECT headline, status, relevance_score, topics, angle_summary
FROM story_queue WHERE DATE(queued_at) = DATE('now');

-- 30-day rolling averages for comparison
SELECT AVG(open_rate) as avg_open, AVG(click_rate) as avg_click,
       AVG(new_subscribers) as avg_subs
FROM issues WHERE issue_date > DATE('now', '-30 days');
```

---

## STEP 2 — DIAGNOSE

Answer each question:

**Open rate vs 30-day average:**
- Above average → what made today's subject line work?
- Below average → was it the subject line, send time, or topic?
- Flag if below 30% → deliverability or relevance issue

**Click rate:**
- Which story got the most clicks?
- Was it the Big Story or a Quick Hit?
- What topic was it? Add weight to that topic in scoring.

**Unsubscribes:**
- Any spike vs 7-day average?
- If yes: what topic or tone might have caused it?
- Flag for blacklist consideration if > 0.5% on a single issue

**New subscribers:**
- Which social platform drove the most today? (check UTM data)
- Was there any Google News pickup? Which story?
- Was there a referral spike?

**Tool brief:**
- Did the tool topic match the article topic?
- What was the CTR?
- Update ctr_30d in affiliates table

**Story quality:**
- Which stories were discarded as STALE? Note source type.
- Were any good stories rejected for no angle? Could the angle-finding prompt be improved?

---

## STEP 3 — UPDATE CONFIGS

**Update story_scoring_weights in SQLite:**
Topics that drove high clicks → increase weight by 0.1
Topics that drove unsubscribes → decrease weight by 0.1
Topics on blacklist → set weight to 0

**Update affiliate weights:**
```sql
UPDATE affiliates SET 
  ctr_30d = (
    SELECT COALESCE(AVG(CAST(clicks AS REAL) / NULLIF(1, 0)), 0)
    FROM tool_brief_log 
    WHERE tool_name = affiliates.tool_name 
    AND issue_date > DATE('now', '-30 days')
  )
WHERE active = TRUE;

UPDATE affiliates SET last_featured = DATE('now')
WHERE tool_name = (SELECT tool_name FROM tool_brief_log WHERE issue_date = DATE('now') LIMIT 1);
```

**Write morning_context.json for tomorrow:**
```json
{
  "date": "{{ TOMORROW }}",
  "format": "{{ TOMORROW_FORMAT }}",
  "top_performing_topics_this_week": [],
  "blacklisted_topics": [],
  "stale_source_types_to_avoid": [],
  "open_rate_trend": "up|flat|down",
  "subscriber_growth_trend": "up|flat|down",
  "notes_for_tomorrow": "{{ CLAUDE_ANALYSIS }}"
}
```

---

## STEP 4 — LOG TO PERFORMANCE_LOG

```sql
INSERT INTO performance_log (
  log_date, track, top_topic, top_format,
  avg_open_rate, avg_click_rate, new_subs_today,
  google_news_pickups, notes
) VALUES (
  DATE('now'), 'track1', '{{ TOP_TOPIC }}', '{{ FORMAT }}',
  {{ OPEN_RATE }}, {{ CLICK_RATE }}, {{ NEW_SUBS }},
  {{ GOOGLE_NEWS_COUNT }}, '{{ ANALYSIS }}'
);
```

---

## STEP 5 — WEEKLY DIGEST (Sundays only)

If today is Sunday, compile weekly summary and send via Ghost to your email:

Subject: "NMS Weekly — {{ WEEK_DATES }}"

Include:
- Total new subscribers this week
- Best performing issue (highest open rate)
- Best performing story (highest clicks)
- Top affiliate performer
- Google News pickups
- Subscriber churn
- Revenue this week (sponsor + affiliate)
- What's working
- What to change next week

---

## OUTPUT LOG

```
NMS NIGHTLY LOOP — {{ TODAY_DATE }}
Open rate today: {{ OPEN_RATE }}% (30d avg: {{ AVG_OPEN_RATE }}%)
Click rate today: {{ CLICK_RATE }}%
New subscribers: {{ NEW_SUBS }}
Unsubscribes: {{ UNSUBS }}
Google News pickup: {{ YES/NO }}
Tool brief CTR: {{ TOOL_CTR }}%
Topics upweighted: {{ TOPICS }}
Topics downweighted: {{ TOPICS }}
morning_context.json: updated ✓
affiliate weights: updated ✓
```
