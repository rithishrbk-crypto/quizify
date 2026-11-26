# Quizify: Online Quiz Application

Node.js + Express backend, vanilla JS frontend, JSON-file storage, glassmorphism UI.

## Run

```bash
npm install
cp .env.example .env     # then set SESSION_SECRET to a long random string
npm start                # http://localhost:3000
```

Generate a secret: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`

The `data/` folder (`users.json`, `quizzes.json`, `attempts.json`) is created automatically on first run, with five sample quizzes of 10 questions each.

## Structure

```
server.js            App setup, sessions, page routes, error handling
routes/auth.js       Register, login, logout, current user
routes/quiz.js       Dashboard, quiz start, answer save, submit, history, results
lib/db.js            Async JSON store: atomic writes, write queue, corruption backup
lib/scoring.js       Server-side scoring
lib/seed.js          Sample quizzes
middleware/auth.js   Route protection
views/               HTML pages (protected pages are served only to logged-in users)
public/              CSS and client JavaScript
```

## How it meets the requirements

- **Passwords**: bcrypt (cost 12). Only hashes are stored.
- **Sessions**: express-session, httpOnly + SameSite cookie, expires after `SESSION_HOURS` of inactivity, session id regenerated on login.
- **Answers stay secret**: the quiz API sends questions and options only. Correct answers and explanations are returned after submission, from the results endpoint.
- **Server-side scoring**: the client sends selected option indexes; the server computes correct, incorrect, unanswered, marks, percentage and pass/fail (`PASS_PERCENTAGE`, default 50).
- **One attempt per quiz**: enforced in a serialised write, so double-clicks and parallel requests cannot create or submit twice.
- **Timer integrity**: the deadline is stored on the server. Answers are saved as you go; if the timer expires while you are away, the attempt is scored from the saved answers.
- **Privacy**: results and history are filtered by the logged-in user. Someone else's attempt returns 404.
- **Data safety**: writes go to a temp file and are renamed into place. An unreadable JSON file is renamed to `*.corrupt-<time>` and recreated, never silently overwritten.

## Notes

- Sessions use Express' in-memory store, which is fine for a college project. Users must log in again after a server restart; stored data is not affected.
- To add quizzes, edit `data/quizzes.json` (same shape as `lib/seed.js`) and restart.
- Behind HTTPS in production, set `NODE_ENV=production` and `COOKIE_SECURE=true`.
