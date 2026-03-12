# System Architecture Overview

## Project

Autonomous Market Research Agent

An AI system that continuously collects signals across the AI ecosystem, analyzes trends, and produces structured intelligence reports.

---

## System Goal

The goal of this system is to transform scattered market signals into structured intelligence that can help operators, investors, and builders stay informed about emerging trends.

The system automates research workflows that would otherwise require manual monitoring of multiple information sources.

---

## High Level Architecture

Signal Sources
      ↓
Data Ingestion (n8n workflows)
      ↓
AI Analysis Layer (Claude)
      ↓
Structured Intelligence Database (Supabase)
      ↓
Automation Outputs (Reports / Alerts / Insights)

---

## Key Components

### Signal Sources

The system collects signals from sources such as:

- AI news websites
- product launches
- startup announcements
- research publications
- social signals
- developer community updates

---

### Data Ingestion

n8n orchestrates workflows that ingest signals using:

- scheduled jobs
- RSS feeds
- API requests
- web scraping
- webhook triggers

This layer normalizes incoming data before analysis.

---

### AI Analysis Layer

Signals are sent to Claude for processing.

The AI performs tasks such as:

- summarizing developments
- identifying trends
- clustering related topics
- extracting key insights

---

### Data Storage

Structured signals are stored in Supabase.

Example stored fields include:

- signal source
- company or product mentioned
- summary
- category
- trend theme
- relevance score

---

### Intelligence Outputs

The system generates structured outputs such as:

- weekly research reports
- trend alerts
- emerging startup signals
- categorized research archives

These insights can be delivered through tools such as Slack, Notion, or email.
