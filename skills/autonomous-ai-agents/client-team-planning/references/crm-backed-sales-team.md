# Pattern: agent team over an external system of record (e.g. SDR team on a CRM)

Use when the team talks to outside people and a CRM/app already owns the data.

## Source of truth
- The external system owns records, knowledge, campaign config, schedules, permissions, appointments. Hermes owns execution only and keeps no independent records.
- Working files carry the external IDs and the knowledge version used. Memory-manager stores pointers to approved knowledge versions, not copies.

## Roles that worked as a split (by tool permission and checker)
- Lead (copy of team-lead): task files, handover log, escalation routing, readiness gate.
- Researcher (missing generic): research tools, read-only, sourced brief, no sending.
- Qualifier (modified copy of sales-agent): decision from the researcher's brief; no outside tools.
- Conversation agent (missing generic): the only member with send and calendar; one owner per conversation.
- Reviewer (modified copy): independent PASS/RETURNED, grounding against approved knowledge, scores simulations; never rewrites; no send.
- Trainer (missing generic): training drafts and simulated prospect personas; separate from the agent it tests; no outbound.
- Memory-manager and document-engineer (copies); document-engineer also drafts reports from the system's exports.

## Phase 1 copy-brief changes (files only, nothing connected)
Keep the copies near-generic; add only what makes the team RIBO-ready without a connection:
- Team-lead copy: task files add Tenant, external ID(s), Knowledge version, Autonomy level (default L0), Environment (none or sandbox), Idempotency key (task number + step); write 'none yet' for IDs/versions, never invent them. Hand tasks only to BUILT members (planned ones are named in the team map as not built). Escalation files in Work/escalations/ESC-<n>-<name>.md that stop the task and go to the human. Readiness checklist in Work/readiness.md; only Robi confirms items. The lead cannot start members (no terminal/delegation): Robi or a later run_team.py does.
- Memory-manager copy: sole writer of Memory/; keeps pointers to external records (ID, knowledge version, date), never copies of records.
- Document-engineer copy: report drafts only on request, only from files it is pointed to, every line naming its source file; read-only on Memory/.
- Copy tests use made-up data only, plus tool-boundary tests ("fetch from RIBO" -> says it has no such tool) and a not-built-member test for the lead.
- Every config gets standard fallback model providers from create_agent.py; mention this to Robi once because prospect data comes later.

## Phase 2 copy-brief changes (remaining roles, sandbox, still no tools)
- Qualifier copy of sales-agent: keep Qualify Lead layout, grades A-D, buying signals, follow-up recommendation, authority and source-of-truth rules; add Appointment Ready (Yes/No/Needs Clarification) and Escalate to Human (Yes/No + reason); write `Work/qualification/<company>-qualification-v<n>.md` with the RIBO-ready header copied from the task; drop quotation, coaching, pipeline, forecast, delegation and the customer-facing mode.
- Reviewer copy: reads any file the task names under Work/; adds grounding (claims must be in the named knowledge), escalation (sensitive topics must escalate, not answer), autonomy-level, RIBO-header (no invented IDs) and rubric-scoring checks, each only when the task supplies the needed input; never declares readiness.
- Conversation agent in Phase 2 only writes SIMULATED replies (no send tool); Robi pastes turns between the trainer persona and the conversation agent until a sandbox exists.
- Order: researcher, qualifier, reviewer, trainer, conversation agent last. Roles still pending in agent-creator must also be added to GENERIC-HELPERS.md (Robi's call, via agent-creator) before they can be copied.
- Scenario runs write real SYNTHETIC files into the project (qualification, escalation, report, readiness, tasks); list them for Robi instead of deleting by hand.

## Asking the CRM vendor for its API (Phase 3 input)
Write one paste-ready request in Work/<Project>/ (never send it yourself; Robi forwards it) and tell the vendor not to put secrets in chat. Cover: a sandbox for the tenant with made-up data that cannot reach real people; token auth scoped to tenant AND role (research, qualification, conversation, review, training, reporting) with rate limits and audit log; read endpoints (campaigns with autonomy level and who set it, prospects with opt-out and time zone, conversations, approved knowledge with version IDs, training config, appointment slots, permissions); write endpoints (research note, qualification result, message draft with status, escalation case, opt-out, appointment proposal and handoff packet, training drafts and simulation records; readiness sign-off read-only); idempotency keys, work-item claim with expiry, queue or signed webhooks, heartbeat, behaviour when Hermes is silent, restart reconciliation, error codes and versioning; controls the vendor must enforce itself (opt-outs, send caps, quiet hours, blocked topics; only a person changes autonomy level, approves knowledge or training, signs off readiness); whether the vendor sends email and books the calendar so the team never holds mailbox credentials; escalation and handover to the human rep; data-protection fields and consent; limits and costs. Deliverables: OpenAPI or equivalent, an end-to-end example work item, a one-page permission matrix, sandbox steps, a test checklist, a list of what is not supported. When the answer arrives, check it against this list and name gaps before planning the first read-only connection.

## Controlled autonomy
L0 simulation only; L1 human-approved drafts; L2 autonomous inside the approved playbook with sampled review. Levels are set in the external system by a person; the conversation brief treats the level as an input it obeys. Fixed non-AI rules override everything: opt-outs, send caps, blocked topics.
Escalate to a human: pricing/discounts, legal/contract, complaints, enterprise/government, request for a human, negative sentiment, low knowledge confidence, unsupported language or request.

## Isolation
Put a tenant ID in every task file and use a tenant-scoped token even for a single tenant (cheap, avoids rework). Keep sandbox and production apart until a pilot is approved. On this Mac isolation is behavioural, not enforced; real enforcement is the external system's token scope.

## Offline recovery
The external system holds queues and state. Hermes writes with idempotency keys and short task claims and reconciles from the system on restart. If Hermes goes silent the system pauses sequences, sends a fixed pre-approved holding reply and routes inbound to humans.

## Phases
0 paperwork; 1 default members (files only); 2 remaining members on a sandbox, no tools; 3 connect tools one by one, read-only first, each with its own OK; 4 training, simulations, reviewer scoring, human sign-off, shadow at L1; 5 pilot with caps, scheduled execution (separate timer OK), reporting, offline drill; 6 promote members one at a time after a correct real result.
