# Blog SEO Rulebook - Built Into the Blog Generator App

**As of September 2026.** Companion to `blog-generator-research.md`.
Every rule below is written as something the app must **generate**, **check**, or **schedule**.
Numbers are starting defaults. Tune per client from Search Console data.

---

## 1. What search engines reward (priority order)

| # | Signal | What it means for a blog | App module that handles it |
|---|---|---|---|
| 1 | **Search intent match** | Post format matches what already ranks (guide vs. list vs. FAQ vs. comparison) | SERP Brief |
| 2 | **E-E-A-T** (experience, expertise, authority, trust) | Real author, credentials, first-hand insight, cited sources, accurate facts | Author Profile + Fact-Check + Approval Gate |
| 3 | **Helpful / original content** | Adds something the top 10 don't have. Runs sitewide, so weak posts drag down the whole site | Info-Gain step + QA Score |
| 4 | **Topical authority** | Many interlinked posts covering one subject in depth | Cluster Planner + Link Engine |
| 5 | **Core Web Vitals / page experience** | Fast, mobile-friendly, stable page | Tech Health Monitor |
| 6 | **Backlinks and brand mentions** | Other sites citing you | Off-site (Section 9), not the blog engine |
| 7 | **Freshness** | Posts updated when facts or rankings change | Refresh Engine |
| 8 | **Local signals** (law firms) | Google Business Profile, reviews, city-level content | Local Module |

Google does not penalize AI content. It penalizes unhelpful, unreviewed volume (scaled content abuse, March and August 2026 updates).

---

## 2. Keyword rules (how many, where)

### How many per post

| Type | Count | Rule |
|---|---|---|
| **Primary keyword** | **1** | One per post. Never reuse a primary keyword on a second post. |
| **Secondary keywords** | **3-5** | Same topic group, same intent. Pulled from the Keyword Library cluster. |
| **Semantic / related terms** | 10-20 | Entities and phrases the top-ranking pages use (from the SERP Brief). Used naturally, not counted. |
| **Questions (PAA)** | 3-5 | "People Also Ask" questions. Become H2s or FAQ items. |

### Keyword density
Don't aim for a target number. The app only runs a **stuffing check**:
- Primary keyword density **over 2% = FAIL** (rewrite)
- **0.5-1.5% = normal range**. It's reported, not enforced.

### Primary keyword placement map (app checks each)

| Location | Required |
|---|---|
| SEO title (front-loaded) | Yes |
| H1 | Yes |
| URL slug | Yes |
| First 100 words | Yes |
| At least 1 H2 | Yes |
| Meta description | Yes |
| Featured image alt text | Yes |
| Image file name | Yes |
| Conclusion / CTA paragraph | Recommended |

Secondary keywords: each appears at least once, ideally in an H2 or H3.

---

## 3. Post blueprints by type

The SERP Brief picks the type from what ranks now. Length is **depth-driven**. The app sets a word-count target from the average of the top 5 ranking pages, capped by the type range below.

| Type | Word range | When to use | Share of calendar |
|---|---|---|---|
| **Pillar page** | 2,500-4,000 | Broad topic hub ("Car Accident Claims in Texas") | 1 per cluster |
| **How-to / guide** | 1,500-2,500 | Process queries ("how to file a claim after...") | ~40% |
| **FAQ / quick answer** | 800-1,300 | Single-question queries, AI Overview targets | ~30% |
| **List / checklist** | 1,200-1,800 | "What to do after...", "X mistakes..." | ~20% |
| **Comparison / cost** | 1,500-2,000 | "Settlement vs. lawsuit", "how much does..." | ~10% |

Top-10 Google results average about 2,400 words. Pages cited in AI Overviews average about 1,300 words, and more than half are under 1,000. The mix above covers both.

---

## 4. On-page SEO checklist (app validates before the approval step)

