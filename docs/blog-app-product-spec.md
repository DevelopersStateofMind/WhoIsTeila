# Blog Generator App - Product Spec v2

**As of September 2026.** This spec overrides the earlier docs wherever they conflict.
Related: [`blog-generator-research.md`](blog-generator-research.md), [`blog-seo-rulebook.md`](blog-seo-rulebook.md).

---

## 1. Locked decisions

| # | Decision |
|---|---|
| 1 | **Standalone app that serves multiple clients from one install.** Sold to clients. Each client is a separate account with its own site, brand, keywords and settings |
| 2 | **Test phase = 4 SPC-owned sites only:** SourcePro Consultants, Esquire Pilot, Find George Atty (domain TBC), teilagarraway.com. Luxe World Travels and Tribe of Judah move to the first client wave after testing |
| 3 | **Publishing:** custom sites first, through the app's Content API. WordPress and GHL connectors come later, on demand |
| 4 | **Business owner approval** on every post, every client, every industry. Industry packs (e.g. the law firm pack) add rules on top |
| 5 | **Owner Insight step** before drafting (Section 4) |
| 6 | **CTA library set at onboarding.** Every post gets a CTA (Section 5) |
| 7 | **Keyword Library lives inside the app.** ClickUp is an optional notification, not where the data lives |
| 8 | **App stack: Next.js** (app + owner portal + Content API in one codebase). Scheduled jobs run in the app (cron/queue); n8n optional |
| 9 | **Bring your own AI keys (BYOK):** each client connects their own text AI and image AI accounts. SPC's keys are used only for the 4 SPC sites (Section 9) |
| 10 | **Owner notifications: email only** (insight request, approval request, published, errors) |

---

## 2. Publishing architecture (custom sites)

### Recommended: headless Content API
The app stores the posts. Each website pulls them in. No redeploy per post.

```
[Blog App]  --(approved + scheduled time)-->  Post status = published
     |
     |-- GET /api/v1/{site}/posts            (list, paginated, by category)
     |-- GET /api/v1/{site}/posts/{slug}     (full post: HTML/MDX, meta, schema JSON-LD, images, CTA)
     |-- GET /api/v1/{site}/sitemap          (blog sitemap entries)
     |-- GET /api/v1/{site}/rss
     |
     '-- POST webhook -> {site}/api/revalidate   (tells the site to refresh the new page right away)

[Client site, Next.js]  /blog and /blog/[slug] pages read from the API
                        page is pre-built and served fast (good Core Web Vitals)
```

**Per-site install (one time):** add `/blog` routes, a revalidate endpoint, and a read-only API key.
Package this as a **small SDK** (`@spc/blog-client`) so every future custom site plugs in the same way.

### Option 2 (not recommended): commit posts to the site's repo
The app writes an MDX file to each site's GitHub repo, which triggers a redeploy.
Pros: posts live in the repo. Cons: a rebuild for every post, a GitHub token for every client, slower, and harder to sell to non-developer clients.

### Future connectors (same post object, different adapter)
| Connector | Build when |
|---|---|
| WordPress (REST API + application password) | First law firm that won't move off WordPress |
| GHL Blog API | First client whose site runs on GHL |
| Webflow / Shopify / Wix | On demand |
| Embed script (JS widget) | Clients with a site builder that has no API |

---

## 3. Existing n8n workflows: verdict

Checked live in n8n on 9/25/26.

| Workflow | Status | Finding |
|---|---|---|
| **W0 - SEO Topic Generator (GSC to Blog Queue)** | Active, **failing since 9/14** | Tested end-to-end 9/12. The last 2 Monday runs (9/14, 9/21) errored: the **"SPC > Google Drive account" credential needs reconnecting** (refresh token expired or was revoked). Breaks at "Google Drive - Download Blog Tracker" |
| W-KEYWORD - Topic Validation Gate | Inactive | DataForSEO competition check before the ClickUp task set is created. Built, never turned on |
| W1 - Blog Generator | Active, runs daily | Runs finish in under 1 second, which most likely means no topics are reaching it (W0 is down) |
| W2 - Backlink Intelligence | Active | Google Alerts RSS feeds, scored by Claude as backlink prospects or blog ideas |

