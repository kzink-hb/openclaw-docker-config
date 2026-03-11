# Cron Job Templates

Template cron jobs for OpenClaw. Install via the CLI after deployment:

```bash
# SSH to VPS
make ssh

# Add a cron job from JSON
oc cron add < ~/.openclaw/workspace/cron/daily-summary.json
oc cron add < ~/.openclaw/workspace/cron/weekly-summary.json
```

Or via `make ssh` and paste the `openclaw cron add` commands from each file's comments.

## Available Templates

| Template | Schedule | Description |
|----------|----------|-------------|
| `daily-summary.json` | 2:00 AM daily | Summarize raw daily notes into structured summaries, clean up old files |
| `weekly-summary.json` | 9:00 AM Fridays | Roll up daily summaries into a weekly overview, memory maintenance |

## Customization

- **Timezone:** Update `schedule.tz` to match your timezone
- **Delivery:** Set `delivery.mode` to `"channel"` and `delivery.channel` to your channel (e.g., `"telegram"`) to receive summaries
- **Paths:** Templates use `~/workspace/memory/` and `~/workspace/summaries/` — adjust if your workspace layout differs
