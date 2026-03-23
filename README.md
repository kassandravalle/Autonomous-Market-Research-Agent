Market Intelligence Agent
By Kasi Systems | Built by Kassandra Valle

An agentic system that continuously monitors AI industry signals, enriches them with Claude, and generates multi-channel content — Slack briefs, LinkedIn posts, and newsletters — automatically.


System Overview
RSS Feeds → n8n Ingestion → Claude Enrichment → Supabase → Content Generation → Slack / LinkedIn / Newsletter
Core Stack
ToolRolen8n CloudOrchestration — scheduling, routing, error handlingClaude API (Sonnet)Intelligence — enrichment, scoring, content generationSupabase (Postgres)Storage — all data, state, and error loggingSlackDelivery — daily briefGitHubVersion control — workflow exports and prompt versioning

Database Schema
feed_sources
Your RSS feed registry. Add/remove sources here without touching n8n.
ColumnTypeNotesidUUID PKAuto-generatednameTEXTHuman-readable name e.g. "OpenAI Blog"urlTEXT UNIQUERSS feed URLcategoryTEXTe.g. "AI Research", "VC/Funding", "Dev Tools"is_activeBOOLEANToggle sources on/off without deletinglast_fetched_atTIMESTAMPTZUpdated after every successful fetchcreated_atTIMESTAMPTZAuto-set
incoming_articles
Everything ingested, untouched. Deduplication happens here via UNIQUE on url.
ColumnTypeNotesidUUID PKAuto-generatedsource_idUUID FKReferences feed_sourcestitleTEXTArticle titleurlTEXT UNIQUEDeduplication key — Postgres rejects duplicate URLscontent_snippetTEXTFirst ~500 chars of article bodypublished_atTIMESTAMPTZFrom RSS feedingested_atTIMESTAMPTZWhen n8n pulled itis_processedBOOLEANn8n gate — FALSE = needs enrichment, TRUE = done
ai_insights
Claude-enriched version of each article. One row per article.
ColumnTypeNotesidUUID PKAuto-generatedarticle_idUUID FKReferences incoming_articlescategoryTEXTe.g. "Model Release", "Funding", "Product Launch"relevance_scoreINT (1–10)Claude scores business relevancekey_insightTEXTWhat happened — 1–2 sentencesso_whatTEXTWhy it matters — Kasi brand voicesuggested_angleTEXTContent angle for LinkedIn/newsletterraw_llm_outputJSONBFull Claude response — always storedenriched_atTIMESTAMPTZAuto-set
generated_content
One row per generated output. Format distinguishes content type.
ColumnTypeNotesidUUID PKAuto-generatedinsight_idUUID FKReferences ai_insightsformatTEXTlinkedin_post | slack_brief | newsletter_section | youtube_scriptdraft_bodyTEXTGenerated contentstatusTEXTdraft → approved → publishedgenerated_atTIMESTAMPTZAuto-setapproved_atTIMESTAMPTZSet when status moves to approved
processing_errors
Every enrichment failure lands here. Workflow never crashes — it routes here instead.
ColumnTypeNotesidUUID PKAuto-generatedarticle_idUUID FKReferences incoming_articleserror_typeTEXTe.g. json_parse_error, api_timeout, empty_responseerror_messageTEXTHuman-readable descriptionraw_payloadJSONBExactly what Claude returned — for debuggingfailed_atTIMESTAMPTZAuto-set
run_log (add via Step 4 SQL)
Execution history for every workflow run.
ColumnTypeNotesidUUID PKAuto-generatedworkflow_nameTEXTe.g. daily_ingestion, enrichment, content_generationarticles_fetchedINTHow many new articles came inarticles_enrichedINTHow many were successfully enrichederrors_countINTHow many failedstarted_atTIMESTAMPTZRun startcompleted_atTIMESTAMPTZRun endstatusTEXTsuccess | partial | failed

Workflow Architecture
Workflow 1 — Daily Ingestion (daily_ingestion)
Trigger: Cron — 6:00 AM daily
Job: Pull new articles from all active feed sources, normalize, deduplicate, write to incoming_articles
Schedule Trigger
  → Get active feed_sources from Supabase
  → Loop: Fetch RSS for each source
  → Normalize article record
  → Upsert to incoming_articles (ON CONFLICT url DO NOTHING)
  → Update last_fetched_at on feed_sources
  → Write run summary to run_log
