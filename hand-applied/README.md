# The Hands-On Mode help desk's enforced tool list (set by hand on 20 Nov 2026)

The agent `handsonmode-help` is offered and may call exactly eight tools:

- `read`, fenced to its own workspace folder (it reads its skill file with it);
- `governed__seat_seat_progress`, `governed__seat_seat_overview`, `governed__seat_seat_events`,
  `governed__seat_seat_logs`, `governed__seat_seat_gitea`, `governed__seat_seat_oc`,
  `governed__seat_workshop_guide`.

Before this, every new conversation was offered 224 tools: OpenClaw's built-ins (a shell and file
editing inside the gateway pod, web fetch, browser, sub-agents) and every tool on the platform's tool
gateway. `spec.tools.allow` in `base/agentworkstation.yaml` never was a limit: the operator reads it
as a list of things to switch on.

## Why this is in a folder called hand-applied

Operator v1.7.85 has no field for an enforced list, so the setting lives only in the gateway's
`openclaw.json` on its volume (`agents.list[handsonmode-help].tools`). The operator's merge keeps keys it
does not render, so it survives reconciles. It does NOT survive a rebuilt volume or a removed and
re-created agent entry: then all tools are back, silently. Check after either:

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- openclaw config get agents.list
```

The declared version is `spec.tools.restrictTo`, planned for operator v1.7.86. When that field is in
`base/agentworkstation.yaml` and rendered, delete this folder.

## Apply it again

Away from the minutes when the operator rewrites the gateway's settings (look at the times of
`~/.openclaw/openclaw.json.bak*` in the pod):

```bash
oc exec -i deploy/handsonmode-help -n agent-office -c openclaw -- node - handsonmode-help read,governed__seat_seat_progress,governed__seat_seat_overview,governed__seat_seat_events,governed__seat_seat_logs,governed__seat_seat_gitea,governed__seat_seat_oc,governed__seat_workshop_guide < set-tool-list.js
```

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- openclaw config patch --file /tmp/zz-toollist-handsonmode-help.patch.json --dry-run
```

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- openclaw config patch --file /tmp/zz-toollist-handsonmode-help.patch.json
```

It applies without a restart.

## Check it

Start a test conversation and read what the gateway recorded for it (`toolCount` must be 8):

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- openclaw agent --agent handsonmode-help --session-id zz-toollimit-check --timeout 150 --json -m "Platform check, no attendee is waiting. Call the workshop guide tool for module 1 and give me its title in one line."
```

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- grep -o '"toolCount":[0-9]*' /home/node/.openclaw/agents/handsonmode-help/sessions/zz-toollimit-check.trajectory.jsonl
```

The session's `*.trajectory.jsonl` cuts every list at 64 entries; `toolCount` in its `session.started`
line is the true number.

## Remove it

```bash
oc exec -i deploy/handsonmode-help -n agent-office -c openclaw -- node - handsonmode-help - < set-tool-list.js
```

```bash
oc exec deploy/handsonmode-help -n agent-office -c openclaw -- openclaw config patch --file /tmp/zz-toollist-handsonmode-help.patch.json
```