**Security flag:** W0's "Google Chat Notification" node has the Chat webhook `key` and `token` typed directly into the URL. Move them into an n8n credential and rotate the webhook.

### Should W0 keep feeding the blog? Yes, as one input, not the whole feed.

Keep its logic. It's the **"GSC opportunity"** signal in the Keyword Engine: queries with 20+ impressions ranking below position 10, plus the Claude relevance filter that drops brand-confusion noise.

W0's limits as the only source:
1. **GSC only.** Newer sites (e.g. Esquire Pilot, Find George Atty, teilagarraway.com) may have little GSC data yet, so they need DataForSEO research to seed their libraries
2. **Hardcoded to SPC**: domain, ClickUp list, brand prompt
3. **Top 3 per week**, with no library, scoring, clusters or dedupe against published posts
4. **ClickUp is the queue.** Clients won't have your ClickUp

### Plan
| Phase | Action |
|---|---|
| **Now** | Reconnect the Drive credential in n8n, then run W0 manually once to confirm. Move the Chat webhook secret into a credential. SPC keeps its content flow while the app is built |
| **Build** | Move W0's GSC gap logic and W-KEYWORD's DataForSEO check into the app's Keyword Engine, set up per client |
| **Cutover** | For SPC, the app's library replaces W0 + W1. Keep the ClickUp Article/Carousel tasks as an **optional output** ("post published: create repurpose tasks") |

---

## 4. Owner Insight step (new)

**Yes, the app should pause and ask the owner for input before drafting.** This is the highest-value step in the pipeline. It supplies the first-hand experience (the first "E" in E-E-A-T) and the original material that AI can't invent and that Google's helpful-content checks reward.

### Flow
```
Keyword picked -> SERP Brief built -> OWNER INSIGHT REQUEST -> (wait) -> Draft -> QA -> OWNER APPROVAL -> Publish
```

### What the owner receives (email with a link to the app portal)
- Topic, working title, one-line angle, outline (H2s)
- **3-5 questions written for this topic by Claude**, drawn from:
  1. "What's the most common mistake you see people make with [topic]?"
  2. "Share a real example or story (no client names)."
  3. "Where do you disagree with the usual advice on this?"
  4. "Any number from your business? (time saved, typical cost, how often it happens)"
  5. "What do you tell people first when they ask about this?"
- **Answer by:** typing, **voice note** (transcribed automatically), or pasting notes
- Buttons: **Submit** / **Skip this one** / **Change topic**

### Rules
| Setting | Default (can be changed per client) |
|---|---|
| Batch | **Monday: one form covering all 3 posts that week** (one touchpoint, not three) |
| Wait time | 48 hours |
| No reply | Draft without insight. The QA "Information gain" score is capped and the post is flagged "no owner input" at approval |
| Use | Owner answers are quoted or paraphrased in the post, shown as the owner's voice ("In our experience...") |
| Reuse | Answers are saved to an **Insight Bank** per client and can be reused in related posts in the same cluster |

### Owner touchpoints per week
1. **Monday:** Insight form (3 topics, ~10 minutes)
2. **Midweek:** Approval (one click per post, or request edits)

---

## 5. CTA system

**Every post gets a CTA.** A blog post without one brings traffic but no leads. Set it up at onboarding as a **CTA library**, not a single link.

### CTA library (per client, at onboarding)
| Field | Example (SPC) |
|---|---|
| **Primary CTA** (high intent) | "Book a Systems & Tool Stack Diagnostic", linking to the booking page |
| **Secondary CTA** (low intent) | Lead magnet / checklist download / newsletter |
| **Service-mapped CTAs** | Cluster "case management" goes to the case management service page |
| Button text + URL + UTM template | `?utm_source=blog&utm_medium=cta&utm_campaign={slug}` |

