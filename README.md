# Pascoin

A fictional cryptocurrency mining application built with **HTML5, Tailwind CSS and vanilla JavaScript (ES6+)**.
There is no framework and no server: the browser's **LocalStorage** simulates the database.
It runs by simply opening `index.html` in Chrome.

> Pascoin is an educational project. It is **not** a real cryptocurrency and has no monetary value.

## Academic Context

This project was developed as part of the Advanced Web Programming (JavaScript) course.
It is a complete rebuild of an earlier React/Vite prototype, redesigned so that every important
JavaScript concept (DOM, events, modules, JSON, dates, validation, storage) is written by hand and visible in the source.

## Features

- Registration and login (validated forms), logout with confirmation, protected pages
- Mining: 1 PAS per hour, 24-hour sessions, live countdown, progress bar, up to 24 PAS per session
- Wallet: balance, total earned, mining rewards, number of transactions, wallet address (copy button)
- Transaction history with filters (All, Mining, Rewards, Other)
- Dashboard with statistics, mining summary, recent transactions and quick actions
- Profile: name, phone, email, profile picture (resized before saving), password change
- Dark mode and three languages (English, French, Swahili), both remembered
- Responsive layout (phone, tablet, laptop), toast messages, confirmation dialog, empty states
- About page with FAQ and a contact form (simulated)
- Multi-column footer (links change when logged in, FAQ and Contact shortcuts, back-to-top), translated and dark-mode ready

## Technologies

| Purpose | Tool |
|---|---|
| Structure | HTML5 (semantic: `header`, `nav`, `main`, `section`, `article`, `footer`, `form`, `dialog`) |
| Styling | Tailwind CSS (Play CDN, `darkMode: 'class'`) + a few lines in `css/styles.css` |
| Logic | Vanilla JavaScript (ES6+): one file per responsibility, loaded with ordinary `<script defer>` tags |
| Data | LocalStorage (JSON) |
| Icons | Font Awesome 6 (CDN) |

## How to run

1. Unzip the project and keep the whole `pascoin` folder together.
2. Double-click `index.html` (or drag it into Google Chrome). No server is needed.

An internet connection is needed for the Tailwind and Font Awesome CDNs. The data is saved in the browser
you use, so register and log in with the same browser.

**Demo account:** `demo@pascoin.test` / `demo1234` (created on first launch and flagged `isDemo`, so it is separate from real registrations).

To reset everything, run `localStorage.clear()` in the browser console and reload.

## Project structure

```
pascoin/
├── index.html  login.html  register.html  dashboard.html
├── mining.html  wallet.html  transactions.html  profile.html  about.html
├── css/styles.css
├── assets/images/avatar-default.svg
├── README.md   TESTING.md
└── js/
    ├── app.js           entry point: starts every page
    ├── storage.js       the only file that uses localStorage (the "database")
    ├── utils.js         pure helpers: ids, formatting, validators, escapeHtml
    ├── auth.js          register, login, logout, current user, page protection
    ├── mining.js        mining rules, sessions, reward calculation, mining page
    ├── transactions.js  create / read / filter transactions, transactions page
    ├── wallet.js        balance and statistics, wallet page
    ├── dashboard.js     dashboard page
    ├── profile.js       profile, photo and password
    ├── contact.js       About page contact form
    ├── settings.js      dark mode and language controls
    ├── i18n.js          translations (en, fr, sw), t(), translating the page
    └── ui.js            form errors, toasts, confirm dialog, navbar, HTML render helpers
```

### How a page starts

Every HTML file loads the same scripts, in the same order (`utils`, `storage`, `i18n`, `ui`, `transactions`, `wallet`, `mining`, `auth`, `profile`, `contact`, `settings`, `dashboard`, and `app` last).
The scripts share one global scope, so a file can call the functions defined in the files loaded before it. `app.js` runs last and starts the page. The `<body>` carries `data-page="mining"` and `data-protected="true"`.

```
initializeStorage()  ->  apply theme and language  ->  getCurrentUser()
  ->  protectPage(user)  ->  setupNavbar()  ->  pay finished mining sessions
  ->  run the function for this page (pageInitializers[data-page])
```

## LocalStorage structure

Only five keys (plus one flag) are used. All data goes through `storage.js`.

| Key | Content |
|---|---|
| `pascoin_users` | array of users |
| `pascoin_current_user` | id of the logged-in user (plain text) |
| `pascoin_transactions` | array of transactions |
| `pascoin_mining_sessions` | array of mining sessions |
| `pascoin_settings` | `{ "theme": "light", "language": "en" }` |
| `pascoin_initialized` | `"true"` after the first launch |

```js
// user
{ id, fullName, phone, email, password, walletAddress, photo, createdAt }
// transaction   (type: "mining" | "reward" | "other")
{ id, userId, type, amount, date, status, description }
// mining session   (status: "active" | "completed")
{ id, userId, startTime, endTime, status }
```

`storage.js` reads and writes JSON inside `try/catch`, validates every item, and drops invalid ones
(corrupted data repairs itself instead of crashing the app).

**The balance is never stored.** It is the sum of the user's completed transactions (`wallet.js`, using `reduce()`),
so the balance and the history can never disagree. Registration creates a 10 PAS "Welcome bonus" transaction.

## Mining logic

- Rate: **1 PAS per hour**. A session lasts **24 hours**. Maximum reward: **24 PAS**.
- `startMining()` saves `startTime = Date.now()` and `endTime = startTime + 24 h`. It refuses to start if the user already has an active session.
- The earning is **never counted by a timer**. It is calculated from timestamps whenever it is needed:

