# Supabase Schema

## Overview

This project uses Supabase as the structured intelligence layer for storing market signals, extracted insights, and generated research outputs.

The schema is designed to support a lightweight but extensible research pipeline that can ingest raw signals, enrich them with AI analysis, and make them queryable for downstream reporting.

---

## Table 1: `market_signals`

This table stores individual raw or processed signals collected from external sources.

### Purpose

Track each market signal as a structured record that can be analyzed, categorized, and reused in reports.

### Fields

| Field | Type | Description |
|---|---|---|
| id | uuid | Primary key |
| created_at | timestamp | Record creation timestamp |
| source_name | text | Name of source site, feed, or platform |
| source_type | text | Source category such as news, launch, research, social |
| source_url | text | Original URL of the signal |
| signal_title | text | Title or headline of the signal |
| raw_content | text | Raw text collected from the source |
| signal_date | date | Date the signal was published |
| processing_status | text | Status such as pending, processed, failed |

---

## Table 2: `signal_insights`

This table stores AI-processed outputs derived from individual market signals.

### Purpose

Convert unstructured signals into structured intelligence that can be searched, filtered, and used in reports.

### Fields

| Field | Type | Description |
|---|---|---|
| id | uuid | Primary key |
| created_at | timestamp | Record creation timestamp |
| market_signal_id | uuid | Foreign key linked to market_signals.id |
| summary | text | AI-generated summary of the signal |
| category | text | Signal category such as product launch, funding, tooling, research |
| company_name | text | Primary company or entity mentioned |
| trend_theme | text | Higher-level trend label |
| signal_type | text | Type of event or update |
| relevance_score | numeric | Relative importance score |
| sentiment | text | Optional sentiment or directional signal |
| key_entities | jsonb | Extracted entities such as products, founders, or technologies |

---

## Table 3: `research_reports`

This table stores compiled reports generated from multiple signals.

### Purpose

Aggregate multiple insights into digestible research outputs such as weekly reports or thematic briefs.

### Fields

| Field | Type | Description |
|---|---|---|
| id | uuid | Primary key |
| created_at | timestamp | Report generation timestamp |
| report_title | text | Name of report |
| report_type | text | Example: weekly brief, trend alert, sector summary |
| report_summary | text | AI-generated high-level summary |
| key_trends | jsonb | List of trends identified in the report |
| signal_count | integer | Number of signals included |
| date_range_start | date | Beginning of reporting window |
| date_range_end | date | End of reporting window |

---

## Table Relationships

### `market_signals` → `signal_insights`

Each raw signal can have one associated processed insight record.

### `signal_insights` → `research_reports`

Multiple processed insights can contribute to a single research report.

For an MVP, this relationship can be documented conceptually even if implemented manually.

---

## Example Query Use Cases

This schema supports queries such as:

- Which companies appeared most often this week
- Which trend themes are increasing in frequency
- What product launches were categorized as high relevance
- Which signals contributed to the latest weekly report

---

## Why Supabase

Supabase was selected for this project because it provides:

- structured relational storage
- query flexibility
- easy integration with automation workflows
- scalability beyond lightweight spreadsheet-based tracking

This makes it a strong fit for AI workflows that need durable, queryable intelligence records.

---

## MVP Scope

For the initial version of this project, the core implementation can focus on:

- `market_signals`
- `signal_insights`

The `research_reports` table can be added once recurring summary generation is implemented.
