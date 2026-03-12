# n8n Workflow Architecture

## Overview

The Autonomous Market Research Agent uses n8n as the orchestration layer for collecting signals, triggering AI analysis, and storing structured intelligence.

n8n manages the automation workflows that connect external data sources, language models, and the intelligence database.

---

# Workflow Pipeline

Scheduled Trigger
↓
Signal Collection
↓
Content Normalization
↓
AI Analysis (Claude)
↓
Structured Data Creation
↓
Supabase Storage
↓
Report / Insight Generation

---

# Workflow Modules

## 1. Scheduled Trigger

The system runs on scheduled intervals using n8n cron triggers.

Example schedule:

• every 6 hours  
• daily signal aggregation  
• weekly intelligence report

This ensures the system continuously collects signals without manual intervention.

---

## 2. Signal Collection

Signals are collected from external sources such as:

• AI news sites  
• startup launches  
• product announcements  
• research publications  
• developer tools updates  

Collection methods may include:

• RSS feeds  
• API requests  
• web scraping  
• webhook ingestion  

---

## 3. Content Normalization

Incoming signals are normalized into a standard format before analysis.

Example normalized signal structure:

{
"source": "AI Newsletter",
"title": "New AI model release",
"url": "https://example.com
",
"content": "Summary of development...",
"date": "2026-03-10"
}

This ensures consistent processing across different sources.

---

## 4. AI Analysis

Signals are sent to a language model for analysis.

The AI performs tasks such as:

• summarizing developments  
• identifying companies mentioned  
• detecting emerging trends  
• clustering related topics  
• extracting key insights  

Example outputs include:

trend_theme: "AI developer tools"
company: "Example AI Startup"
summary: "New open-source model released"
signal_type: "product launch"

---

## 5. Structured Intelligence Storage

Processed signals are stored in Supabase as structured records.

Example stored fields include:

• signal source  
• company name  
• summary  
• category  
• trend theme  
• relevance score  

This database becomes a searchable archive of AI ecosystem signals.

---

## 6. Intelligence Outputs

The system generates structured outputs such as:

• weekly market research reports  
• emerging trend alerts  
• startup activity summaries  
• categorized research archives  

These outputs can be delivered to platforms such as:

• Slack  
• Notion  
• Email  
