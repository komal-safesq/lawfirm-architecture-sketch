# Steven's Core Vision — and What's Actually Verified

Distilled from ~3 hours of onboarding/meeting transcripts (2026-07-11 through 2026-07-14), paired with what's been directly confirmed against the live Supabase project and GitHub repos as of 2026-07-15.

---

## Mission

Build an AI-powered knowledge platform that captures information once, stores it in structured databases, connects related information across multiple systems, and allows AI to reason over trusted data instead of isolated documents.

## The Problem

Today's information is fragmented: SharePoint stores documents, Outlook stores emails, WhatsApp stores conversations, Calendar stores meetings, Supabase stores structured records, OCR extracts text, and AI knows nothing unless the information is connected. The goal is to remove these silos.

## Design Philosophy

```text
Real World
  ↓
Canonical Source
  ↓
Structured Database
  ↓
Relationships
  ↓
AI
  ↓
Knowledge
```

AI is not the database. AI sits on top of the database.

## Core Principles

1. **Canonical Data** — use the official source whenever possible (DMV license, government forms, medical coding, standard identifiers). Do not invent your own version of existing information.
2. **Capture Information Once** — scan license → create person → scan ticket → populate database → generate calendar/deadlines/documents. Everything else reuses the same data.
3. **Structured Before AI** — schemas, validation, relationships, databases before AI. The database is the source of truth.
4. **Relationships** — the database should understand relationships between clients, matters, tickets, emails, hearings, documents, medical records, financial records. Everything should connect.
5. **Standards** — reuse government, legal, medical, and industry standards whenever possible, rather than inventing new ones.
6. **AI on top of structure** — AI should interpret structured data, not compensate for poor data organization.

## System Architecture (as envisioned)

```text
Client → Intake → OCR → Supabase → AI Layer → SharePoint / Outlook / Calendar / Planner / Reports
```

Supabase is meant to become the central knowledge repository.

## Major Projects

- **Intelligent Intake** — natural language and scanned documents into structured records
- **Smart OCR** — document type + expected fields + validation rules + missing-info detection, not just text extraction
- **Unified Knowledge Base** — emails, documents, calendar, SharePoint, OCR, databases into one connected system
- **AI Search** — "tell me everything about this client," AI gathers from the connected database
- **Time Intelligence** — every minute of work belongs to a client/matter/project/task, not just "Chrome was open"

## Learning Roadmap (priority order)

SQL → PostgreSQL → Supabase → Database design → APIs → Git/GitHub → Docker → OCR → RAG → Microsoft 365 (SharePoint, Teams, OneDrive)

## The one-sentence version

> Build a canonical, structured knowledge system first; use AI to reason over that knowledge rather than replacing it.

---

# Verified Reality Check (2026-07-15)

Every row below is confirmed directly against the live database, GitHub repos, or Steven's own written documents, not inferred from the vision alone.

| Vision component | What's actually verified |
|---|---|
| **Capture Once** (scan → create person → ticket → calendar → deadlines → documents) | This automated chain does not exist as one pipeline yet. Pieces exist (`ticket_docket_canonical`, `case_calendar_event`), but real records have gaps: unvalidated ticket IDs, missing companion tickets, years of historical tickets never migrated (confirmed via the Florans and Fasten examples). |
| **Structured Before AI** | True as doctrine, an explicit, enforced hard rule, not just philosophy. But one structural safeguard is currently broken: RLS is ineffective on 123+ tables firm-wide (`USING(true)` policies, 166 tables open to `anon`). |
| **Relationships** | The identity resolver is real and working (`v_fsid_signal_graph`, ~4,950 signals). But the deeper knowledge-graph layer (Apache AGE, described in the LPIS/time-reconstruction document) is installed and empty, "nothing is writing edges" in Steven's own words. |
| **AI Search** ("tell me everything about this client") | Actively being prototyped via Perplexity connected to multiple apps. Currently unreliable: it hallucinated an entire non-existent Supabase schema (`court_matters`, `matter_documents`, `people`, none of which exist) rather than reading the real one. |
| **Time Intelligence** | Fully designed (sessionize → coalesce → cluster → narrate pipeline), but exists "as prompts, not as code" per Steven's own document. Nothing automated is running yet. |
| **Intelligent Intake** | No dedicated build exists yet. The "intake gate" mentioned in doctrine (830000-series matter numbering) was searched exhaustively, function bodies, table/column names, object comments, and found nowhere in the database. Either external or unbuilt. |
| **The three repos** (`safesq-registry`, `safesq-paperless`, `safesq-fsid-knowledge`) | Real and exist on GitHub, created 2026-07-13. `safesq-registry`'s first backlog item is exporting 149 migrations that currently exist only in the live database. |
| **Semantic/AI search over facts** | Fully dead: 0 successful runs ever (1,723+ failed cron attempts as of last check). Do not build on `match_facts_semantic()`. |

## The sixth operational principle, learned the hard way

Not in the original five, but arguably the most consequential day-to-day: **audit before building.** Query `artifact_register` before creating anything new. As of the last audit, 125 artifacts in this database are built-but-unwired or outright duplicates, created by agents (human or AI) who didn't check first. This isn't philosophy, it's a repeatedly documented, costly failure mode in this specific system.