| Element | Rule |
|---|---|
| **SEO title** | 50-60 characters, keyword first, 1 per post, unique |
| **H1** | Exactly 1. Can differ slightly from the SEO title |
| **Meta description** | 120-155 characters, keyword included, ends with a benefit or CTA, unique |
| **URL slug** | 3-5 words, lowercase, hyphens, keyword, no dates or stop words |
| **Headings** | H2 every 200-300 words. H3 for sub-points. No skipped levels |
| **Paragraphs** | 2-3 sentences max. Short paragraphs get extracted by AI |
| **Opening** | 40-60 word direct answer to the keyword question in paragraph 1 |
| **Table of contents** | Posts over 1,500 words |
| **Images** | 1 featured (1200x630) + 1 inline image per ~500 words. WebP, under 150KB, descriptive file name, alt text on every image |
| **Internal links** | 3-5 per post: 1 to the pillar page, 1 to the matching service page, 1-3 to sibling cluster posts. Descriptive anchor text, no "click here" |
| **External links** | 2-4 to authoritative sources (statutes, .gov, court rules, studies). Opens in a new tab |
| **FAQ section** | 3-5 questions from People Also Ask |
| **CTA** | 1 mid-post + 1 end-of-post (consult / intake form) |
| **Author box** | Name, credentials, bio, photo, link to author page |
| **"Reviewed by"** | Business owner (or named expert) + review date. All industries |
| **Last updated date** | Shown on the page, updated on every refresh |
| **Schema (JSON-LD)** | `BlogPosting` + `Person` (author) + `FAQPage` + `BreadcrumbList`. Helps Google understand the page and earn rich results. A May 2026 Ahrefs study found no AI-citation boost from schema, so it's hygiene, not a ranking lever |
| **Open Graph** | OG title, description and image for social shares |
| **Canonical** | Self-referencing canonical URL |
| **Readability** | Grade 7-9 level (Flesch-Kincaid). Plain language for PI audiences |

---

## 5. AI search optimization (Google AI Overviews, ChatGPT, Perplexity)

About 60% of Google AI Overview citations come from brand sites, so on-page work pays off there first. ChatGPT leans on Reddit, Wikipedia and news, which is off-site work.

App rules:
1. **Answer first.** Direct answer in the first 40-60 words.
2. **Question-style H2s** that match how people ask ("How long do I have to file...").
3. **Answer under each H2** in the first 1-2 sentences, then the detail.
4. **Short paragraphs, lists and tables.** Easy for AI to extract.
5. **Neutral, factual tone** in answer blocks. Save the sales copy for the CTA.
6. **Specific facts**: numbers, deadlines, statute citations. AI cites specifics.
7. **Consistent entity data**: business name, address and author/owner names identical everywhere.
8. **Track AI visibility.** DataForSEO AI Optimization endpoints (LLM mentions) run monthly per client.

---

## 6. Site architecture: topic clusters

Isolated posts don't build authority. Clusters do.

```
                 [Service Page: Car Accident Lawyer - City]
                               ^
                               |
                 [PILLAR: Car Accident Claims in (State)]  2,500-4,000 words
               /        |          |          |         \
        [Cluster] [Cluster]  [Cluster]  [Cluster]  [Cluster]   8-15 posts
        what to do  SOL        settlement  insurance  uninsured
        after       deadline   value       tactics    driver
```

**Cluster rules (app enforces):**
- Every Keyword Library entry is assigned a `cluster_id` and a `pillar_id`
- **8-15 cluster posts per pillar**
- The **pillar is written first**, or before the 3rd post in that cluster
- Every cluster post links **up** to the pillar using the pillar keyword as anchor text
- The pillar links **down** to every cluster post. The Link Engine updates the pillar on each new publish
- Cluster posts link **sideways** to 1-2 siblings
- The pillar links to the **money page** (service/practice-area page)
- **Finish one cluster before opening the next**: at least 70% complete before a new pillar starts

**Starting cluster set for a PI firm:** car accidents, truck accidents, motorcycle, slip and fall, wrongful death, dog bites, insurance claims process, medical treatment after injury, settlement and compensation, hiring a lawyer.

---

## 7. Publishing cadence and refresh

| Rule | Default |
|---|---|
| New posts | **3/week** (Mon/Wed/Fri). Only while every post passes the QA score |
| Consistency | Fixed days. Twelve posts at one a month beats twelve posts in one week and then nothing |
| Refresh ratio | 0-50 posts: new only. 50-150 posts: 1 refresh per new post. 150+ posts: 2 refreshes per new post |
| Time to peak | A post takes 3-6 months to reach its traffic ceiling. Law firm SEO ROI shows at 6-12 months. Set this expectation with clients |

**Refresh triggers (Refresh Engine checks Search Console weekly):**

