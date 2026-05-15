# WatchTower&Tester App Local Setup Guide

Use this guide to run the WatchTower backend server, dashboard, and collector tester locally.

## 1. Get The Latest Project Code

If you do not have the project yet, clone it from GitHub:

```bash
git clone <repo-url>
cd WatchTower
```

If you already cloned the project, update your local copy:

```bash
cd WatchTower
git fetch
git pull
```

## 2. Add The Backend Environment File

Download the environment file from the Slack `#environment-files` channel.

Put the file here:

```text
WatchTower/backend/.env
```

Important checks:

- The filename must be exactly `.env`.
- It should not be named `env`, `.env.txt`, or `env.txt`.
- Do not commit `.env` to GitHub.

## 3. Install Backend Dependencies

Open a terminal in the backend directory:

```bash
cd WatchTower/backend
npm install
```

## 4. Start The Backend Server

Run this command from the `WatchTower/backend` directory:

```bash
npm start
```

If startup works, you should see:

```text
Connected to MongoDB
WatchTower backend listening on http://localhost:3000
```

Keep this terminal open while working. Press `Ctrl+C` to stop the backend server.


## 5. Tester App

The tester app validates that `collector.js` can send error and performance events to the backend.

Make sure these files are in the same tester folder:

```text
WatchTower-Tester/
  index.html
  collector.js
```

The tester loads the collector through a script tag, the same way a real monitored app would:

```html
<script
  id="collector-script"
  src="./collector.js"
  data-apikey="YOUR_APP_API_KEY"
  data-release="1.0.0">
</script>
```

Important checks:

- Use `src`, not `href`.
- Keep `id="collector-script"` because the current collector reads config from that script element.
- Make sure the API key in `index.html` matches a valid app API key from the backend/database.
- Make sure `collector.js` points to the backend:


Start the tester from the `WatchTower-Tester` directory:

```bash
cd WatchTower-Tester
py -3 -m http.server 5501 --bind 127.0.0.1
```

Open the tester page:

```text
http://127.0.0.1:5501/
```

Use this flow:

1. Click `Load script tag`.
2. Click one tester button, such as `Manual error` or `Page performance`.
3. Check the Network Log.
4. A successful event should show `201 Created`.

The tester includes these event checks:

- `Manual error`: sends a manually tracked error event.
- `Runtime error`: triggers the browser `error` listener.
- `Promise rejection`: triggers the browser `unhandledrejection` listener.
- `Page performance`: reminds you that page performance is sent automatically when the collector script loads.
- `Fetch 12 MB image`: fetches a large public image to test API latency tracking.
- `Fetch 21 MB image`: fetches a larger public image to test API latency tracking.
- `Direct error POST`: sends a direct backend smoke test event.
- `Direct performance POST`: sends a direct backend smoke test event.

The tester intentionally forces `sendBeacon` to fall back to `fetch` so it can display backend status codes and response bodies in the log.

## 6. Check Saved Events

You can check saved collector events through the backend API.

Replace this app id if your app uses a different one:

```text
6a056df9898e4ad76b75665a
```

Error events:

```powershell
$json = (Invoke-WebRequest "http://localhost:3000/api/events/error/apps/6a056df9898e4ad76b75665a").Content | ConvertFrom-Json
$json.events | Sort-Object timestamp -Descending | Select-Object -First 5
```

Performance events:

```powershell
$json = (Invoke-WebRequest "http://localhost:3000/api/events/performance/apps/6a056df9898e4ad76b75665a").Content | ConvertFrom-Json
$json.events | Sort-Object timestamp -Descending | Select-Object -First 5
```
