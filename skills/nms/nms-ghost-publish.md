---
name: nms-ghost-publish
description: Publishes finished NMS issue to Ghost CMS (SEO archive + Google News) AND to Beehiiv (email delivery). Ghost publishes the blog post. Beehiiv sends the email. Two destinations, one write.
schedule: "0 8 * * 1-5"
---

You are publishing today's Notice Me Senpai issue.

Pull today's issue JSON from SQLite issues table WHERE issue_date = DATE('now') AND status = 'draft'.

This skill posts to TWO destinations:
1. Ghost — the SEO website and Google News archive (does NOT send email)
2. Beehiiv — the email newsletter sent to subscribers (does NOT host public archive)

---

## DESTINATION 1 — GHOST

Build and publish the Ghost post with NewsArticle schema for Google News.
Ghost email newsletter sending must be DISABLED in Ghost settings.
Beehiiv handles all email — do not attempt to send from Ghost.

```javascript
const ghostPost = {
  title: issue.subject_line_a,
  slug: `nms-${issue.issue_date}-${issue.format}`,
  html: issue.html_content,
  status: "published",
  published_at: `${issue.issue_date}T08:00:00.000Z`,
  tags: [{ name: "nms-issue" }, { name: issue.format }, ...topicTags],
  meta_title: issue.subject_line_a,
  meta_description: issue.meta_description,
  codeinjection_head: newsArticleSchema,
};
```

NewsArticle schema fields: headline, datePublished, dateModified, author (Organization: Notice Me Senpai), publisher with logo, mainEntityOfPage.

---

## DESTINATION 2 — BEEHIIV

Create the post via Beehiiv API, then schedule send for 8:05am (5 min after Ghost publish).

```javascript
// Step 1: Create post
POST https://api.beehiiv.com/v2/publications/{BEEHIIV_PUB_ID}/posts
Authorization: Bearer {BEEHIIV_API_KEY}
{
  subject: issue.subject_line_a,
  preview_text: issue.preview_text,
  content: { type: "html", value: issue.html_content },
  audience: "free"  // sends to all free + paid subscribers
}

// Step 2: Schedule send
POST https://api.beehiiv.com/v2/publications/{pub_id}/posts/{post_id}/send
{ send_at: "ISSUE_DATE T08:05:00Z" }
```

Beehiiv appends the referral block automatically to every email if the referral program is enabled in Beehiiv settings.

---

## ERROR HANDLING

Ghost fails → alert Telegram, do NOT send Beehiiv, set status = ghost_failed
Beehiiv fails after Ghost succeeds → alert Telegram "manual Beehiiv send required", Ghost post stays live, set status = beehiiv_failed, retry once after 5 minutes

---

## OUTPUT LOG

Ghost: published URL
Beehiiv: scheduled 8:05am, post ID
Tool brief: tool name + tier
