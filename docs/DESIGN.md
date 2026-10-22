# Design notes

## Stages
1. **Input**: the form is normalised (signals become `recent_funding`, `hiring_growth`, …; job titles always become an array).
2. **Company discovery**: up to 3 Apollo searches run in parallel and are tagged with their source branch (native match or base).
3. **Normalise**: URLs are cleaned; LinkedIn-only companies are flagged; companies with no website and no LinkedIn are dropped.
4. **Website analysis** (per company): fetch with an 18 s timeout and a retry; dead-site filter; extraction of title, meta, OG tags, H1/H2, JSON-LD, careers/pricing/blog links, embedded tools (Intercom, Drift, HubSpot, Salesforce, Calendly, Zendesk, LiveChat).
5. **Decision maker**: Apollo people search (up to 5, filtered by the selected titles), then bulk match on the top person for email and phone; a phone-lookup row is logged in parallel.
6. **AI enrichment and scoring**: LLM analysis → parse → weighted score; the loop continues.
7. **Output**: filter skipped leads → rank → dedupe against the sheet → keep the top `Max_lead` → append rows and email the report.
8. **Phone callback**: asynchronous webhook → `Phone_lookup` update matched on Person ID.

## Buying signals (11)
Recent funding*, promoted executives, new client signed, partnerships, office expansion, merger or acquisition, hiring growth*, leadership changes, product launches, technology adoption, revenue growth.
\* filtered natively by Apollo; the rest are detected by keyword in the cleaned page text.

## Healthcare vertical scan (9 categories)
Patient acquisition, patient engagement, practice management, telehealth, provider management, EHR, revenue cycle management, clinical trials, EMR integrations. Each matched category adds 2 points to a 0–10 healthcare fit score.

## LLM prompt contract
The system prompt asks for **raw JSON only** with: `industry, business_summary, target_customer, automation_opportunities, key_pain_points, personalized_message, ai_readiness, automation_need, growth_signal`.
- The message must be one paragraph of 60–120 words, start with an observation about the company, cite one pain point and one opportunity, and end with a 15-minute meeting ask.
- If the site content is under 200 characters, the model must use neutral scores (3/5/3) rather than punishing the company for a thin site.
- Website text is passed between clear `WEBSITE_CONTENT_START/END` markers.
