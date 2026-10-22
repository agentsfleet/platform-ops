# platform-ops

> [!IMPORTANT]
> **This repository is sunset and archived.**
> [`agentsfleet`](https://github.com/agentsfleet/agentsfleet) now allows a user
> to author their own fleets directly, so this standalone Fleet Bundle is no
> longer maintained. Use
> [github.com/agentsfleet/agentsfleet](https://github.com/agentsfleet/agentsfleet)
> instead. The content below is kept for reference only.

A prebuilt `agentsfleet` fleet that diagnoses production incidents. It reads Fly.io app state and logs, pulls Upstash Redis stats, correlates the two, and posts a plain-prose diagnosis to one Slack channel. It is strictly read-only against Fly and Upstash — it can list, read, and reason, but never deploy, scale, restart, or flush. Its single write path is the Slack post.

It wakes three ways: a GitHub Actions `workflow_run` failure on your production branch, a cron schedule you set at install time ("morning health check"), or a direct `agentsfleet steer`.

## The bundle

| File | Role |
|---|---|
| `SKILL.md` | The fleet's instructions — what it does, how it reasons, when it stops. |
| `TRIGGER.md` | The wake rules, tool allowlist, credential names, network allowlist, and spend caps. |

The frontmatter `name:` in both files is `platform-ops`, matching this repository. That identity is load-bearing: the importer rejects a bundle whose `SKILL.md` and `TRIGGER.md` names disagree, and the catalog entry id is taken from that name.

## Install

```bash
agentsfleet secret create fly     --data='{"host":"api.machines.dev","api_token":"<fly token>"}'
agentsfleet secret create upstash --data='{"host":"api.upstash.com","api_token":"<upstash token>"}'
agentsfleet secret create slack   --data='{"host":"slack.com","bot_token":"<slack bot token>"}'
agentsfleet secret create github  --data='{"webhook_secret":"<base64>","api_token":"<gh token>"}'

agentsfleet install --library platform-ops
agentsfleet steer <FLEET_ID> "morning health check"
```

Credentials resolve from your vault at the HTTPS boundary, after the sandbox has closed around the agent — the fleet only ever sees `${secrets.fly.api_token}`-style placeholders, never the raw bytes.

## Spend

Two independent hard caps, declared in `TRIGGER.md`: `$1.00` daily (a blast-radius guard) and `$8.00` monthly (the real envelope). The first to trip blocks further runs. A typical diagnosis costs well under a dollar.