```js
// Calculate mining reward based on elapsed time (never more than 24 hours counts)
const elapsed = Math.min(Math.max(now - session.startTime, 0), SESSION_DURATION_MS);
const earning = (elapsed / HOUR_MS) * MINING_RATE_PER_HOUR;
```

- A `setInterval` (every second) only **redraws** the countdown, progress bar and earning from `Date.now()`.
- When `Date.now() >= endTime`, `completeSession()` marks the session `completed`, then creates one `mining` transaction.
  This check also runs every time any page loads, so a session that ended while the browser was closed is paid on the next visit.
- Because the data is timestamps, closing the browser or refreshing the page changes nothing.

## JavaScript concepts demonstrated

| Concept | Where to look |
|---|---|
| Separation of responsibilities | one file per topic in `js/`, loaded in order by the `<script defer>` tags |
| Variables, constants, arrow functions | everywhere; constants such as `KEYS`, `MAX_REWARD` |
| Objects and arrays | users, transactions, sessions; `FILTERS` object of functions in `transactions.js` |
| `map`, `filter`, `find`, `some`, `reduce`, `forEach`, `sort` | `wallet.js` (`reduce`), `storage.js` (`filter`, `map`), `auth.js` (`some`, `find`), `dashboard.js` (`map`) |
| Destructuring, spread, template literals | function parameters, `updateById`, `renderTransactionItem` |
| DOM manipulation | `textContent`, `innerHTML`, `classList`, `dataset`, `createElement` (`ui.js`, `dashboard.js`) |
| Events | `submit`, `click`, `input`, `change`, `keydown`; event delegation in the transactions filters |
| Forms and validation | `validateRegistration`, `validateProfile`, `FormData`, `Object.fromEntries` |
| LocalStorage + JSON | `storage.js` |
| Date and time | `Date.now()`, `toLocaleString`, `formatDuration` |
| Timers | `setInterval` in `mining.js` and `dashboard.js` |
| Promises and `async/await` | `confirmDialog`, `resizeImage`, `copyAddress` |
| Browser APIs | `FileReader`, `<canvas>`, `Image`, `URLSearchParams`, `matchMedia`, Clipboard, `<dialog>` |
| Error handling | `try/catch` in `readData`, `writeData`, photo and clipboard code |
| Optional chaining, nullish coalescing | `pageInitializers[page]?.(user)`, `?? null` |
| Security awareness | `escapeHtml` against HTML injection |

## Explaining the project (quick answers)

- **Authentication:** `registerUser` validates, creates a user and saves it. `loginUser` finds a user with matching email and password and saves **only the user id** in `pascoin_current_user`. `getCurrentUser` rebuilds the user from that id.
- **Protected pages:** `protectPage` (in `auth.js`) runs on every page. If `data-protected="true"` and there is no user, it calls `window.location.replace("login.html")`. It is real JavaScript redirection, not just hidden elements.
- **LocalStorage:** it stores text only, so objects are saved with `JSON.stringify` and read with `JSON.parse`. `storage.js` is the single place that does this.
- **The timer:** `setInterval` runs every second but only redraws. The numbers come from `Date.now()` and the saved timestamps.
- **Transactions:** `createTransaction` adds an object to `pascoin_transactions`. It is called at registration and when a mining session completes.
- **Dashboard data:** `dashboard.js` reads transactions and sessions, computes totals with `reduce`, builds HTML strings with `map().join("")` and puts them into the empty containers of `dashboard.html`.
- **Events:** each form listens to `submit` and calls `preventDefault()` so the page does not reload; `FormData` reads the fields.
- **Validation:** the logic returns an object like `{ email: "error.emailTaken" }`. Empty means valid. `ui.js` shows each message under its field.
- **Dark mode:** `applyTheme` toggles the `dark` class on `<html>` (Tailwind `dark:` variants react to it) and `saveSettings` stores the choice.
- **Language:** `i18n.js` holds a `translations` object. HTML elements carry `data-i18n="key"`; `applyTranslations` replaces their text. Text built in JavaScript uses `t("key")`. Changing language saves it and reloads the page.

## Limitations

- **Security:** LocalStorage authentication is **not secure**. Passwords are stored as **plain text**, and anyone with access to the browser can read and edit all data in DevTools. In a real application passwords must be hashed on a server and sessions protected. LocalStorage is used here only because this university project simulates a backend in the browser.
- **The clock can be cheated:** mining uses the device clock (`Date.now()`). Changing the system time or editing LocalStorage changes the rewards. A real system would use server time.
- **No shared data:** each browser (and each address, e.g. `localhost:5500` and `localhost:8000`) has its own separate data. Nothing is synchronised.
- **Two tabs:** `completeSession` re-checks the stored session before paying, which prevents double payment in practice, but LocalStorage has no real transactions, so a perfectly simultaneous race cannot be excluded.
- **Storage size:** LocalStorage holds about 5 MB. Profile photos are shrunk to 200×200 JPEG (a few KB) for this reason.
- **Fictional features only:** there is no sending or receiving of coins, withdrawals or referral system (these were fake in the original prototype and were removed).
- **No ES modules:** the first version used `import`/`export`, but Chrome blocks modules on `file://`, so the files are ordinary scripts sharing one global scope. This was chosen so the project opens with a double-click; in a larger project modules would be preferred.
- **Language change reloads the page**, a deliberate simplification. Transaction descriptions are stored in English and translated when displayed.
- **Translations:** the Swahili text should be reviewed by a native speaker.
- **CDNs:** Tailwind's Play CDN is meant for development. A production build would compile the CSS.

## Testing

`TESTING.md` contains a manual checklist covering authentication, mining (including simulating elapsed time from the console), wallet, profile, settings, robustness and responsive layouts.
