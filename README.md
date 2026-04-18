# 📦 File2Linken

A Telegram bot that accepts forwarded files and generates **direct download** and **stream links** — bypassing Telegram's 20MB Bot API limit using MTProto (gramjs). Supports videos, audio, photos, documents, voice messages, and more.

---

## ✨ Features

- 📤 Forward any file to the bot → get instant download + stream links
- ▶️ In-browser video/audio streaming with seek support
- ⚡ ffmpeg fast-start remux for instant browser playback
- 🔗 Unlimited file size via Telegram MTProto (gramjs)
- 🌐 Self-hostable on Railway (free tier)

---

## 🏗️ Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 24, TypeScript 5.9 |
| Monorepo | pnpm workspaces |
| API | Express 5 |
| Bot | Telegraf v4 |
| MTProto | gramjs (`telegram`) |
| Database | PostgreSQL + Drizzle ORM |
| Validation | Zod v4 |

---

## ⚙️ Environment Variables

Copy `.env.example` to `.env` for local dev, or set these in Railway's Variables dashboard:

| Variable | Description |
|---|---|
| `PORT` | Server port (Railway sets this automatically) |
| `BASE_URL` | Your public deployment URL, e.g. `https://your-app.up.railway.app` |
| `TELEGRAM_BOT_TOKEN` | Bot token from [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_API_ID` | API ID from [my.telegram.org](https://my.telegram.org) |
| `TELEGRAM_API_HASH` | API Hash from [my.telegram.org](https://my.telegram.org) |
| `LOG_CHANNEL_ID` | Telegram channel ID for activity logs (e.g. `-1001234567890`) |
| `TELEGRAM_SESSION` | gramjs session string (see first-run note below) |
| `DATABASE_URL` | PostgreSQL connection string |

### 📝 Getting `TELEGRAM_SESSION`

On the **very first deployment**, leave `TELEGRAM_SESSION` blank. gramjs will authenticate with Telegram and print a line like:

```
TELEGRAM SESSION UPDATED — copy this value to your TELEGRAM_SESSION environment variable
{ session: "1BQANOTEu..." }
```

Copy that session string into the `TELEGRAM_SESSION` variable and redeploy. All future restarts will reconnect in ~1–2 seconds instead of 5–15 seconds.

---

## 🚀 Deploy to Railway (Free)

### 1. Fork / push this repo to GitHub

### 2. Create a Railway account
Go to [railway.app](https://railway.app) → sign up free with GitHub.

### 3. Add a PostgreSQL database
New Project → Add Service → **Database → PostgreSQL**
Copy the `DATABASE_URL` from its Variables tab.

### 4. Deploy the bot
New Service → **Deploy from GitHub repo** → select this repo.
Railway auto-detects `railway.toml` and builds with nixpacks.

### 5. Set environment variables
In your Railway service → Variables tab, add all variables from the table above.

### 6. Get your public domain
Settings → Networking → **Generate Domain**
You'll get `https://your-app.up.railway.app` — set this as `BASE_URL`.

### 7. Run the database migration
In Railway → your service → **Shell** tab:
```bash
pnpm --filter @workspace/db run push
```

### 8. Test
- Open `https://your-app.up.railway.app` → should show the landing page
- Open `https://your-app.up.railway.app/api/healthz` → should return `{"status":"ok"}`
- Send a file to your bot → should get download + stream links pointing to your domain

---

## 💻 Local Development

```bash
# Install dependencies
pnpm install

# Set up environment
cp .env.example .env
# Fill in your values in .env

# Push DB schema
pnpm --filter @workspace/db run push

# Run the server + bot
pnpm --filter @workspace/api-server run dev
```

---

## 📂 Project Structure

```
artifacts/
  api-server/         # Express server + Telegraf bot + gramjs streaming
    src/
      bot/            # Telegram bot handlers
      lib/            # gramjsClient, telegramStream, ffmpegStream
      routes/         # download, stream, stream-page, home
lib/
  db/                 # Drizzle ORM schema + PostgreSQL client
  api-spec/           # OpenAPI spec
  api-client-react/   # Generated React hooks
  api-zod/            # Generated Zod validators
```

---

## 🔐 Security Notes

- Never commit `telegram_session.txt` or `.env` to a public repo (both are in `.gitignore`)
- `TELEGRAM_SESSION` contains full account auth — treat it like a password
- Rotate your session if you ever accidentally expose it: delete the env var, restart, copy the new session from logs
