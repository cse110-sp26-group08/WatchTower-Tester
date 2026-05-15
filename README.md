# WatchTower Tester

Use this tester to verify that the WatchTower collector can send error and performance events to the backend.

The tester is hosted on GitHub Pages:

```text
https://cse110-sp26-group08.github.io/WatchTower-Tester/
```

The hosted tester sends events to the backend URL hardcoded in `collector.js`:

```text
http://localhost:3000
```

That means each person using the tester must run the WatchTower backend locally first.

## 1. Get The Latest WatchTower Backend

If you do not have the WatchTower project yet, clone it from GitHub:

```bash
git clone git@github.com:cse110-sp26-group08/WatchTower.git
cd WatchTower
```

If you already cloned it, update your local copy:

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

Check the filename carefully:

- It must be exactly `.env`.
- It should not be named `env`, `.env.txt`, or `env.txt`.
- Do not commit `.env` to GitHub.

## 3. Install Backend Dependencies

Open a terminal in the backend directory:

```bash
cd WatchTower/backend
npm install
```

## 4. Start The Backend Server

Run this from the `WatchTower/backend` directory:

```bash
npm start
```

If startup works, you should see:

```text
Connected to MongoDB
WatchTower backend listening on http://localhost:3000
```

Keep this terminal open while using the tester. Press `Ctrl+C` to stop the backend.

## 5. Open The Hosted Tester

Open:

```text
https://cse110-sp26-group08.github.io/WatchTower-Tester/
```

Use this flow:

1. Click `Load script tag`.
2. Click a test button, such as `Manual error`, `Runtime error`, or `Fetch 12 MB image`.
3. Check the Network Log.
4. A successful event should show `201 Created`.

The tester buttons include:

- `Manual error`: triggers a normal JavaScript error for the collector to report.
- `Runtime error`: triggers the browser `error` listener.
- `Promise rejection`: triggers the browser `unhandledrejection` listener.
- `Page performance`: reminds you that page performance is sent automatically when the collector script loads.
- `Fetch 12 MB image`: fetches a large public image to test fetch-latency tracking.
- `Fetch 21 MB image`: fetches a larger public image to test fetch-latency tracking.
- `Direct error POST`: bypasses the collector and smoke-tests the error endpoint.
- `Direct performance POST`: bypasses the collector and smoke-tests the performance endpoint.

The tester intentionally forces `sendBeacon` to fall back to `fetch` so it can display backend status codes and response bodies.

## 6. Check Saved Events

You can check saved events through the backend API.

Current app id used by the tester:

```text
6a056df9898e4ad76b75665a
```

Error events:

```powershell
$json = (Invoke-WebRequest "http://localhost:3000/api/events/error/apps/6a056df9898e4ad76b75665a" -UseBasicParsing).Content | ConvertFrom-Json
$json.events | Sort-Object timestamp -Descending | Select-Object -First 5
```

Performance events:

```powershell
$json = (Invoke-WebRequest "http://localhost:3000/api/events/performance/apps/6a056df9898e4ad76b75665a" -UseBasicParsing).Content | ConvertFrom-Json
$json.events | Sort-Object timestamp -Descending | Select-Object -First 5
```

## Notes

- `localhost` means the computer running the browser. If a teammate opens the GitHub Pages tester, it will send events to that teammate's local backend.
- The red JavaScript errors in Chrome DevTools are expected for the error test buttons. Those errors are intentionally thrown so the collector can report them.
- The `Errors` counter in the tester means failed network/backend responses, not intentionally thrown JavaScript test errors.
