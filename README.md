# antfarm-telemetry

Append-only telemetry for [A.N.T.F.A.R.M.](https://github.com/project-antfarm)
colonies. Written by deterministic workflow steps through a dedicated GitHub
App. Agents never hold a credential for this repository.

## Layout

```
runs/<specimen>/<run>/manifest.json      experimental configuration of the run
runs/<specimen>/<run>/events/*.jsonl     one immutable file per execution
state/<specimen>.json                    display-only projection (see below)
```

Event files are named `<UTC timestamp>_<role>_<execution id>.jsonl`. Files are
never edited after they are written. To reconstruct a run, read every file
under `events/` and sort by `ts`, then `event_id`.

## What is recorded here, and what is not

Recorded here: what GitHub does not know. Executions, model, turns, tokens,
cost estimate, duration, guard and budget blocks, the agent's own report of
what it did and why, human notes.

Not recorded here: Issues, pull requests, reviews, CI results, merges and
commits. GitHub already keeps those with timestamps and authors. An exporter
joins both sources by Issue, PR, SHA and workflow run id.

## Facts and interpretation

Every field comes from the infrastructure except `agent_report`, which is the
agent's own account. Treat the two as different kinds of evidence.

## Event schema (version 1)

```json
{
  "schema_version": 1,
  "event_id": "worker-35131491949-1-2",
  "ts": "2026-09-17T12:19:52Z",
  "run": "run-001",
  "specimen": "antfarm-daily",
  "execution_id": "worker-35131491949-1",
  "actor": { "type": "worker", "model": "claude-sonnet-5", "app": "antfarm-worker" },
  "type": "worker.completed",
  "trigger": "issues",
  "target": { "issue": 7, "attempt": 0 },
  "usage": { "turns": 13, "duration_ms": 26559, "cost_usd": 0.24,
             "input_tokens": 20, "output_tokens": 1934,
             "cache_read_tokens": 320824, "cache_create_tokens": 38082 },
  "agent_report": { "summary": "...", "pr": 9 },
  "workflow_run": "https://github.com/<owner>/<repo>/actions/runs/<id>"
}
```

Event types: `queen.started`, `queen.completed`, `queen.failed`,
`worker.started`, `worker.completed`, `worker.failed`, `guard.blocked`,
`human.note`. New types and fields may be added; existing ones do not change
meaning. Fields that do not apply are omitted.

## state/<specimen>.json

A convenience for dashboards: runs today, tokens this week, last event. It is
recomputed from the event files on every write, never incremented, and never
read by the budget guards. If it is wrong or missing, delete it; the next
execution rebuilds it.