| Trigger | Action |
|---|---|
| Position 8-20 for its primary keyword, 60+ days old | **Expand**: add sections for queries it ranks for but doesn't cover, add FAQs, more internal links |
| Top 5 position but CTR under 3% | **Rewrite title and meta description only** |
| Clicks down 30%+ compared with the prior 90 days | **Full refresh**: update facts, add new SERP subtopics, new date |
| Law, deadline or statistic changed | **Immediate fact update** |
| Two posts ranking for the same query | **Merge** into the stronger post, 301-redirect the weaker one |
| Evergreen post | Review every 6-12 months |
| Pillar page | Review quarterly |

Refreshing old posts has driven traffic gains of 100%+ in documented cases (HubSpot, Backlinko).

---

## 8. Technical health (site-level, monitored monthly)

The blog can't outrank a slow or broken site. The app runs **DataForSEO `on_page_lighthouse`** on new posts and a monthly sample.

| Metric | Pass threshold |
|---|---|
| LCP (Largest Contentful Paint) | 2.5s or less |
| INP (Interaction to Next Paint) | 200ms or less |
| CLS (Cumulative Layout Shift) | 0.1 or less |
| Mobile-friendly | Required |
| HTTPS | Required |
| XML sitemap | Auto-updated, submitted in Search Console |
| Indexing | Search Console URL inspection 7 days after publish. Flag if not indexed |
| Broken links | 0 (monthly crawl) |
| Orphan posts | 0. Every post has 2+ internal links pointing in |

Failures produce a **fix ticket** (ClickUp/GHL task), not an automatic change.

---

## 9. Traffic growth outside the blog (the app tracks or triggers these)

| Lever | What the app does |
|---|---|
| **Google Business Profile** | Auto-drafts 1 GBP post per blog (summary + link). For law firms, GBP is the #1 local ranking factor. Primary category must be specific ("Personal Injury Attorney", not "Lawyer") |
| **Reviews** | Reports monthly review count and velocity. Blog content doesn't replace reviews |
| **Social distribution** | 3 social posts per blog via GHL Social Planner (your 5-post framework) |
| **Email** | Monthly newsletter digest of new posts (GHL) |
| **Backlinks** | Monthly DataForSEO backlink summary + new/lost links. Flags linkable posts (stats, guides) for outreach |
| **Local citations** | Consistent firm name, address and phone. Checked quarterly |
| **Off-site AI visibility** | Monthly check of whether the firm is mentioned in LLM answers for its top 10 keywords |

---

## 10. Industry compliance packs (optional add-on per client)

**Owner approval is required for every client, every industry** (see `blog-app-product-spec.md`). The packs below add rules on top of it.

### Law firm pack

- **The approving owner must be a licensed attorney** (ABA Formal Opinion 512, Model Rule 7.1). Name and date go in the audit log
- **Banned claims**: "guarantee", "best", "#1", "expert/specialist" (unless certified), promised outcomes, unverified settlement amounts
- **Disclaimers**: "Attorney Advertising" and "not legal advice", per the state bar
- **Jurisdiction-specific**: statute of limitations, comparative fault and damage caps cited to the state statute
- **Case results** only with the required past-results disclaimer
- **No client-identifying details** in examples

---

## 11. How the app enforces this: QA Content Score (100 points)

Every draft is scored before it reaches the approval step.
**Under 80 = automatic rewrite (max 2 attempts), then flagged for a human.**
**Any hard-fail item = blocked regardless of score.**

| Category | Points | Checks |
|---|---|---|
| **Keyword placement** | 15 | Primary keyword in title, H1, slug, first 100 words, H2, meta, alt text. Secondaries used. Density under 2% (hard fail) |
| **Intent and depth** | 20 | Format matches the SERP. Covers the subtopics shared by the top 5 pages. Word count within the target range |
| **Information gain** | 10 | At least 1 element the top 10 don't have: local data, **Owner Insight** answers, owner quote, checklist, example |
| **Structure / AEO** | 15 | Answer-first opening. Question H2s. Short paragraphs. FAQ block. Table of contents if needed |
| **E-E-A-T** | 15 | Author + owner approval. 2+ authoritative citations. Facts verified (hard fail on an unverified statute or deadline) |
| **Links** | 10 | 3-5 internal (pillar + service page required). 2-4 external. No broken links |
| **Meta and media** | 10 | Title/meta lengths. Slug rules. Images with alt text, WebP, correct sizes. Schema valid. OG tags |
| **Readability and brand** | 5 | Grade 7-9. Brand voice match. Zero banned claims (hard fail) |

