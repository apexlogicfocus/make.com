# Getting started

Zero to your first running Make.com scenario in ~10 minutes.

---

## 0. Prerequisites

- A [Make.com](https://www.make.com) account (the free tier is enough to import and test).
- Accounts for whatever apps a blueprint touches (e.g. a CRM, Slack, Google Sheets). Each blueprint's README lists exactly which.

## 1. Pick a blueprint

Browse the [library map](../README.md#-library-map). Each category folder has scenarios with their own README. If you're new, start with [`Utilities/daily-ops-digest`](../Utilities/daily-ops-digest/) — it only needs a Slack connection and a schedule.

## 2. Import the blueprint

1. In Make, click **Create a new scenario**.
2. Click the **⋯** (more) button in the bottom toolbar → **Import Blueprint**.
3. Upload the `blueprint.json` file from the blueprint's folder.

> 📷 *Screenshot placeholder:* `docs/screenshots/import-blueprint.png` — the Import Blueprint dialog. (Add your own once you import — see [the screenshots guide](screenshots/README.md).)

The scenario appears on the canvas with all modules and routing intact.

## 3. Reconnect the apps

Blueprints ship **without credentials**. Each app module shows a "reconnect" prompt:

1. Click each module flagged with a warning.
2. Choose **Add** under *Connection* and authenticate your own account.
3. Repeat for every flagged module (usually: the data source, the action app, and the notifier).

## 4. Set config values

Open the blueprint's `README.md` and fill in the **Setup** table — things like which Slack channel, which CRM pipeline id, or which spreadsheet. These map to module fields or scenario variables.

## 5. Test with sample data

- For **webhook** triggers: copy a payload from [`examples/`](../examples/) and POST it to your webhook URL (the README gives a `curl` example).
- For **scheduled** triggers: click **Run once** to fire immediately.

Confirm two things happened:
- ✅ a row landed in your **log** sink (Data Store or Google Sheet), and
- ✅ you received the expected **notification**.

## 6. Go live

Once a manual run is clean, flip the scenario's **scheduling** toggle on. Set the interval per the README (instant for webhooks, or e.g. every 15 min / daily for polled triggers).

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Module shows red "connection" error | App not reconnected after import | Click the module → add your connection |
| Scenario halts mid-run | A fragile module missing its handler, or a real upstream outage | Check the error log row; retries should cover transient cases |
| No log row | Log sink not connected / wrong Data Store name | Re-point the final module to your Data Store or Sheet |
| No Slack message | Wrong channel id or bot not in channel | Invite the bot; verify the channel in the Setup table |

Deeper failure-mode guidance: [conventions.md → Retries](conventions.md#2-retries-self-healing-on-transient-failure).
