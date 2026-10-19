# IntentionAI

A Chrome extension and Next.js site that notice when you look tired or stressed while working, log it, and show you which parts of the day it tends to happen in. Team project from PennApps 2025, where it won Best Beginner.

The extension reads your webcam with face-api.js, labels each reading focused, tired, or stressed, sends it to the site, and shows a Chrome notification with a short tip every 15 minutes. The site handles accounts, a weekly dashboard split by time of day (morning, afternoon, evening, late night), Gemini-written insights and suggestions for the week, and notification settings.

## How it's put together

- **Extension** (`chrome-extension/`, Manifest V3). Loads face-api.js and its models from a CDN, runs the tiny face detector and expression model on webcam frames, and maps the expression scores onto the three states: sad and neutral count toward tired, angry, fearful and disgusted toward stressed, neutral and surprised toward focused.
- **Auth.** Email and password, hashed with bcrypt. The server returns a JWT good for 7 days. The dashboard shows your token so you can paste it into the extension popup, which then sends it as a Bearer token with each reading.
- **Database.** MongoDB through Mongoose. Two models: `User` (profile and notification settings) and `EmotionEvent` (state, confidence, time of day).
- **Gemini.** `gemini-1.5-flash` over the REST API, called from `/api/insights` and `/api/suggestions` with the week's percentages.

## API

| Route | Purpose |
|---|---|
| `POST /api/auth/custom-register`, `POST /api/auth/custom-login` | Sign up or log in, returns a JWT |
| `POST /api/extension-auth` | Check a token pasted into the extension |
| `POST /api/emotions` | Record one reading |
| `GET /api/emotions?week=current` | Counts per state and time of day (`week=previous` for last week) |
| `POST /api/insights`, `POST /api/suggestions` | Gemini summaries of the week |
| `GET`, `PUT`, `DELETE /api/user` | Profile, settings, account deletion |

## Running it locally

You need Node 18.18 or newer, a MongoDB instance, and a Gemini API key. Create `.env.local`:

```
MONGODB_URI=mongodb://localhost:27017/workflow-ai
JWT_SECRET=some-long-random-string
GEMINI_API_KEY=...
```

```bash
npm install
npm run dev    # http://localhost:3000
```

To load the extension, open `chrome://extensions`, turn on Developer mode, click "Load unpacked", and pick `chrome-extension/`. Make an account on the site, copy the token from the dashboard, and paste it into the extension popup.

## Notes

- The extension talks to `http://localhost:3000`; the URL is hardcoded in `popup.js` and `background.js`.
- In Chrome the extension is still called "WorkFlow AI", the project's working name.
- If the webcam detector hasn't produced a reading, the extension falls back to a guess based on the time of day.
- Notification tips are fixed text per state. Gemini is only used on the dashboard.
- The settings page saves the notification and webcam settings to your account, but the extension doesn't read them. It uses its own popup toggles and a fixed 15 minute notification interval.