The score, the failing checks and the fixes are logged against the post in the Keyword Library.

---

## 12. Updated app module map

| Module | Runs | Inputs | Outputs |
|---|---|---|---|
| **Onboarding** | Once per client | Form, site crawl, Search Console access | Brand Profile, existing-content index, author profiles |
| **Keyword Engine** | Weekly | DataForSEO, Search Console, competitors | Scored keywords into the Library |
| **Cluster Planner** | Weekly | Library | cluster_id, pillar_id, publishing order (pillar first) |
| **SERP Brief** | Per post | DataForSEO SERP, People Also Ask | Post type, word target, subtopics, questions, info-gain angle |
| **Writer** | Per post | Brief, Brand Profile | Draft + full metadata package |
| **Fact-Check** | Per post | Draft | Verified claims, citation list |
| **Link Engine** | Per post | Existing-content index | Internal links in the new post, pillar updated with a link back |
| **Image Engine** | Per post | Brief | Featured + inline images, alt text, WebP |
| **QA Score** | Per post | Draft package | Score, pass/fail, rewrite loop |
| **Owner Insight** | Per post, before drafting | Brief + 3-5 questions | Owner's stories, opinions, examples (or skipped) |
| **Approval Gate** | Per post | Passing draft | Owner approves / requests edits in the app portal |
| **Publisher** | Mon/Wed/Fri | Approved post | Live post via Content API (custom sites) or connector (WordPress / GHL later), sitemap updated |
| **Distribution** | On publish | Live post | GBP post, 3 social posts, newsletter queue |
| **Index Check** | Publish + 7 days | Search Console URL inspection | Indexed status, alert if not indexed |
| **Refresh Engine** | Weekly | Search Console performance | Refresh / rewrite-meta / merge queue |
| **Tech Health** | Monthly | DataForSEO Lighthouse + crawl | Core Web Vitals, broken links, orphans, fix tickets |
| **Authority Monitor** | Monthly | DataForSEO backlinks + LLM mentions | Links gained/lost, AI visibility |
| **Client Report** | Monthly | All of the above | Dashboard: posts, rankings, traffic, leads |

### New Keyword Library fields
`cluster_id`, `pillar_id`, `is_pillar`, `post_type`, `word_target`, `paa_questions`, `qa_score`, `qa_fails`, `reviewer`, `reviewed_at`, `indexed`, `last_refreshed`, `refresh_reason`

---

## Sources
- [Sapphire SEO - How many keywords per blog post (2026)](https://sapphireseosolutions.com/blog/how-many-keywords-per-blog-post-2026)
- [Shopify - Keyword density best practices (2026)](https://www.shopify.com/blog/keyword-density-seo)
- [Analytify - Google ranking factors 2026](https://analytify.io/google-ranking-factors/)
- [Keywords Everywhere - E-E-A-T 2026 playbook](https://keywordseverywhere.com/blog/google-e-e-a-t-guidelines-an-overview/)
- [Bluehost - Ideal blog post length 2026](https://www.bluehost.com/blog/ideal-blog-post-length/)
- [ClickRank - Content length and AI Overviews](https://www.clickrank.ai/ideal-content-length-for-seo/)
- [Frase - GEO playbook](https://www.frase.io/blog/how-to-get-cited-by-ai-search-engines-the-complete-geo-playbook)
- [LLMrefs - GEO 2026 guide](https://llmrefs.com/generative-engine-optimization)
- [Conductor - Topic clusters](https://www.conductor.com/academy/topic-clusters/)
- [Digital Applied - Content clusters 2026](https://www.digitalapplied.com/blog/seo-content-clusters-2026-topic-authority-guide)
- [inblog - Publishing frequency data](https://inblog.ai/blog/publishing-frequency-data)
- [Shno - Content refresh statistics 2026](https://www.shno.co/marketing-statistics/content-refresh-statistics)
- [SeekLab - Titles and meta descriptions 2026](https://seeklab.io/blog/on-page-seo-titles-metas-structure/)
- [Google - Search Analytics API](https://developers.google.com/webmaster-tools/v1/searchanalytics)
- [PILMMA - Local SEO for law firms 2026](https://www.pilmma.org/blog/what-local-seo-looks-like-for-law-firms-in-2026/)
- [Rankings.io - SEO for lawyers 2026](https://rankings.io/seo-for-lawyers/)
