# SafeSQ Invoice Generation Guidelines

## Purpose

This document defines how a weekly invoice/timesheet is presented for employer review. It governs **formatting and presentation only**. The underlying classification of work (Domain, Phase, Activity, narrative style) is defined in `SAFE_SQ_WORKLOG_GUIDELINES.md` and should be treated as the source of truth, this file does not redefine it.

## Invoice Structure

Each invoice entry should contain the following columns.

| Column | Description |
|---------|-------------|
| Date | Date work was performed |
| Matter / Project | High-level project or matter |
| Domain | Engineering domain (see work log guidelines) |
| Phase | Stage of work (see work log guidelines) |
| Activity | Primary activity performed |
| Hours | Hours worked |
| Work Performed | Concise professional narrative |

**Example:**

| Date | Matter / Project | Domain | Phase | Activity | Hours | Work Performed |
|------|-------------------|--------|--------|----------|------:|----------------|
| July 9 | AI Legal Platform | ARCH | Analyze | System Architecture | 8.0 | Reviewed system architecture, mapped database relationships, documented AI workflow and integrations. |

## Style

Invoices should read like professional engineering consulting reports.

**Avoid:**
- Vague descriptions
- Conversational language
- Unnecessary marketing language

**Prefer:**
- architecture, schema, integration, implementation, workflow, validation, prototype, automation, synchronization, analysis

## Workflow: "Generate my invoice"

When asked to generate an invoice:

1. Read the work log (`SAFE_SQ_WORKLOG_GUIDELINES.md` classifications applied to the actual hours/tasks for the period).
2. Organize work into projects/matters.
3. Assign the most appropriate Domain, Phase, and Activity per entry.
4. Write concise engineering narratives per the work log's narrative guidelines.
5. Ensure hours are correct and sum exactly to the reporting total, never invented.
6. Produce a clean, professional invoice table suitable for employer review.

This document is the authoritative formatting guide for invoices unless superseded by future instructions from Steven.
