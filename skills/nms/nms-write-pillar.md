---
name: nms-write-pillar
description: Writes NMS Track 2 SEO pillar posts. Keyword-targeted, 2000-3500 words, practitioner-focused. Publishes to Ghost with full SEO metadata. Runs Tue and Thu mornings.
schedule: "0 10 * * 2,4"
---

You are writing a Track 2 SEO pillar post for noticemesenpai.com.

Pull the top keyword opportunity from SQLite seo_opportunity_queue WHERE status = 'queued' ORDER BY priority DESC LIMIT 1.
Load brand config from /config/brand.json.
Pull existing posts from SQLite posts table for internal linking opportunities.

---

## STEP 1 — KEYWORD BRIEF

Research the target keyword:

```
Focus keyword: {{ KEYWORD }}
```

Web search tasks:
1. Search "{{ KEYWORD }}" — analyze top 5 results
   - What do they cover?
   - What's their word count?
   - What H2 structure do they use?
   - What do they all miss?

2. Search "{{ KEYWORD }} site:reddit.com" — what are practitioners actually asking?

3. Search "{{ KEYWORD }} statistics 2026" — find data to include

4. Search related questions: "best [keyword]", "how to [keyword]", "[keyword] examples"

Output keyword brief:
- Estimated monthly searches (rough)
- Top 3 competing pages + their weaknesses
- Content gap (what they all miss that NMS will cover)
- Related keywords to include naturally
- Questions to answer in FAQ section

---

## STEP 2 — OUTLINE

Build the outline for topical completeness:

```
H1: [Keyword-optimized headline — under 60 chars]
Meta description: [150-160 chars — includes keyword, includes benefit]

H2: [Section 1 — setup the problem/opportunity]
H2: [Section 2 — the core concept explained]
H2: [Section 3 — how to actually do it]
H2: [Section 4 — common mistakes / what not to do]
H2: [Section 5 — advanced tactics or edge cases]
H2: FAQ — [Keyword]: Common Questions Answered
  H3: [Question 1]
  H3: [Question 2]
  H3: [Question 3]

Internal links to include:
- [Related pillar post from posts DB]
- [Related news issue from posts DB]

Subscribe CTA placement: after Section 2, and at end
```

---

## STEP 3 — WRITING PASSES

**Pass 1 — Draft**
Write 2,000-3,500 words following the outline.
NMS voice throughout — same brand voice as the newsletter.
First-person where relevant. Opinionated. Data-backed.
Every H2 section ends with a specific action or takeaway.

**Pass 2 — Depth Pass**
This is the pass that justifies long-form.
For each section: have you gone deeper than any other result would?
The reader should feel they got something they couldn't find by scanning the top results.
The COUNTERARGUMENT section is non-negotiable — take the opposing view seriously.

**Pass 3 — SEO Pass**
- Focus keyword in H1, first 100 words, at least 2 H2s, meta description
- Related keywords distributed naturally — never forced
- FAQ section uses exact question phrasing from "people also ask" in search results
- Internal links: minimum 3 links to other NMS posts
- External links: 2-3 links to authoritative primary sources (studies, platform docs)
- Subscribe CTA: embedded naturally mid-article and at end

**Pass 4 — Voice + Tone**
- Banned words check
- Formula phrases check  
- Sentence rhythm check
- Ensure it sounds like the same publication as the newsletter

---

## STEP 4 — GHOST PUBLISH

Build Ghost post payload for Track 2:

```javascript
const post = {
  title: seoTitle,
  slug: seoSlug,
  html: postContent,
  status: "published",
  published_at: `${TODAY}T10:00:00.000Z`,
  tags: [
    { name: "pillar" },
    { name: "track2" },
    { name: focusKeyword },
    ...topicTags
  ],
  meta_title: seoTitle,
  meta_description: metaDescription,
  codeinjection_head: buildArticleSchema(post),
  // NO newsletter send — pillar posts do NOT go to email list
  send_email_when_published: false
};
```

Schema type is "Article" (not "NewsArticle") — pillar posts are not time-sensitive news.

---

## STEP 5 — UPDATE SQLITE

```sql
INSERT INTO posts (
  published_date, track, title, slug, url, 
  ghost_post_id, focus_keyword, lsi_keywords, topics, word_count
) VALUES (
  DATE('now'), 'track2', '{{ TITLE }}', '{{ SLUG }}', '{{ URL }}',
  '{{ GHOST_POST_ID }}', '{{ FOCUS_KEYWORD }}', '{{ LSI_KEYWORDS }}', 
  '{{ TOPICS }}', {{ WORD_COUNT }}
);

UPDATE seo_opportunity_queue SET status = 'used' WHERE keyword = '{{ KEYWORD }}';
```

---

## STEP 6 — NEWSLETTER MENTION

Add to next_day_newsletter_notes in SQLite:
"From the archive: {{ TITLE }} — {{ URL }}"
Tomorrow's issue Brief will include a one-liner pointing to the new pillar post.

---

## OUTPUT LOG

```
NMS PILLAR POST — {{ TODAY_DATE }}
Keyword: {{ FOCUS_KEYWORD }}
Word count: {{ WORD_COUNT }}
Ghost URL: {{ URL }}
Internal links: {{ N }}
Newsletter mention: queued for tomorrow
Email send: NO (Track 2 does not send to list)
```
