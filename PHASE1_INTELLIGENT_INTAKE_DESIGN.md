# Intelligent Intake, Phase 1 Design (Draft)

Case type: traffic tickets (default, matches Steven's own onboarding example and existing infrastructure). Not yet confirmed by Steven, revise if he picks differently.

Status: design only. No code written against the live database yet, blocked on repo access.

---

## 1. Staging table (candidate intake, modeled on existing epistemic pattern)

The `ocr` schema and `fsid_ai.system_expressions` already use a candidate → confidence → validated → promoted pattern. Intake staging should follow the same shape rather than invent a new one.

```sql
create table if not exists intake_staging (
    staging_id           bigint generated always as identity primary key,
    raw_input            text not null,              -- the client's original sentence
    input_source         text not null,               -- 'text' | 'voice_transcript'
    submitted_at          timestamptz not null default now(),

    -- candidate structured fields (mirrors ticket_docket_canonical's real columns)
    candidate_motorist_name     text,
    candidate_ticket_utt_no     text,
    candidate_hearing_office    text,
    candidate_vtl_section_code  text,
    candidate_violation_desc    text,
    candidate_incident_date     date,

    -- resolution
    candidate_fsid        text,          -- result of v_fsid_signal_graph lookup, null if no match
    is_new_matter          boolean,       -- true if no existing FSID matched

    -- epistemic fields (same shape as ocr.field_results)
    confidence            numeric,
    disposition            text not null default 'needs_review', -- needs_review | approved | rejected
    reviewed_by            text,
    reviewed_at            timestamptz
);
```

Nothing here is canonical. `ticket_docket_canonical` only gets written to after human approval and a real `fsid_mint()` call.

## 2. Extraction step

LLM call, structured/schema-constrained output (not free-text parsing), same field list as the staging table. Model proposes values and a confidence score per field. Never writes directly to `ticket_docket_canonical`.

Relative dates ("yesterday") resolved against submission timestamp before storage.

## 3. Sensitive-data routing (runs before anything else)

Any extracted value matching SSN, DOB, IP PIN, or medical detail patterns never enters `intake_staging`. Routes directly to `person_sensitive` (Tier 0). This check runs on the raw input text too, not just structured fields, since a client may mention sensitive details in a sentence that don't map to any of the fields above.

## 4. Validation rules (before a row can move to `approved`)

- `candidate_hearing_office` must match a known court from the existing TVB/court list (closed value set, not free text)
- `candidate_vtl_section_code` must match VTL code format
- `candidate_incident_date` must not be in the future
- `candidate_motorist_name` triggers a `v_fsid_signal_graph` lookup. Match found: sets `candidate_fsid`, `is_new_matter = false`. No match: `is_new_matter = true`, flagged for mint on approval.

## 5. Human review step

Reviewer sees staged row, candidate fields, confidence, and whether it resolves to an existing matter or a new one. Approve, edit, or reject. No UI decision made yet, Steven hasn't specified a preferred tool, placeholder for now.

## 6. On approval

- Existing matter: candidate fields written to `ticket_docket_canonical` against the resolved FSID.
- New matter: `fsid_mint()` called (never a direct insert), then fields written.

## 7. Registration (once real repo access exists)

`intake_staging` needs a `COMMENT`, an `artifact_register` row (what/modeled-on/purpose/problem-solved, modeled_on = `ocr.field_results` pattern), and RLS written before it takes its first real row, not inherited from the current broken default.

---

## Open items blocking real implementation

- Repo access (`safesq-registry`), needed before this becomes real, committed code
- Confirmation on case type from Steven
- Court/jurisdiction list for the closed value set in rule 4, need the real list, not guessed
- Review tool decision (placeholder for now)
