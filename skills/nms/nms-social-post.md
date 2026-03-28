---
name: nms-social-post
description: Generates and posts platform-native social teasers for today's NMS issue. All posts drive subscribe CTAs. Social is a funnel, not a repost.
schedule: "30 8 * * 1-5"
---

You are writing and posting social teasers for today's Notice Me Senpai issue.

Pull today's issue from SQLite: title, ghost_url, topics, html_content, best_line.
Load brand voice from /config/brand.json.

CORE PRINCIPLE:
Social posts are NOT reposts of the issue. They are teasers.
The full value is in the inbox. Social's job is to drive subscribes.
Each platform gets content written natively for that platform.

SUBSCRIBE LINK: https://noticemesenpai.com/#/portal/signup
UTM: ?utm_source={{ PLATFORM }}&utm_medium=social&utm_campaign={{ ISSUE_DATE }}

---

## PLATFORM 1 — X / TWITTER

Post the sharpest single take from today's issue.
The take that would get quoted or replied to.

Format:
```
[The take — 1-3 sentences, punchy]

[Why it matters in 1 sentence]

Full breakdown in today's NMS → [SUBSCRIBE_LINK]
```

Rules:
- No thread. One post. If it can't stand alone in 3 sentences it's the wrong take.
- The subscribe link goes at the end, not mid-post
- Don't use "Check out" or "Read more" — be direct
- Hashtags: none. They look desperate.

---

## PLATFORM 2 — LINKEDIN

Post THE NUMBER stat with full context.
LinkedIn rewards educational content. Give them something to save.

Format:
```
[The number — large, specific]

[What it means in 2-3 sentences — the non-obvious implication]

[One specific action they can take today]

If you want this breakdown in your inbox every morning:
→ noticemesenpai.com [SUBSCRIBE_LINK]
```

Rules:
- No "I'm excited to share" opener. Start with the number.
- LinkedIn algorithm rewards line breaks. Use them.
- The subscribe CTA is the last thing — earn it first
- 150-200 words max

---

## PLATFORM 3 — INSTAGRAM

Create a quote card brief (for manual creation or Canva template):

```
QUOTE CARD TEXT:
"[Best single line from today's issue — the most quotable]"

CAPTION:
[2-3 sentence context for the quote]
Full issue link in bio. Subscribe for daily.
```

Rules:
- The quote must work without context — standalone insight
- Caption is short — Instagram isn't for reading
- Link in bio → subscribe page

---

## PLATFORM 4 — TIKTOK SCRIPT

60-second talking head script. Shot from the waist up. No captions needed.

```
[0-3s HOOK]
"[One sentence that stops the scroll. Usually the most counterintuitive claim from today's issue.]"

[3-30s INSIGHT]
"Here's what actually happened. [Story in plain English — 3-4 sentences max. Conversational.]"

[30-50s THE TAKE]
"[The non-obvious implication. What most people are getting wrong.]"

[50-60s CTA]
"Full breakdown of this plus everything else that mattered in marketing today — it's in the NMS newsletter. Link in bio."
```

Rules:
- Hook is the most important 3 seconds. It must create tension or contradiction.
- Speak like a person, not a presenter
- The CTA names the newsletter — not "my newsletter" — NMS

---

## PLATFORM 5 — FACEBOOK

Repurpose the LinkedIn post with slight tone adjustment.
Facebook audience skews slightly older — more context, slightly less data-forward.
Same subscribe CTA. Same structure.

---

## POSTING

Post to each platform via OpenClaw social skills:

```
openclaw skill post-x --content="{{ X_CONTENT }}"
openclaw skill post-linkedin --content="{{ LINKEDIN_CONTENT }}"
openclaw skill post-instagram --caption="{{ IG_CAPTION }}" --image="{{ QUOTE_CARD_BRIEF }}"
openclaw skill post-tiktok --script="{{ TIKTOK_SCRIPT }}"
openclaw skill post-facebook --content="{{ FB_CONTENT }}"
```

Log all posts to social_log table with platform, content, posted_at.

---

## OUTPUT LOG

```
NMS SOCIAL — {{ TODAY_DATE }}
X: posted ✓
LinkedIn: posted ✓
Instagram: brief generated (manual post required) 
TikTok: script generated (manual record + post required)
Facebook: posted ✓

Subscribe links: all UTM-tagged
```

Note: Instagram and TikTok require manual recording/posting until automation is validated.
X, LinkedIn, Facebook post automatically.
