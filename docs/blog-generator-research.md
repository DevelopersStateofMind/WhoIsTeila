# Blog Generator App - Research Brief

**As of September 2026.** Research only. Build happens in a separate session.

SEO rules, keyword counts, QA scoring and the full module map: see [`blog-seo-rulebook.md`](blog-seo-rulebook.md).

> **Superseded in part by [`blog-app-product-spec.md`](blog-app-product-spec.md):** owner approval (not attorney) for all clients, custom sites via a Content API first (WordPress/GHL later), Owner Insight step, CTA library.

---

## 1. Verdict: Is this already done?

Yes, the category exists ("autoblogging" / "AI SEO agent"). No off-the-shelf tool does your exact spec:
DataForSEO + Google Search Console cross-referenced against a brand profile, feeding a living keyword library, with a law-firm compliance gate.

| Tool | What it does | Price (Sept 2026) | Gap vs. your spec |
|---|---|---|---|
| Outrank | Keyword research, SERP analysis, auto-publish on schedule, backlink exchange | $99/mo, 30 articles | No attorney review gate, you don't control the keyword scoring |
| SEObot | Full autopilot, backlinks | $49/mo (9), $99 (20), $199 (50) | Zero-involvement model = risky for law firms |
| RightBlogger | Keyword research + WordPress auto-publish | Lite $17.99 to Business $69.99/mo | Writing-first, lighter automation |
| BlogSEO | Keyword research, internal linking, brand voice, CMS publishing | ~$97/mo | Closest to spec; still no GSC + DataForSEO cross-reference you own |
| ClearPost (WP plugin) | Autonomous SEO agent, connects to GSC | WordPress only | WordPress only, no GHL |
| Koala AI | SERP-grounded drafts | From $9/mo | You pick keywords, you publish |
| GHL Blog Post AI (Labs) | Drafts in GHL blog editor, Brand Voice | Included in GHL | Manual, no keyword library, no scheduling engine |
| n8n templates (#6117, #11135, #3085) | Keyword > GPT draft > image > WordPress, Sheets tracking | Free (self-hosted n8n) | Starting point only; no GSC loop, no brand scoring |

## 2. Top 2 options

**Option A - Buy (BlogSEO or Outrank).** Fastest. ~$100/mo per site. You lose control of keyword logic, compliance review, and margin. Not resellable as an SPC system.

**Option B - Build on n8n (RECOMMENDED).** Your stack already has every piece: self-hosted n8n, DataForSEO, GSC API, Claude, GHL Blog API (public API: posts, categories, authors, slug check), WordPress REST API. You own the logic, add the attorney review gate, and productize it as a recurring SPC service across PI firms. Fork n8n template #6117 or #11135 as scaffolding.

## 3. Architecture (Option B)

```
[Client Onboarding Form]
   -> Brand Profile (services, practice areas, geo, voice, banned claims, competitors, CTA)
        |
[Keyword Engine - weekly, n8n Schedule]
   DataForSEO: keywords_for_site, keyword_ideas, related_keywords, search_intent, bulk_keyword_difficulty
   GSC API:   Search Analytics (query, page, impressions, position) last 90 days
   Competitors: DataForSEO domain_intersection / ranked_keywords on 2-3 competitors
        |
   Claude: Brand-fit scoring (0-10) against Brand Profile + cluster assignment
        |
[Keyword Library - n8n Data Table / Airtable / Sheet]
        |
[Content Engine - Mon/Wed/Fri]
   Pick top-scored "Queued" keyword -> SERP brief (serp_organic_live_advanced)
   -> Outline -> Draft (Claude) -> QA pass (facts, claims, cannibalization, internal links)
   -> Images (featured + 1-2 inline, alt text) -> SEO meta package
        |
[Approval Gate]  -> Attorney/owner approves in GHL task, ClickUp, or email link
        |
[Publisher]  WordPress REST API  OR  GHL Blog API  (status: scheduled)
        |
[Feedback Loop - 28 / 90 days post-publish]
   GSC: impressions, clicks, position per URL
   Positions 8-20 -> "Refresh" queue; CTR < 3% at top-5 -> rewrite title/meta
        |
[Repurpose]  GHL Social Planner: LinkedIn/FB/IG posts per blog
[Client Report]  Monthly dashboard
```

## 4. Keyword Library spec

**Rule: one primary keyword per blog + 3-5 secondary keywords from the same cluster.** Never two posts on the same primary keyword (cannibalization).

| Field | Source |
|---|---|
| keyword | DataForSEO / GSC |
| search_volume, KD, CPC | DataForSEO |
| intent (info / commercial / transactional / navigational) | DataForSEO search_intent |
| gsc_impressions, gsc_position, ranking_url | GSC API |
| brand_fit_score (0-10) + reason | Claude vs. Brand Profile |
| cluster / pillar page | Claude clustering |
| priority_score | formula below |
| status | New > Queued > Drafting > In Review > Scheduled > Published > Refresh |
| post_url, publish_date | Publisher |

**Priority score (starting formula, tune per client):**
`brand_fit x 0.35 + (volume, normalized) x 0.20 + (100 - KD) x 0.20 + GSC opportunity x 0.25`
GSC opportunity = keyword already gets impressions but no dedicated page, or ranks 8-20.

**Replenishment rule:** if Queued < 12 (4 weeks at 3/wk), trigger the Keyword Engine early.

## 5. Per-post output package (what you listed + what's missing)

**You listed:** blog body, images, tags, SEO title, meta description, keywords.

**Add these:**
1. **URL slug** (check uniqueness via GHL `check-url-slug-exists` or WP)
2. **Excerpt** + **category** (map to practice area)
3. **Image alt text + WebP compression** (featured 1200x630, inline)
4. **Schema JSON-LD** - `BlogPosting`, `FAQPage`, `Person` (attorney author), `LegalService` / `LocalBusiness` (hygiene for rich results; a May 2026 Ahrefs study found no AI-citation boost)
5. **FAQ block (3-5 Qs)** - feeds AI Overviews / ChatGPT citations
6. **Answer-first opening paragraph** - 40-60 word direct answer (AEO/GEO)
7. **Internal links** - 2-4 to related posts + 1 to the matching practice-area page
8. **External citations** - statutes, court rules, government sources (E-E-A-T)
9. **CTA block** - free consultation / intake form (GHL form or calendar)
10. **Open Graph / social meta** - OG title, description, image
11. **Author + "Reviewed by [Attorney]" byline** - E-E-A-T and bar compliance
12. **Jurisdiction disclaimer** - "attorney advertising" / "not legal advice" per state bar
13. **Social repurpose pack** - 3 posts per blog (ties to your 5-post framework)

## 6. System features you're missing

| Feature | Why it matters |
|---|---|
| **Human approval gate** | ABA Formal Opinion 512 + Model Rule 7.1: the firm is responsible for AI content. Named reviewing attorney on every post. |
| **Cannibalization check** | Before drafting, compare keyword against existing posts (GSC ranking_url + site crawl). |
| **Content refresh loop** | Striking-distance (pos 8-20) posts get rewritten; often more ROI than new posts. |
| **Existing-content crawl on onboarding** | Pull all existing posts/pages so the library skips covered topics and internal linking works. |
| **Brand/claims guardrails** | Banned words, no "guarantee"/"best"/results claims, no case outcomes without disclaimers. |
| **Fact-check pass** | Second model pass verifying statutes, deadlines (SOL), dollar figures. |
| **Sitemap / indexing** | WordPress & GHL auto-update sitemaps. Use IndexNow for Bing. Do NOT use Google Indexing API (only for job postings/livestreams). |
| **Local SEO geo-modifiers** | "[practice area] + [city]" keywords; PI is local-intent heavy. |
| **Monthly client report** | Posts published, keywords gained, impressions/clicks trend. This is your retention asset. |
| **Error handling + audit log** | Every run logged; failed publish alerts you. |
| **Multi-tenant config** | One workflow, per-client config row (site, CMS, API keys, brand profile). |

## 7. Risk: cadence vs. Google spam policy

3 posts/week is safe **if** each post is distinct, reviewed, and adds value. Google's March 2026 core update and August 2026 spam update targeted scaled content abuse (mass AI pages, identical structure, no editorial review) with 50-80% traffic drops. Google does not penalize AI content itself; it penalizes unhelpful, unreviewed volume. The approval gate and firm-specific insight (attorney quote, local data, case-type specifics) are your protection.

## 8. Estimated run cost per client (Option B, estimates)

| Item | Est. monthly |
|---|---|
| DataForSEO (weekly keyword pulls + 12 SERP briefs) | $3-10 |
| Claude API (12 posts, research + draft + QA) | $5-15 |
| Image generation (12 featured + inline) | $2-8 |
| GSC API | Free |
| n8n self-hosted | Existing infra |
| **Total** | **~$10-35/client/mo** vs. ~$100 for Option A |

Verify current per-model pricing before quoting clients.

## 9. Open decisions (for build session)

1. CMS targets: WordPress only, GHL only, or both? (affects publisher node)
2. Approval channel: GHL task, ClickUp task, or email approve/reject link?
3. Image model: which provider (brand consistency vs. cost)?
4. Library storage: n8n Data Tables vs. Airtable vs. Google Sheet?
5. Is this SPC internal first, or a client product from day 1 (multi-tenant)?

## Sources

- [RightBlogger - 7 Best Autoblogging Tools (2026)](https://rightblogger.com/blog/best-autoblogging-tools)
- [BlogSEO - Best AI SEO Tools for Auto-Publishing 2026](https://www.blogseo.io/blog/best-ai-seo-tools-auto-publishing-2026)
- [ClearPost - Autoblogging.ai Alternatives](https://clearpostplugin.com/clearpost-vs-autoblogging-ai-seo-agent-vs-bulk-generator/)
- [Outrank vs SEObot (2026)](https://www.outrank.so/compare/seobot)
- [Ryan Doser - Outrank Alternatives with Pricing](https://ryandoser.com/outrank-alternatives/)
- [n8n template #6117 - AI SEO Blog Automation for WordPress](https://n8n.io/workflows/6117-ai-seo-blog-automation-for-wordpress-with-featured-images-end-to-end/)
- [n8n template #11135 - WordPress blog automation](https://n8n.io/workflows/11135-wordpress-blog-automation-ai-seo-content-images-scheduling-and-email-alerts/)
- [HighLevel API - Create Blog Post](https://marketplace.gohighlevel.com/docs/ghl/blogs/create-blog-post)
- [HighLevel - Blog Post AI](https://help.gohighlevel.com/support/solutions/articles/155000007201-create-blog-posts-with-ai)
- [DataForSEO pricing update](https://dataforseo.com/update/pricing-update-in-dataforseo-apis)
- [Google Search Console Search Analytics API](https://developers.google.com/webmaster-tools/v1/searchanalytics)
- [GSQI - August 2026 Google Spam Update](https://www.gsqi.com/marketing-blog/august-2026-google-spam-update-case-studies/)
- [Digital Applied - Scaled Content Abuse](https://www.digitalapplied.com/blog/scaled-content-abuse-google-march-update-ai-pages-decimated)
- [LaFleur - State Bar rules on AI-generated ads](https://lafleur.marketing/blog/navigating-state-bar-rules-on-ai-generated-ads-a-50-state-overview/)
- [Legal Reader - Law Firm AI Policy for Content](https://www.legalreader.com/law-firm-ai-policy-what-you-owe-clients-when-ai-writes-your-content/)
