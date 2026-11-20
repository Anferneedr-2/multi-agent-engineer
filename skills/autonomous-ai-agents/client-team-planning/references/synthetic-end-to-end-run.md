# Synthetic end-to-end run (exit test for a built team, made-up data only)

Use after all members pass check_agent --scenarios and before moving the manifest forward. It proves the members work TOGETHER; per-member checks do not.

## Who runs it
The engineer never runs or monitors the team and may call only agent-creator, even when Robi says "I give you the authority". Do not argue the rule twice: write a script, syntax-check it with `bash -n` (never execute it), hand it over, give a newbie step sheet, and read the files afterwards. Plan first (fixtures, steps, pass test, minimum vs full run) and get a yes to the plan before writing the script.

## Script design (bash, started by Robi)
- Confirmation prompt (`Type RUN`), `set -u`, one `call <member> <message>` helper that runs `hermes -p <slug> --in <project folder> -z "<msg>"`, writes each answer to a numbered log file under Work/sandbox/run-log-<stamp>/, stops on a non-zero exit, prints `STOPPED:` with the step.
- Relay between a trainer persona and the conversation agent: ask the member to print the message between `<<<MSG` and `MSG>>>` lines and extract it with sed. Stop if the block is missing.
- Messages to members say "do the open task in Work/tasks/ that names you as owner" instead of hard-coding task numbers (the lead chooses names).
- Human gates stay human: when the lead escalates (e.g. low confidence), the script pauses with `Type GO` and passes Robi's decision to the lead in the next message; it never decides for him. Before a re-run, offer an `ARCHIVE` prompt that MOVES the previous run's Work/ files into Work/_archive-run<n>/ (task numbers and versions would clash); it deletes nothing and leaves Memory/ alone.
- Newbie step sheet: open Terminal, paste one command, type RUN, expected output, how long (tens of minutes), Ctrl-C to stop, what to send back if `STOPPED:` appears, then "run done".

## Fixture and script pitfalls (each cost a failed pass-test line)
- Every relayed message must carry the full six-field task header values the reviewer will check (Tenant, RIBO ID(s), Knowledge version, Autonomy level, Environment, Idempotency key). Members cannot see the lead's task file, so omitted fields end up as "not given" and the reviewer rightly scores the header check NOT MET.
- Any text a rubric calls "exact" (e.g. a holding line) must be written in a fixture file the reviewer can read. Wording that exists only inside a member's brief cannot be verified and gets flagged as an unsourced promise.
- When Robi says "you decide" on wording a rubric calls exact (holding line), keep the wording already in the approved generic brief, put it verbatim in the sandbox playbook and rubric, and say plainly it is sandbox-only: the real wording belongs in the external system and a person approves it there.
- Capture the last reply of every simulation turn; a transcript that ends before the helper's final reply leaves that turn unscored.
- Pass the date and time in the messages; the lead has no clock, so the handover log shows blank times otherwise.
- Cover every rubric line: include a price question (exact quote) and, in the full run, the legal persona; the minimum run leaves both untested and the report must say so.
- A hold written by the lead (e.g. escalation on low confidence) is a good result; a script that runs the next step anyway overrides it, so say so in the report.
- A member that asks a question instead of writing breaks the relay (no `<<<MSG` block, the script stops). The usual cause is a missing fixture value, so give every value a member's brief says to ask about: the prospect's time zone, today's date, the conversation owner, the playbook and the times file. Re-running the same script after a stop repeats the stop; fix the fixture first.
- Give outreach and each simulation its own conversation ID (e.g. C-O1 vs C-S1); sharing one ID makes the agent see an earlier message file and the trainer refuse to treat it as the reply.
- Compute 'today' inside the call helper on every call (substitute a placeholder per message), because a 30-call run can cross midnight and members then flag the date mismatch and stop to ask.
- The task limits the lead writes must list every approved source the member legitimately needs (knowledge file, playbook, times file). A lead that writes 'use only the knowledge' makes the reviewer return an outreach or training draft that correctly used the playbook or the times.
- Tell the lead which tasks finished (a message after each member step, or one batch message), or the handover log lacks return rows for them. Ask for a return row per returned file with the real file name.
- Gates that must always stop (discount, legal, opt-out) belong to fixed rules in the brief or playbook. The qualifier's low-confidence escalation flipped between Yes and No on the same facts across runs, so never base a pass-test line on a judgment call being repeated.
- End the script's closing line with the run's own label (generate it from the script name) so the user does not report the wrong run number.

## Confirm the run really finished before assessing it
A user may report 'run done' after a stop. Check, before reading any content: the number of files in the run-log folder equals the script's call count (the script prints it), no `STOPPED` text in the terminal paste, and the last log is the document-engineer's. If not, say plainly that the run did not finish, list what the completed steps showed, fix the cause in a new script version (keep the old one), and give the run command again. Keep earlier script versions for the record.

## Reading the result
Spot-check the reviewer's PASS lines against the raw files (the message body quotes the knowledge price exactly, the holding line matches the playbook word for word, the legal case has no prospect-facing message file, nothing is written after the handoff packet); a reviewer score is not the evidence. Read, in this order: handover log, task header, research, qualification, escalation files, each message/handoff, transcripts, reviews, readiness checklist, report draft, Memory/ and CHANGELOG tails. Report against each pass-test line as met, partly met or not tested, separate agent faults from script/fixture faults, and list proposed fixes (each needs its own yes and `--update` flow). Do not clean up or promote; ask.

## Design gaps this run exposed (check for them when building the next team)
- The lead invented output file names and lengths ("ASSUMED") that members' own briefs overrode; the lead's Output field should name each owner's standard output path.
- Only the lead's brief said to read Team/SHARED-BRIEF.md; every member copy needs that instruction.
- Header fields differed between copies (four vs six); use one six-field header definition in the shared brief and in every copy.
- One member once reported it could not find `Team/SHARED-BRIEF.md` while the others found it; write the shared-brief and team-map paths into every copy as full absolute paths.
- The generic conversation-agent brief says not to write when the prospect's time zone is missing, even for a reply that proposes no times; a scenario can then flip between pass and REVIEW. Treat a REVIEW on that as a generic wording gap (needs a new generic version through agent-creator), not as an update regression.
- The lead must record return rows in the handover log for every returned file, including simulations without a task file.