Workflow 2 — AI Enrichment (enrichment)
Trigger: Cron — 6:15 AM daily (after ingestion) OR webhook from Workflow 1
Job: Find unprocessed articles, enrich with Claude, write to ai_insights
Schedule / Webhook Trigger
  → Query incoming_articles WHERE is_processed = FALSE
  → Limit to 20 per run (rate limit protection)
  → Loop each article:
      → Build LLM prompt
      → Call Claude API
      → Validate JSON response
      → On success: write to ai_insights, set is_processed = TRUE
      → On failure: write to processing_errors, continue loop
  → Write run summary to run_log
Workflow 3 — Content Generation (content_generation)
Trigger: Cron — 7:00 AM daily
Job: Take top insights, generate Slack brief + LinkedIn draft
Schedule Trigger
  → Query ai_insights WHERE relevance_score >= 7
    AND enriched_at >= NOW() - INTERVAL '24 hours'
  → Generate Slack brief (top 3–5 insights)
  → Generate LinkedIn post draft (highest scored insight)
  → Write both to generated_content with status = 'draft'
  → POST Slack brief to channel via webhook

Claude Prompt Architecture
Enrichment Prompt (Workflow 2)
One job: Analyze a single article and return structured JSON.
Never ask Claude to generate content here. Enrichment and generation are separate calls.
System: You are a market intelligence analyst specializing in AI industry signals
for B2B operators and founders. Your job is to analyze a single article and return
a structured JSON object. Return ONLY valid JSON. No preamble, no explanation,
no markdown code fences.

User: Analyze this article:
Title: {title}
Source: {source_name}
Snippet: {content_snippet}

Return this exact JSON structure:
{
  "category": "Model Release | Funding | Product Launch | Research | Regulation | Other",
  "relevance_score": 1-10,
  "key_insight": "1-2 sentence summary of what happened",
  "so_what": "1 sentence on why this matters for B2B operators and founders",
  "suggested_angle": "1 content angle for a LinkedIn post or newsletter"
}
Content Generation Prompt (Workflow 3)
Separate call, separate job. Uses enriched data as input, not raw articles.

Scalability Decisions
DecisionWhyurl UNIQUE constraintDeduplication at DB level — zero extra logic in n8nis_processed flagStateless — workflow can re-run safely without double-processingraw_llm_output JSONBReprocess failed parses without re-calling Claude APIPartial index on is_processed = FALSEQuery stays fast at 100k+ rowsLimit 20 articles per enrichment runClaude API rate limit protectionSources stored in DB, not n8nAdd/remove sources without touching workflowSeparate error tableFailures are data, not crashesrun_log tableObservability — know exactly what ran, when, and what broke

Folder Structure (GitHub)
market-intelligence-agent/
├── README.md                  ← This file
├── schema/
│   ├── 01_create_tables.sql
│   ├── 02_indexes.sql
│   ├── 03_run_log.sql
│   └── 04_seed_sources.sql
├── prompts/
│   ├── enrichment_prompt.md
│   └── content_generation_prompt.md
├── workflows/
│   ├── 01_daily_ingestion.json     ← n8n export
│   ├── 02_enrichment.json
│   └── 03_content_generation.json
└── docs/
    └── demo_script.md

RSS Feed Sources (Seed Data)
NameCategoryOpenAI BlogAI ResearchGoogle DeepMindAI ResearchSimon Willison's BlogDeveloperEthan Mollick's BlogAI StrategyTomasz Tunguz BlogVC/FundingStrictlyVCVC/FundingTechCrunch AIIndustry NewsWired AIIndustry News

Demo Flow (Moxo Interview)

Show feed_sources table — "These are the 8 sources the system monitors daily"
Show incoming_articles — "This is everything that came in this morning, normalized and deduplicated automatically"
Show ai_insights — "Claude scored and enriched each one — relevance score, key insight, the so-what"
Show generated_content — "From the top-scoring insights, it generated a Slack brief and a LinkedIn draft"
Show Slack — "This hit my channel at 7am without me touching anything"
Show processing_errors — "And anything that broke landed here — the system never crashes, it routes failures"
