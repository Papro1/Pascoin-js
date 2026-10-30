# Pascoin — manual test checklist

Open `index.html` in Chrome (double-click), and keep DevTools (F12) → Application → Local Storage visible. Start from a clean state with
`localStorage.clear()` in the console, then reload.

Demo account: `demo@pascoin.test` / `demo1234`

## Authentication
- [ ] Open `dashboard.html` without logging in → redirected to `login.html`
- [ ] Register with empty fields → one message under each field
- [ ] Register with different passwords → "Passwords do not match."
- [ ] Register correctly → login page with a green message; the user has a 10 PAS transaction
- [ ] Register again with the same email (any capitals) → "already exists"
- [ ] Log in with a wrong password → "Incorrect email or password."
- [ ] Log in correctly → dashboard; `pascoin_current_user` is set
- [ ] Open `login.html` while logged in → sent back to the dashboard
- [ ] Log out → confirmation → login page; `pascoin_current_user` is gone

## Mining
- [ ] Press **Start mining** → status Active, button disabled, countdown ~24:00:00
- [ ] Refresh the page → countdown continues; start button still disabled
- [ ] Close the tab, reopen the site → same session, earning has grown
- [ ] Simulate time (console):
      `const x = getAll("miningSessions")[0]; const h = 3600000, n = Date.now();
      updateById("miningSessions", x.id, { startTime: n - 6.5*h, endTime: n + 17.5*h });`
      then reload → about 6.5 PAS, 27.1 %
- [ ] Simulate the end (same line with `startTime: n - 30*h, endTime: n - 6*h`) and reload → toast "+24.00 PAS",
      balance +24, exactly one mining transaction, status Inactive
- [ ] Reload several times → no second payment
- [ ] Start another session → works

## Wallet and transactions
- [ ] Wallet balance equals the dashboard balance
- [ ] Transactions page lists each reward; filters All / Mining / Rewards / Other work
- [ ] A second account sees only its own data

## Profile
- [ ] Change name and phone, save, refresh → values remain
- [ ] Use another account's email → refused
- [ ] Upload a large photo → shown, remains after refresh (small Base64 string in `pascoin_users`)
- [ ] Upload a PDF → error message
- [ ] Change password with a wrong current password → refused; correct → old password stops working

## Settings
- [ ] Dark mode toggle → refresh → still dark; open another page → still dark
- [ ] Language FR / SW / EN → whole interface changes; refresh → still the same language

## Footer
- [ ] Logged out: footer shows Home, Mining, Wallet, Log in, Register, About, FAQ, Contact
- [ ] Logged in: it also shows Dashboard, Transactions, Profile, and hides Log in / Register
- [ ] "FAQ" and "Contact" scroll to their sections on the About page, just under the header
- [ ] "Back to top" scrolls up; footer text changes with the language and with dark mode
- [ ] On a phone width (logged in) the footer stays fully visible above the bottom bar

## Robustness
- [ ] Set `pascoin_transactions` to `{{bad` → pages still load, empty lists
- [ ] Set `pascoin_users` to `[1,2]` while logged in → redirected to login, no crash

## Responsive (DevTools device toolbar)
- [ ] 375 px: hamburger menu, bottom bar on logged-in pages, no horizontal scrolling
- [ ] 768 px: hamburger menu, two-column cards
- [ ] 1280 px: top navigation links, no bottom bar
- [ ] French at exactly 1024 px: top links do not wrap or overflow
