# Open Slots

A bookable availability calendar that runs as a single HTML file on GitHub Pages.
Month view and week view, 15-minute precision, drag on the grid to pick a time.
No build step, no framework, no server of your own.

---

## Part 1 — Get it online (about 5 minutes)

### 1. Make the repository

On GitHub, click **+** (top right) → **New repository**.

- **Repository name:** `boscofungg-web.github.io` — using *your own username* exactly, all lowercase, followed by `.github.io`. This exact name is what gets you the short address.
- **Public** (GitHub Pages is only free on public repositories).
- Tick **Add a README file** so the repo isn't empty.
- **Create repository**.

### 2. Upload the file

In the new repository: **Add file** → **Upload files** → drag `index.html` in → **Commit changes**.

That's the whole deployment. There is nothing to build or compile.

### 3. Switch Pages on

**Settings** (repo tab) → **Pages** in the left sidebar → under **Build and deployment**:

- **Source:** Deploy from a branch
- **Branch:** `main`, folder `/ (root)`
- **Save**

### 4. Wait a minute, then visit

```
https://boscofungg-web.github.io
```

First publish takes 1–3 minutes. After that, every commit is live in under a minute.
HTTPS is on automatically and free.

### 5. Make later edits without leaving the browser

Open `index.html` in the repo, click the pencil icon, edit, **Commit changes**. Done.

---

## Part 2 — About the "free domain"

Worth being precise, because the word *domain* covers two different things:

| What | Cost | Looks like |
|---|---|---|
| GitHub Pages subdomain | **Free forever** | `boscofungg-web.github.io` |
| Your own domain name | **You buy it, ~US$10–15/yr** | `boscofung.com` |

GitHub hosts your site for free and gives you free HTTPS, but it does not sell or give away
domain names — nobody does, because registrars pay a fee per name per year. If someone
advertises a "free domain," it's either a subdomain of *their* domain, or a first-year
discount on a paid one.

**If the `.github.io` address is fine,** you're done — it's stable, it's HTTPS, and it costs nothing.

**Genuinely free alternatives to a paid domain**, if you want something shorter:

- **`is-a.dev`** — free `yourname.is-a.dev`, granted by pull request. Aimed at developers.
- **`eu.org`** — free `yourname.eu.org`, has been running since 1996. Approval takes a few days.
- **`js.org`** — free `yourname.js.org`, but only for JavaScript-related projects.

**If you buy a real domain** (Cloudflare Registrar sells at cost, roughly US$10/yr for `.com`),
point it at GitHub with these DNS records:

For a bare domain like `boscofung.com` — four **A** records, all with the name `@`:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

For `www.boscofung.com` — one **CNAME** record, name `www`, value `boscofungg-web.github.io`.

Then in **Settings → Pages → Custom domain**, enter the domain and save. Tick **Enforce HTTPS**
once it becomes available (it can take up to 24 hours while the certificate is issued).

---

## Part 3 — Make bookings actually shared

**Read this bit carefully, because it's the part people get wrong.**

GitHub Pages serves files. It cannot store anything. Straight after step 4 above, your calendar
saves bookings into *each visitor's own browser* — so you see your bookings, your student sees
theirs, and neither of you sees the other's. Useless for real booking.

To make it a shared calendar you need a database, and Firebase Firestore has a free tier that is
far more than a booking calendar will ever use (50,000 reads and 20,000 writes a day).

### Setting up Firebase

1. Go to **console.firebase.google.com** and sign in with a Google account.
2. **Create a project.** Any name. Turn Google Analytics **off** — you don't need it.
3. In the left sidebar: **Build → Firestore Database → Create database**.
   Choose **Start in production mode**, and pick a location near you (`asia-east2` is Hong Kong).
4. Now register the website. Click the **gear icon → Project settings**, scroll to **Your apps**,
   and click the **`</>`** (web) icon. Give it a nickname, click **Register app**.
5. Firebase shows you a `firebaseConfig` block. **Copy the values.**
6. Open `index.html`, find the `CALENDAR_CONFIG` block near the top, and replace `firebase: null`
   with your values:

```js
firebase: {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.firebasestorage.app",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
}
```

7. Back in Firebase: **Firestore Database → Rules** tab. Replace what's there with the contents
   of `firestore.rules` from this repo, and click **Publish**.
8. Commit `index.html` to GitHub. Within a minute the calendar is shared.

**That `apiKey` is not a secret.** Every Firebase web app ships it in public JavaScript — it
identifies your project, it doesn't authorise anything. What actually controls access is the
Rules file in step 7, which is why that step is not optional.

---

## Part 4 — Owner mode

Anyone visiting the plain URL can add a booking request and edit or cancel *their own* requests.
They cannot touch anyone else's.

You get full control by adding your secret word to the end of the URL:

```
https://boscofungg-web.github.io/#owner
```

In owner mode you can edit and delete any booking, and new entries default to **Confirmed**
rather than **Requested**. An `owner` badge appears next to the title so you know it's on.

**Change the secret word** before you share anything. In `index.html`:

```js
ownerKey: "owner",        →      ownerKey: "some-word-only-you-know",
```

Then your private URL becomes `https://boscofungg-web.github.io/#some-word-only-you-know`.

**Be honest with yourself about what this is.** It's a lock on a door, not a bank vault. The
secret word sits in the page's JavaScript, so anyone determined enough to open the browser's
developer tools can find it and grant themselves owner mode. For a tutoring calendar shared with
students, that's a perfectly sensible trade — it stops accidents, not attackers. If you ever need
real security, the upgrade is Firebase Authentication, where the *server* checks who you are
instead of the page checking a word.

Related: don't put anything in a booking note that would be a problem if a stranger read it. The
whole calendar is public by design — that's what lets people see when you're free.

---

## Part 5 — Things you can change

Everything routine lives in the `CALENDAR_CONFIG` block at the top of `index.html`:

| Setting | What it does |
|---|---|
| `title` | Page heading and browser tab |
| `tz` | The small line under the heading |
| `ownerKey` | Your secret word for owner mode |
| `weekStartsMonday` | `false` gives you a Sunday-first week |
| `dayStart` / `dayEnd` | The hours the week grid shows. Currently `08:00`–`23:00`, so the earliest session is 8am and the latest ends at 11pm. Nothing outside these hours can be booked — the form refuses it and the time pickers won't offer it. |
| `demo` | `true` fills the calendar with examples so you can try it. **Set to `false` before sharing.** |
| `firebase` | Your Firebase config, or `null` for browser-only storage |

For colours, look for the `:root` block in the `<style>` section. `--accent` is the green used for
confirmed bookings, `--brass` the amber used for requests. Changing those two changes the whole
palette; light and dark themes are already handled.

---

## How to use it

| | |
|---|---|
| **Book a slot** | Drag down the week grid, or click any day in month view |
| **Change one** | Click the booking |
| **Switch views** | The Month / Week buttons, or press `M` and `W` |
| **Move around** | Arrow keys, or `T` for today |
| **New booking** | `N` |
| **Export** | Downloads an `.ics` file that imports into Google Calendar, Apple Calendar or Outlook |

Times snap to 15-minute steps when you drag, and you can type any time you like in the form —
11:30 to 12:15 works exactly as you'd expect. Overlapping bookings sit side by side in the week
grid, and the form warns you before you double-book without stopping you.

The week grid runs **08:00 to 23:00**. The overnight hours are gone entirely, so the working day
fills the screen instead of being squeezed into a third of it. Change `dayStart` and `dayEnd` in
the config if your hours move.
