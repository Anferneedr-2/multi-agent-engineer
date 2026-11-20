# Tearing down a client team (retire, delete, hand the rest to agent-creator)

Use when the owner says 'start fresh', 'remove everything', or 'cleanup all' for a team that was built.

## 1. Confirm scope before anything runs
List three groups in plain words, then ask ONE question with numbered options (recommend 'everything, with a backup zip'; a clickable choice may fail to deliver, so repeat the options in text):
- What the engineer can remove with a yes: the team's helper profiles (`create_agent.py --retire`), its own planning files in Work/, the project folder contents, the manifest file it wrote.
- What it cannot remove: registry rows, the client entry in registry.yaml (never hand-edit; no script removes them), test reports and drafts in Agents/briefs/, generic helpers and metadata that agent-creator added, the vendor's files and code.
- What it will not touch: any other client or project.
Warn that a deletion is permanent and the evidence snapshot is the only copy of the test results; a 'no backup' answer is the owner's call, record it.

## 2. Order that keeps the registry consistent
1. Retire members one by one, reverse build order (conversation agent first, team lead last): `create_agent.py --retire <slug> --approved`. It deletes the profile (memory, session history, profile backups) and KEEPS the registry row and the brief path. Verify with `hermes profile list`, `registry.py status` (rows show `retired`) and `validate_registry.py` (expect 0 errors).
2. Delete the engineer's own files, but KEEP the member briefs that the retired registry rows point to, and keep the now-empty project folder (the rows' `folder` points there). Delete the manifest the engineer wrote, and its scratch files. Never delete the briefs or folder until agent-creator has removed the rows.
3. Hand the rest to agent-creator via a paste-ready message for Robi (the engineer may only send it a background request for a missing generic helper, never a deletion request): remove the retired rows and the client entry, the test reports and drafts, then tell Robi so the briefs can go; ask it to list which generics and which added metadata belong to this project and not remove any until Robi confirms each. Warn that generics may be marked production and reusable by other clients, so removing them is a separate decision.

## 3. When the destructive command is blocked
- A recursive delete may be refused with 'BLOCKED: Failed to send approval request' when the approval prompt cannot reach the owner (e.g. on Telegram). Nothing ran. Do NOT retry, split it into smaller deletes, or reword it to slip past the guard.
- Instead write a user-run cleanup script in `Work/` (absolute paths only; refuses to run if any of the team's profiles still exist; prints counts and the exact list; requires a typed confirm word such as CLEANUP; deletes only the listed paths and prints what is left), check it with `bash -n`, and give the owner Terminal steps in plain words. The script itself must live outside the folders it deletes.
- Do not put a CHANGELOG line in the same parallel batch as a command that may be blocked: the log line will assert work that did not happen. Log after verifying the real state, and if a wrong line was written, correct that line in place and say so.