### Placement rules (app enforces)
| Post intent | Mid-post (after ~40% of the post) | End of post |
|---|---|---|
| Informational (how-to, FAQ) | Secondary (soft) CTA | Primary CTA |
| Commercial (cost, comparison, "hire") | Primary CTA | Primary CTA |

- UTM on every CTA so **leads are tracked back to each post** (this feeds the monthly client report)
- Test phase: Teila provides the primary and secondary CTAs for all 4 SPC sites at onboarding

---

## 6. Client onboarding config (one record per client)

| Group | Fields |
|---|---|
| Business | Name, industry, service area, services/offers, audience |
| Brand | Voice notes, banned words, competitors, brand-fit rules |
| Site | Domain, publishing method (Content API / WordPress / GHL), revalidate URL, API key |
| Data | GSC property access, DataForSEO location + language |
| Owner | Name, bio, credentials, headshot (author box), email/phone for insight + approval requests |
| CTA library | Section 5 |
| Cadence | Posts/week (default 3), publish days/time, time zone |
| Compliance pack | None / Law firm / (future: medical, financial) |
| Notifications | Email, SMS, Google Chat, ClickUp (optional) |

---

## 7. Updated pipeline

```
Onboarding -> Keyword Engine (GSC gaps [W0 logic] + DataForSEO research [W-KEYWORD logic] + brand-fit scoring)
          -> Keyword Library -> Cluster Planner
          -> SERP Brief -> OWNER INSIGHT (Monday batch, 48h)
          -> Writer -> Fact-Check -> Link Engine -> Image Engine -> CTA Placement
          -> QA Score (100 pts) -> OWNER APPROVAL
          -> Publisher (Content API + revalidate webhook | future connectors)
          -> Distribution (GBP, social, newsletter, optional ClickUp repurpose tasks)
          -> Index Check -> Refresh Engine -> Monthly Report (includes CTA leads per post)
```

---

## 8. Image Engine

Every post gets images generated to the client's brand style, reviewed with the post at approval.

### Images per post
| Image | Size | Purpose |
|---|---|---|
| Featured / hero | 1200x630 (also used as the Open Graph share image) | Top of post, blog index card, social shares |
| Inline images | 1 per ~500 words (usually 2-4) | Break up sections, illustrate steps or concepts |
| Social variants (optional) | 1080x1080, 1080x1350 | Repurpose pack |

### How an image is made
```
Post outline -> Claude writes 1 prompt per image slot
             (brand style preset + section topic + "no text" unless a text model is used)
          -> Image API (client's own key) -> 2 options per featured image
          -> Auto-check: size, no garbled text, no faces of real people, brand colors
          -> Convert to WebP, under 150KB, SEO file name ({primary-keyword}-{n}.webp)
          -> Claude writes alt text (describes the image + uses the keyword naturally, featured image only)
          -> Stored in the app's media storage and served through the Content API / CDN
```

### Brand style preset (per client, set at onboarding)
- Style: photo / illustration / 3D / flat graphic
- Brand colors (hex), mood, lighting
- Always / never lists (e.g. "never: gavels, scales of justice clichés, stock handshakes")
- 3-5 reference images uploaded by the owner
- Owner can **regenerate** any image or **upload their own** at approval

### Providers (connected per client via BYOK)
| Provider | Best for | Approx. cost / image (2026) |
|---|---|---|
| **Flux 2 Pro** (Black Forest Labs, fal.ai, Replicate) | Photoreal blog heroes. **Recommended default** | $0.015-0.05 |
| Google Imagen 4 / Gemini image | High quality, consistent style | $0.02-0.06 |
| OpenAI GPT Image | Complex scenes, cheapest mini tier | $0.005+ |
| Ideogram 3 | Images with readable text (titles on graphics) | $0.03-0.09 |
| Higgsfield API | Teila's current tool. SPC sites only (see below) | Per image, published rates |

**Estimated image cost per client:** 12 posts/month x ~4 images x 2 options = about 100 images = **$2-5/month** on the client's own key.

