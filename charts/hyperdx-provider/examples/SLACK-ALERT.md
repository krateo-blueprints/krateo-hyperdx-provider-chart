# Declarative pod-restart → Slack alert (via the HyperDX operator)

This replaces `obs-stack/pod-restart-alert/*.sh` with CRs reconciled by the
`hyperdx-{webhook,alert}-controller`. Apply `pod-restart-alert.yaml` after filling in
the inputs below.

## What it creates
- `WebhookConfiguration` / `AlertConfiguration` — carry the HyperDX Bearer API key.
- `Webhook krateo-troubleshooting` — the Slack incoming-webhook target (HyperDX assigns
  its id into `status.metadata._id`).
- `Alert pod-restart-alert` — fires when Warning pod events in 5m exceed the threshold,
  grouped per pod, notifying the Webhook via `channel.webhookId`.

## Inputs YOU must supply (cannot be generated)
| Placeholder | Where to get it |
| --- | --- |
| `hyperdx-api-key` Secret `token` | HyperDX UI → Settings → API Keys (the HyperDX LB is `krateo-clickstack-app-lb`, port 3000) |
| `Webhook.spec.url` | A Slack **Incoming Webhook** URL (Slack app → Incoming Webhooks) |
| `Alert.spec.savedSearchId` | A HyperDX **source** id — ClickStack 3.0 attaches alerts to a source (`GET /api/sources`), not the old `/saved-searches` |
| `<@KAGENT_BOT>` in the message | the Slack member id of the kagent bot (if mentioning it) |

## Remaining external dependency for the FULL closed loop
The Slack message → agent hop still needs the **a2a-slack-bot** (Socket Mode → A2A),
which is not in any repo. Until it's deployed, the alert reaches Slack but does not
auto-trigger the autopilot. Two ways to close it:
1. Deploy the a2a-slack-bot (external) and `@mention` it in `Alert.spec.message`.
2. Point the HyperDX webhook at a small in-cluster proxy that calls the autopilot A2A
   endpoint (`http://krateo-autopilot.krateo-system:8080/`) — the "autopilot-alert-proxy"
   noted as planned in obs-stack `docs/IMPROVEMENT_PLAN.md`.

With `installer --set hitlApproval=false` the autopilot then remediates with **no manual
approval** once triggered (verified against the OOMKill demo).
