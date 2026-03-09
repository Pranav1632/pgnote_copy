# pgnote (under developemt)

> **Your thoughts. Your rules. Zero passwords.**

### REMOVED SOME BACKEND for SECURITY PURPOSE
A secure, minimal personal note-pasting app where your authentication is a *method you define* — not a password anyone can steal, reset, or leak.

🔗 **Live at [pgnote.vercel.app](https://pgnote.vercel.app)**

---

## What is pgnote?

pgnote is a passwordless personal notes app built for people who want private, fast, distraction-free writing —  without trusting any platform with their credentials.

Instead of a password, you define an **algorithm** — a personal transformation rule applied to a random key. Every login, the server challenges you with the *result* of your algo. You mentally reverse it and type the original key. No password is ever stored. Not even hashed.

```
Server generates:   X9mK#p7Q
Applies your algo:  K#pQ7m9X  ← shown on screen (challenge)
You type back:      X9mK#p7Q  ← only you know how to reverse it
```

If someone steals the entire database — there is nothing useful in it.

---

## How the Algo Works

During signup you define an algo by slicing a key into chunks by index positions:

```
Key length: 8

Chunk 1 → positions [0–2], reversed
Chunk 2 → positions [3–5], not reversed
Chunk 3 → positions [6–7], reversed

Display order: Chunk 2 → Chunk 3 → Chunk 1
```

At login the server generates a fresh random key, applies your algo, and shows you the challenge. You reverse the process mentally and type the original key. The server compares. Match means access granted.

The raw key lives in Redis for **2 minutes** then is permanently deleted. It is never written to MongoDB.

---

## Features

- **Passwordless auth** — algo-based challenge-response, nothing sensitive in DB
- **Your own URL** — every user gets `pgnote.vercel.app/yourslug`
- **Bash-style editor** — monospace, dark, distraction-free writing environment
- **Folder organisation** — create folders, move notes, delete with confirmation
- **Auto-save** — triggers every 4 newlines typed + 3 second debounce fallback
- **Manual save** — Ctrl+S always works
- **Collapsible sidebar** — folders collapse to 2-letter badges, editor expands to fill space
- **Session timeout** — you choose at signup: 2 / 5 / 10 / 30 / custom minutes
- **Inactivity logout** — timer resets on keypress, mousemove, or paste
- **Warning before logout** — toast notification 30 seconds before session ends
- **Recovery email** — optionally receive a plain-English description of your algo by email
- **Custom MongoDB** — optionally connect your own MongoDB database to store your notes
- **Rate limiting** — 5 login attempts per 15 minutes per IP and slug
- **Security headers** — CSP, HSTS, X-Frame-Options, nosniff on all responses

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router) + TypeScript |
| Database | MongoDB + Mongoose |
| Cache + Rate Limiting | Upstash Redis |
| Session Auth | iron-session (httpOnly cookies) |
| Styling | Tailwind CSS v4 + shadcn/ui |
| Validation | Zod + react-hook-form |
| Email | Nodemailer (SMTP) |
| Encryption | Node.js crypto (AES-256-CBC) |
| Deployment | Vercel |

---

## Project Structure

```
pgnote/
├── app/
│   ├── (auth)/signup/        # Multi-step signup flow
│   ├── [slug]/               # Login + challenge page per user
│   │   └── notes/            # Protected notes dashboard
│   └── api/
│       ├── auth/             # signup, challenge, verify, logout, recovery
│       ├── notes/            # CRUD for notes
│       ├── folders/          # CRUD for folders
│       └── user/slug-check/  # Real-time slug availability
├── components/
│   ├── auth/                 # AlgoBuilder, ChallengeBox, LoginForm
│   ├── notes/                # BashTextArea, FolderSidebar, NotesList
│   ├── layout/               # SessionTimer, EndSessionButton
│   └── shared/               # LogoutWarningToast
├── lib/                      # Core utilities and engine
├── models/                   # Mongoose schemas
└── middleware.ts              # Route protection
```

---

## Database Schema

```
User {
  slug           String   unique permanent URL handle
  algo           Object   keyLength + chunks + order (NO password field)
  sessionTimeout Number   chosen by user at signup
  recoveryEmail  String?  optional
  customMongoUri String?  AES-256 encrypted if provided
  lastLogin      Date?
}

Folder {
  userId   ObjectId
  name     String
}

Note {
  userId   ObjectId
  folderId ObjectId
  content  String
  timestamps
}
```

**No password field. Anywhere. Ever.**

---

## Security Design

| Threat | How pgnote handles it |
|--------|-----------------------|
| Database breach | No passwords stored — algo only, useless without the user's mind |
| Brute force | 5 attempts per 15 min per IP and slug via Upstash sliding window |
| Session hijack | httpOnly + Secure + SameSite=Strict cookie via iron-session |
| CSRF | SameSite=Strict + userId always sourced from server session |
| Clickjacking | X-Frame-Options: DENY |
| Stale challenges | Redis TTL 120 seconds — key auto-deletes after expiry |
| Replay attack | Challenge key deleted from Redis immediately on correct login |
| Custom URI exposure | AES-256-CBC encrypted before writing to MongoDB |

---

## Running Locally

```bash
git clone https://github.com/yourusername/pgnote.git
cd pgnote
npm install
cp .env.example .env.local
# fill in your environment variables
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## Environment Variables

```env
MONGODB_URI=
SESSION_SECRET=
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
NEXT_PUBLIC_APP_URL=
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=
SMTP_FROM=
```

---

## Why No Password?

Most auth systems protect a secret stored somewhere on a server. If that server is breached, secrets leak — even hashed passwords can be cracked given enough time and compute.

pgnote stores no secret at all. Your algo is not sensitive — it is just a transformation rule. The actual secret lives entirely in your head: knowing how to reverse the challenge. An attacker with full database access gains nothing because there is no password, no hash, and no key to crack.

The only way to authenticate is to know the method. That method is never written down anywhere on the server.

---

## Deployment

Deployed on Vercel. All API routes are forced dynamic to prevent static pre-rendering at build time. MongoDB hosted on Atlas. Redis hosted on Upstash.

---

## License

MIT — use it, fork it, build on it.

---

<p align="center">
  Built by a second-year BTech student who wanted notes without passwords.
  <br /><br />
  <a href="https://pgnote.vercel.app"><strong>pgnote.vercel.app</strong></a>
</p>