### Higgsfield: SPC only, not shared with clients
- Higgsfield's terms say you **cannot resell, sublicense or make Developer Access available to third parties, or act as a pass-through**. Generated images **can** be used commercially, including for client work
- **Use Higgsfield for the 4 SPC test sites** (your own sites, your own key)
- **Clients connect their own image account** (Flux via fal.ai, Google, OpenAI, or their own Higgsfield key)
- Read the current terms before offering a "we generate images for you" managed plan

---

## 9. Bring Your Own Key (BYOK)

Each client connects their own AI accounts. AI usage is billed to them by the provider, not to SPC.

### What each client connects
| Connection | Options | Required |
|---|---|---|
| **Text AI** | Anthropic (Claude), OpenAI, Google Gemini | Yes |
| **Image AI** | fal.ai (Flux), Google, OpenAI, Ideogram, Higgsfield | Yes (or "upload my own images" mode) |
| **Search Console** | Google OAuth (read-only) | Yes |
| **DataForSEO** | SPC's account (included in plan) **or** client's own key | Optional BYOK |

### How the app handles keys
1. **Settings > Connections** page: client pastes the key, the app runs a **test call** and shows "Connected" or the exact error
2. Keys are **encrypted at rest** (AES-256, per-tenant encryption, master key in the host's secret manager), never shown again in full (last 4 characters only), never logged, never sent to the browser
3. **Provider layer:** one internal `generateText()` / `generateImage()` interface, with an adapter per provider. Swapping providers doesn't change the pipeline
4. **Model per job** (defaults, client can change): drafting = top-tier model, QA/scoring = mid-tier, alt text/meta = low-cost tier
5. **Usage log per client**: tokens, images, estimated cost per post. Shown in the monthly report
6. **Failure handling:** invalid, expired or out-of-credit key means the job **pauses** (no silent skip), the owner and Teila get an email, and the job resumes when the key is fixed
7. **Spend cap** per client per month (optional). The job pauses at the cap

### SPC test phase
All 4 SPC sites run on **Teila's keys**: Anthropic for text, Higgsfield for images, DataForSEO, and Search Console for each property.

---

## 10. Email notifications (owner + Teila)

| Email | When | Contains |
|---|---|---|
| **Weekly Insight Request** | Monday | 3 topics, outlines, questions, link to answer (text or voice) |
| **Insight reminder** | 24h before the deadline | Link |
| **Approval Request** | Draft passes QA | Preview link, QA score, images, Approve / Request edits |
| **Published** | Post goes live | Live URL, social pack link |
| **Action needed** | Key failed, publish failed, site webhook down | Exact error + fix link |
| **Monthly Report** | 1st of the month | Posts, rankings, traffic, CTA leads, AI cost |

Sender: a transactional email service (e.g. Resend or Postmark) on a verified SPC sending domain, with reply-to set to Teila.

---

## 11. Open decisions (for build session)

1. **Exact domains** for Esquire Pilot and Find George Atty. Are all 4 sites Next.js in Teila's codebase (one repo or separate repos)?
2. **CTAs** for all 4 SPC sites (Teila to provide)
3. **Image style preset** per site (4 presets)
4. **Hosting** for the app (Vercel / Hostinger VPS / other) + database (Postgres recommended)
5. **Pricing / packaging** for resale (run through spc-quote-builder once the build is scoped)

---

## Sources
- [Higgsfield - Terms of Use](https://higgsfield.ai/terms-of-use-agreement)
- [Higgsfield - Who owns your generations](https://higgsfield.ai/creator-hub/help-center/account/who-owns-my-generations-and-can-i-use-them-commercially)
- [Higgsfield API pricing](https://open.higgsfield.ai/pricing)
- [BuildMVPFast - AI image API pricing (July 2026)](https://www.buildmvpfast.com/api-costs/ai-image)
- [Atlas Cloud - Best AI image generation APIs 2026](https://www.atlascloud.ai/blog/guides/best-ai-image-generation-apis-in-2026-complete-developer-guide)
