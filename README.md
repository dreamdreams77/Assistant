# Personal Assistant

A phone-friendly web app with a calendar, to-dos, voice notes, repeating reminders, alarms, a journal, an email queue, and an emergency panel with a medical ID. It installs to your home screen from GitHub Pages and shares its data with scheduled Claude tasks through a private GitHub repository.

## What is in this folder

| File | Purpose |
| --- | --- |
| `index.html` | The whole app |
| `manifest.webmanifest`, `icons/` | Lets phones install it to the home screen |
| `sw.js` | Keeps the app opening when you are offline |

## Put it on your phone

You need two repositories: a **public** one for the app and a **private** one for your data. Keep them separate. The app repository is public on a free GitHub plan, so your journal and contacts must never go in it.

1. Create a public repository, for example `assistant`, and upload every file in this folder to it.
2. In that repository open **Settings > Pages**, choose **Deploy from a branch**, pick `main` and `/ (root)`, and save. After a minute the app is live at `https://YOUR-USERNAME.github.io/assistant/`.
3. Open that address on your phone.
   - iPhone: Safari > Share > **Add to Home Screen**.
   - Android: Chrome > menu > **Install app** (or **Add to Home screen**).

## Connect your data and Claude

1. Create a **private** repository, for example `assistant-data`. Add a README so it has a `main` branch.
2. Create a fine-grained token at **GitHub > Settings > Developer settings > Fine-grained tokens**. Give it access to only `assistant-data`, with **Contents: Read and write**.
3. In the app tap the status chip in the top bar to open Settings. Enter your username, `assistant-data`, and the token, then tap **Save and sync**.

The app now saves to `assistant-data.json` in that repository. The token stays in that browser on that device. Repeat step 3 on each device you use.

## How scheduled Claude tasks fit in

A scheduled Claude task opens the private data repository, edits `assistant-data.json`, and commits. The app picks the change up within a minute while open, or the next time you open it. Changes merge item by item, so the app and a task can both write without losing each other's work.

Things a task can do:

- **Send queued email.** Emails you queue in the app sit in `outbox` with `status: "queued"`. A task sends each through Gmail and sets `status: "sent"` and `sentAt`.
- **Write briefings.** Items a task adds to `briefings` show on the Today screen under "From Claude".
- **Add events, reminders and alarms**, for example copied from Google Calendar.
- **Email you about due reminders**, since the app itself only alerts while it is open.

### Data format

```json
{
  "version": 2,
  "events":    [{ "id": "", "u": 0, "title": "", "date": "YYYY-MM-DD", "time": "HH:MM or empty", "repeat": "'' | daily | weekly | monthly | yearly", "notes": "" }],
  "reminders": [{ "id": "", "u": 0, "text": "", "due": 0, "done": false, "repeat": "'' | daily | weekly | monthly" }],
  "todos":     [{ "id": "", "u": 0, "text": "", "done": false, "star": false }],
  "alarms":    [{ "id": "", "u": 0, "time": "HH:MM", "label": "", "enabled": true, "days": [0, 6] }],
  "journal":   [{ "id": "", "u": 0, "date": "YYYY-MM-DD", "mood": "", "text": "", "created": 0 }],
  "voice":     [{ "id": "", "u": 0, "title": "", "note": "", "created": 0, "dur": 0, "mime": "audio/webm", "size": 0, "up": 1 }],
  "contacts":  [{ "id": "", "u": 0, "name": "", "phone": "", "email": "", "ice": false }],
  "outbox":    [{ "id": "", "u": 0, "to": "", "subject": "", "body": "", "status": "queued | sent | failed", "created": 0, "sentAt": 0, "error": "" }],
  "briefings": [{ "id": "", "u": 0, "title": "", "body": "", "created": 0 }],
  "deleted":   { "item-id": 0 },
  "prefs":     { "emergencyNote": "", "medical": { "name": "", "blood": "", "allergies": "", "meds": "", "conditions": "", "doctor": "", "share": false }, "u": 0 }
}
```

Rules for anything that writes this file:

- All times (`u`, `due`, `created`, `sentAt`) are milliseconds since 1970 UTC.
- Give every new item a unique `id`.
- An event with `repeat` set shows on its `date` and on every matching day after it. Store it once; do not add one item per occurrence.
- Alarm `days` uses 0 for Sunday through 6 for Saturday. An empty list means every day.
- `prefs` is replaced as a whole, so copy the existing fields when changing one.
- Set `u` to the current time whenever you add or change an item. The newest `u` wins a merge.
- To delete an item, remove it from its list **and** add `"its-id": <now>` to `deleted`. Otherwise another device puts it back.
- Leave items you are not changing exactly as they are.

### Voice notes

Recordings are saved on the device and copied to the data repository as `voice/<id>.webm` (or `.m4a`). `up: 1` means the copy exists. Recordings made on Android may not play on an iPhone and the other way round.

## The sky panel

The panel on Today follows real sunrise and sunset: its colour, the sun or moon on the arc, the moon's phase and the stars. By default the times are estimated from the device's time zone. **Settings > Appearance > Use my location** makes them exact; the location is rounded and kept on that device only, never synced. The same card can tint the whole app with the sky and switch light and dark with the sun.

## Limits

- Reminders and alarms in the app alert only while the app is open. Phones do not let a web app wake itself.
- The emergency panel opens your phone's call, text and email apps. It cannot place a call or send a message on its own.
- The app does not send email. Queued email is sent by a Claude task; "Open in email app" hands the draft to your mail app.

## Backups

Settings has **Download backup** and **Restore from a backup file**. Restoring merges the file with what is already in the app.
