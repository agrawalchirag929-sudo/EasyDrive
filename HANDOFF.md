# EasyDrive — Developer Handoff Document

> **Generated:** 2026-10-15  
> **Purpose:** Safe project continuity across agents or accounts.  
> **Ground truth:** Every fact in this document was verified by reading the actual workspace files. Items that have NOT been browser-tested are explicitly flagged as **[Untested in browser]**.  
> **Rule for incoming agents:** Read this document first. Inspect the files it references before touching any code. Do not rebuild or overwrite working features.

---

## 1. Project Overview

**EasyDrive** is a premium car-rental booking platform targeting the Indian market. The intended user experience is editorial-luxury: dark ink sidebar, warm cream page background, copper accents, Playfair Display for headlines, DM Sans for body text, and restrained animations. It is NOT a generic colourful SaaS dashboard.

### Core user flows (planned, not all implemented)
1. Browse the fleet → filter by category, transmission, price, availability
2. Select a vehicle → open a booking modal → pick location and dates → see price breakdown → confirm
3. Track bookings in "My Bookings" (upcoming / completed / cancelled)
4. Sign in / register via an auth modal
5. View and edit profile; contact support

### Technology decisions
| Concern | Choice | Rationale |
|---------|--------|-----------|
| Frontend | Vanilla HTML + CSS + JS | No build step needed; single-page app via innerHTML injection |
| Styling | Vanilla CSS with CSS custom properties | Full design-token system; no framework dependency |
| SPA routing | Custom `app.navigate()` | Simple view-registry pattern; no URL changes |
| Backend (planned) | Node.js + Express | Matches `.env.example` configuration |
| Database (planned) | MongoDB + Mongoose | Connection string in `.env.example` |
| Auth (planned) | JWT (7-day expiry) | Specified in `.env.example`; frontend hooks stubbed in `auth.js` |

---

## 2. Current Architecture

### Verified folder structure (as of 2026-10-15)

```
MAIN/                          <- project root
├── HANDOFF.md                 <- this file
├── README.md                  <- phase checklist and startup instructions
├── .gitignore                 <- ignores .env, node_modules, logs, dist, .DS_Store
├── .git/                      <- git repo (exists, no remote confirmed)
│
├── frontend/
│   ├── index.html             <- 232 lines  — SPA shell (sidebar, topbar, page-content div)
│   └── assets/
│       ├── css/
│       │   └── style.css      <- 1,725 lines — full design system
│       ├── js/
│       │   ├── data.js        <- 174 lines   — mock CARS, LOCATIONS, MOCK_BOOKINGS
│       │   ├── home.js        <- 215 lines   — Home view (hero, stats, featured cars, strip)
│       │   ├── catalogue.js   <- 284 lines   — Catalogue view (REWRITTEN in Phase 2)
│       │   ├── bookings.js    <- 106 lines   — My Bookings view (Phase 1 state)
│       │   ├── auth.js        <- 296 lines   — Auth modal UI + mock handlers
│       │   └── app.js         <- 533 lines   — SPA router, booking modal, toast, profile, support
│       └── images/
│           ├── hero_car.jpg   <- 618 KB  — hero background
│           ├── car_sedan.jpg  <- 597 KB
│           ├── car_suv.jpg    <- 514 KB
│           ├── car_hatchback.jpg <- 527 KB
│           ├── car_sports.jpg <- 599 KB
│           └── car_mpv.jpg    <- 533 KB
│
└── backend/
    └── .env.example           <- only file that exists; no server.js, no package.json
```

### Script load order in `index.html` (verified)
```html
<script src="assets/js/data.js"></script>      <!-- 1: globals CARS, LOCATIONS, MOCK_BOOKINGS -->
<script src="assets/js/home.js"></script>       <!-- 2: renderHomeView, buildCarCard, formatPrice, getTodayStr -->
<script src="assets/js/catalogue.js"></script>  <!-- 3: renderCatalogueView, afterCatalogueRender, toggleFavourite, escHtml -->
<script src="assets/js/bookings.js"></script>   <!-- 4: renderBookingsView, formatDate -->
<script src="assets/js/auth.js"></script>       <!-- 5: buildAuthModal, openAuthModal, handleLogin, handleRegister -->
<script src="assets/js/app.js"></script>        <!-- 6: app IIFE, showToast, DOMContentLoaded boot -->
```

### Global window namespace (verified exports)
| Symbol | Defined in | Used by |
|--------|-----------|---------|
| `window.CARS` | data.js | home.js, catalogue.js, app.js |
| `window.LOCATIONS` | data.js | home.js, app.js |
| `window.MOCK_BOOKINGS` | data.js | bookings.js |
| `window.buildCarCard` | home.js | home.js (featured cars) |
| `window.formatPrice` | home.js | home.js, catalogue.js, bookings.js, app.js |
| `window.getTodayStr` | home.js | home.js, app.js |
| `window.renderHomeView` | home.js | app.js VIEW_RENDERERS |
| `window.renderCatalogueView` | catalogue.js | app.js VIEW_RENDERERS |
| `window.afterCatalogueRender` | catalogue.js | NOT YET CALLED by app.js (see bug list) |
| `window.setCatalogueFilter` | catalogue.js | NOT YET CALLED by any code |
| `window.toggleFavourite` | catalogue.js | inline onclick in buildCatalogueCard |
| `window.escHtml` | catalogue.js | catalogue.js internally |
| `window.renderBookingsView` | bookings.js | app.js VIEW_RENDERERS |
| `window.formatDate` | bookings.js | bookings.js internally |
| `window.openAuthModal` | auth.js | index.html topbar buttons, app.js |
| `window.closeAuthModal` | auth.js | auth.js, app.js |
| `window.buildAuthModal` | auth.js | app.js init() |
| `window.handleLogin` | auth.js | login form onsubmit |
| `window.handleRegister` | auth.js | register form onsubmit |
| `window.togglePasswordVisibility` | auth.js | password eye buttons |
| `window.showToast` | app.js | everywhere |
| `window.dismissToast` | app.js | toast close buttons |
| `app` (IIFE) | app.js | everywhere as `app.navigate()`, `app.openBooking()`, etc. |

### CSS design tokens (key values, verified from style.css)
| Token | Value | Usage |
|-------|-------|-------|
| `--ink` | `#101820` | Sidebar BG, primary buttons, text |
| `--cream` | `#F5F3EE` | Page background |
| `--surface` | `#FFFFFF` | Cards, modals, topbar |
| `--copper` | `#D97736` | Accent: nav active bar, eyebrows, CTA tint |
| `--copper-dark` | `#B85E20` | Hover state for copper elements |
| `--sidebar-width` | `240px` | Fixed sidebar width on desktop |
| `--font-body` | DM Sans | Body text (Google Fonts) |
| `--font-display` | Playfair Display | Hero headline only (Google Fonts) |
| Breakpoints | 1100px, 768px, 480px | Responsive media queries |

**Important:** style.css contains BOTH new token names (`--copper`, `--sp-4`, `--r-lg`) AND legacy aliases (`--clr-accent`, `--space-4`, `--radius-lg`). Both sets must be kept because inline styles in JS files use both naming conventions.

### Backend (verified: does NOT exist yet)
The `backend/` directory contains only `.env.example`. There is **no `package.json`, no `server.js`, no `src/` directory, no models, no routes, no controllers, no middleware**. Everything listed in the README's folder tree for the backend is aspirational planning only.

---

## 3. Completed Work

### Phase 1 — Frontend UI Foundation (complete)

| Feature | File(s) | Tested? |
|---------|---------|---------|
| SPA shell: sidebar + topbar + content area | index.html | Code complete |
| Sidebar navigation with active state | index.html, app.js | Code complete |
| Mobile hamburger + overlay | index.html, app.js, style.css | [Untested in browser] |
| Escape key closes all modals | app.js | Code complete |
| Toast notification system | app.js, style.css | Code complete |
| Full CSS design token system | style.css | Code complete |
| Inline SVG icon sprite | index.html | Code complete |
| Home: hero with gradient overlay and image | home.js, style.css | Code complete |
| Home: hero quick-search bar rendered | home.js | Rendered; Search button has no onclick |
| Home: KPI stats grid | home.js | Code complete |
| Home: featured cars (first 3) | home.js | Code complete |
| Home: feature strip (4 trust pillars) | home.js | Code complete |
| Car card component `buildCarCard()` | home.js | Code complete |
| Catalogue: filter toolbar rendered | catalogue.js | Rendered; not wired (see Phase 2) |
| Catalogue: all 6 cars displayed | catalogue.js | Broken — grid empty (see B1) |
| My Bookings: card list from mock data | bookings.js | Code complete |
| My Bookings: filter tabs rendered | bookings.js | Rendered; not wired (see B3) |
| Auth modal: login + register forms | auth.js | Code complete |
| Auth modal: tab switching + validation | auth.js | Code complete |
| Auth modal: mock login/register | auth.js | Code complete (no real API) |
| Booking modal: per-car, date pickers, price | app.js | Code complete |
| Booking modal: auth guard | app.js | Code complete |
| Booking modal: mock submit (toast + navigate) | app.js | Code complete (no real API) |
| Profile view (logged-in + guest states) | app.js | Code complete (static mock data) |
| Support view + contact form stub | app.js | Code complete |
| Responsive CSS breakpoints | style.css | [Untested in browser] |

### Phase 2 — Catalogue Filtering (code written, NOT fully connected)

| Feature | File | Status |
|---------|------|--------|
| Live text search (name, category, fuel) | catalogue.js | Logic written |
| Category filter dropdown | catalogue.js | Logic written |
| Transmission filter dropdown | catalogue.js | Logic written |
| Sort: name, price asc/desc, rating | catalogue.js | Logic written |
| Availability-only toggle | catalogue.js | Logic written |
| Reset all filters button | catalogue.js | Logic written |
| Filter count label "Showing X of 6" | catalogue.js | Logic written |
| Empty state when no results | catalogue.js | Logic written |
| Favourites toggle (localStorage) | catalogue.js | Logic written |
| `escHtml()` XSS guard | catalogue.js | Written |
| `afterCatalogueRender()` called by router | app.js | NOT DONE — app.js unmodified |
| Hero search "Search" passed to catalogue | home.js, app.js | NOT DONE |
| Bookings filter tabs wired | bookings.js | NOT DONE |

---

## 4. Current UI Status

### Home Dashboard
- Hero section: dark ink background, hero_car.jpg at 50% opacity, left-to-right gradient overlay, Playfair Display headline with copper italic accent.
- Quick-search bar: renders Location select, Pickup Date, Return Date inputs, and "Search" button. **The button has no onclick handler — clicking does nothing.**
- Stats grid: 4 stat cards with emoji icons.
- Featured cars: first 3 cars from `window.CARS`.
- Feature strip: 4 trust pillars.

### Catalogue Page
- Filter toolbar: search input, two selects, sort select, "Available Only" toggle, "Reset" button — all rendered.
- **CRITICAL BUG:** The car grid (`#catalogue-grid`) is empty on load because `afterCatalogueRender()` is never called after navigation. The Catalogue page currently shows only the filter toolbar with no cars underneath it.
- Once B1 is fixed, all filtering will work immediately.

### My Bookings Page
- Renders 4 filter tab buttons and 2 mock booking cards.
- Filter tabs have no event listeners attached. Clicking has no effect on the list.
- "Cancel" button shows a placeholder toast only.

### Sidebar
- Fixed 240px, full-height, dark ink (#101820).
- Logo: copper SVG car icon + "EasyDrive" / "Premium Rentals" tagline.
- Nav badge on "My Bookings" shows static "2" (hardcoded in HTML).
- Footer: user avatar, name, email. Shows "Guest User / Not signed in" by default.

### Topbar
- 60px height, white, sticky.
- Left: page title. Right: notification bell + Sign In / Get Started (guest) or avatar pill (logged in).

### Design Palette (verified)
- Background: #F5F3EE (warm cream)
- Cards/panels: #FFFFFF
- Sidebar: #101820 (deep ink)
- Accent: #D97736 (copper)
- Fonts: DM Sans (body), Playfair Display (hero headline only)
- Breakpoints: 1100px, 768px, 480px

---

## 5. Outstanding UI Improvements Needed

1. **[B1 — highest priority]** Catalogue car grid is empty on load — `afterCatalogueRender()` never called.
2. **[B2]** Hero quick-search "Search" button is dead — no onclick, no event listener.
3. **[B3]** Bookings filter tabs rendered but unresponsive — no event listeners.
4. **[B4]** `updateBookingPrice()` in app.js reads carId via fragile regex on the form's onsubmit attribute string (app.js ~line 235).
5. **[B5]** Favourite buttons in home.js `buildCarCard()` have no `id` attribute — `toggleFavourite()` cannot find them.
6. **[B6]** `hero-search-divider` class in home.js has no CSS rule in style.css.
7. Auth state lost on page refresh — no localStorage/sessionStorage persistence.
8. Profile view shows entirely hardcoded mock data.
9. `dismissToast()` references keyframe `toastSlide` which does not exist — only `toastIn` is defined. Dismiss animation may silently fail.

---

## 6. Current Development Phase

| Phase | Description | Actual Status |
|-------|-------------|---------------|
| 1 | Frontend UI foundation | Complete |
| 2 | Catalogue filters, search, booking form | Partially started — logic in catalogue.js, not connected in app.js |
| 3 | Express server, MongoDB, Mongoose models | Not started |
| 4 | Auth — registration, login, JWT | Not started |
| 5 | Booking engine and availability | Not started |
| 6 | Frontend–backend integration | Not started |
| 7 | Responsiveness, security, deployment | Not started |

### Phase 2 work completed in this session
`catalogue.js` was fully rewritten (81 → 284 lines) to add live filtering, sorting, favourites, empty state, XSS safety, and `afterCatalogueRender()`. The remaining Phase 2 tasks (B1, B2, B3) require changes to `app.js`, `home.js`, and `bookings.js` respectively.

---

## 7. Existing Functionality (verified by reading source)

### Navigation
- `app.navigate(viewName)` updates sidebar active state, topbar title, and re-renders `#page-content`.
- Sidebar uses event delegation on `#sidebar-nav`. Clicking a `.nav-item` triggers navigation.
- Keyboard `Escape` closes auth modal, booking modal, and mobile sidebar.

### Authentication (mock only — no backend)
- `handleLogin()` validates email format and password length (min 6), then creates a mock user object.
- `handleRegister()` validates 5 fields, then creates a mock user object using the entered name.
- Mock user is stored in the `app` IIFE closure (`currentUser`). State is lost on browser refresh.
- No real credentials are checked. Any syntactically valid email and 6+ character password succeeds.

### Booking modal
- Requires login. Shows toast + opens auth modal if guest.
- Date inputs enforce `min = today`. Both dates call `updateBookingPrice()` on change.
- Price breakdown: days × pricePerDay + 5% service fee = total.
- Submit: 800ms timeout → success toast + navigate to bookings. No data is stored.

### Catalogue filtering (written, not yet connected)
- `_catFilter` module-level object holds current filter state.
- `applyCatalogueFilters()` filters `window.CARS` and re-renders `#catalogue-grid` innerHTML.
- Favourites stored in `localStorage` key `ed_favourites` as a JSON array of car IDs.

### Toast system
- `showToast(type, title, message, duration)` — types: success, error, warning, info.
- Auto-dismisses after `duration` ms (default 4000).
- `dismissToast(id)` plays reverse animation (may silently fail — see B9 above).

### Mock data (data.js)
- 6 cars: Audi A8 (car-001, ₹8,500), Mercedes GLE (car-002, ₹9,200), VW Golf GTI (car-003, ₹2,800), Mercedes-AMG GT-R (car-004, ₹15,000), BMW 5 Series (car-005, ₹7,200), VW Sharan MPV (car-006, ₹4,500, available: false).
- 10 pickup locations across Indian cities and airports.
- 2 mock bookings: 1 upcoming, 1 completed.

---

## 8. Known Bugs and Risks

### Confirmed code bugs

| ID | Severity | File | Line(s) | Description |
|----|----------|------|---------|-------------|
| B1 | High | app.js | 40–52 | `navigate()` never calls `afterCatalogueRender()` after rendering catalogue — car grid stays empty, no filter events attached. |
| B2 | High | home.js | 52–54 | Hero "Search" button (`#qs-search-btn`) has no onclick and no event listener anywhere. |
| B3 | Medium | bookings.js | 46–51 | Bookings filter tabs rendered with no event listeners — clicking has no effect. |
| B4 | Medium | app.js | ~234–237 | CarId extracted from form `onsubmit` attribute via regex in `updateBookingPrice()`. Fragile coupling. |
| B5 | Medium | home.js | ~145 | Favourite button in `buildCarCard()` has no `id` attribute. `toggleFavourite()` cannot find it. |
| B6 | Low | home.js | 42 | `<div class="hero-search-divider">` rendered but no CSS rule for `.hero-search-divider` exists. |
| B7 | Low | auth.js | 76–80 | Mock login always succeeds if format is valid — any password works. |
| B8 | Low | app.js | ~508–522 | Profile data (bookings count, loyalty points, member since, preferred location) all hardcoded. |
| B9 | Low | app.js | 407 | `dismissToast()` sets animation to `toastSlide 0.3s ease reverse` but keyframe `toastSlide` is not defined — only `toastIn` is. Toast removal still works (setTimeout removes element) but animation does not play. |

### Untested features
- Mobile responsive layout at 768px and 480px
- Mobile hamburger sidebar open/close and overlay tap-to-close
- Keyboard tab order and focus management throughout the app
- Auth modal backdrop click to close
- Booking modal backdrop click to close
- Price breakdown accuracy across month boundaries
- Google Fonts loading fallback when offline

### Placeholder / mock data risks
- No booking is ever stored. Confirmation references (ED-xxxx) are generated client-side, not persisted.
- Auth creates a mock user object in memory only. No account exists anywhere.
- Car data is entirely static. There is no admin or CMS interface.

---

## 9. Environment Setup

### Running the frontend right now (no server required)
```bash
# Option 1: Open directly
# Windows: double-click frontend/index.html, or:
start frontend/index.html

# Option 2: local static server (recommended — avoids file:// edge cases)
npx serve frontend
# Serves at http://localhost:3000

# Option 3: VS Code Live Server extension
# Right-click frontend/index.html > "Open with Live Server"
```

There is no `npm install`, no build step, and no `package.json` yet. The frontend is plain HTML/CSS/JS.

### Prerequisites (frontend only)
- Any modern browser (Chrome 110+, Firefox 110+, Edge 110+, Safari 16+)
- Internet connection (for Google Fonts — DM Sans and Playfair Display)

### Backend setup (when Phase 3 begins — do not do this yet)
```bash
cd backend
cp .env.example .env
# Fill in .env with real values — see variable list below
npm init -y
npm install express mongoose dotenv cors express-rate-limit jsonwebtoken bcryptjs
node server.js
```

### Environment variables required (from .env.example — no real secrets listed)
| Variable | Purpose | Safe example value |
|----------|---------|-------------------|
| `PORT` | Express listen port | `5000` |
| `NODE_ENV` | Environment flag | `development` |
| `MONGODB_URI` | MongoDB connection | `mongodb://localhost:27017/easydrive` |
| `JWT_SECRET` | JWT signing key | *generate a long random string — never commit* |
| `JWT_EXPIRES_IN` | Token lifetime | `7d` |
| `ALLOWED_ORIGINS` | CORS whitelist | `http://localhost:3000,http://127.0.0.1:5500` |
| `RATE_LIMIT_WINDOW_MS` | Rate window in ms | `900000` |
| `RATE_LIMIT_MAX` | Max requests/window | `100` |

> The `.gitignore` already excludes `.env`. Never commit a file with real credentials.

---

## 10. Next Steps (Recommended Order)

### Step 1 — Fix B1: wire `afterCatalogueRender()` into `app.js navigate()`
This is the single most important immediate change. In `app.js`, inside `navigate()`, after `contentEl.innerHTML = VIEW_RENDERERS[viewName]()` (line 43), add:
```js
if (viewName === 'catalogue' && typeof window.afterCatalogueRender === 'function') {
  window.afterCatalogueRender();
}
```
Effect: Catalogue page immediately shows all 6 cars and all filters become live.

### Step 2 — Fix B2: wire hero quick-search into catalogue navigation
In `home.js`, add an `afterHomeRender()` function that attaches a click listener to `#qs-search-btn`. When clicked, read `#qs-location` value, call `window.setCatalogueFilter({ search: '' })` with appropriate state, then call `app.navigate('catalogue')`. Requires Step 1 to be done first. Register the call in `app.js` navigate() similarly.

### Step 3 — Fix B3: wire bookings filter tabs
In `bookings.js`, add an `afterBookingsRender()` function. Attach click listeners to the 4 filter buttons. Each listener: (a) sets active class on clicked button, (b) shows/hides `.booking-card` elements by their `data-status` attribute. Register in `app.js` navigate().

### Step 4 — Fix B5: add ID to favourite button in `buildCarCard()`
In `home.js`, change the car-card-favourite button to include `id="fav-btn-${car.id}"` so `toggleFavourite()` can find it.

### Step 5 — Fix B4 and B9: clean up fragile carId reading and missing animation keyframe
- In `app.js`: add `data-car-id="${carId}"` to the booking form element, and read it via `form.dataset.carId` instead of regex.
- In `style.css`: add `@keyframes toastSlide` or change the dismiss animation name to `toastIn`.

### Step 6 — Phase 3: scaffold backend
After all Phase 2 fixes are verified in browser:
1. Create `backend/package.json` via `npm init`
2. Install dependencies (express, mongoose, dotenv, cors, jsonwebtoken, bcryptjs, express-rate-limit)
3. Create `backend/server.js`, `backend/src/config/db.js`
4. Create Mongoose models: User, Car, Booking
5. Create route files: auth, cars, bookings
6. Seed Car collection with the 6 vehicles from `data.js`

### Steps 7–9 — Phases 4, 5, 6
- Phase 4: replace mock `handleLogin`/`handleRegister` with real fetch() calls to backend; store JWT in localStorage
- Phase 5: implement real booking creation, availability checking, and cancellation
- Phase 6: replace `window.CARS` and `window.MOCK_BOOKINGS` with API fetch calls; handle token expiry

---

## 11. Testing Checklist

Open `frontend/index.html` in a browser (or use `npx serve frontend`). Test each item manually.

### Navigation
- [ ] Clicking each sidebar item renders the correct view and updates the topbar title
- [ ] Active nav item has a copper left-border highlight
- [ ] Pressing Escape closes the auth modal when open
- [ ] Pressing Escape closes the booking modal when open

### Home view
- [ ] Hero background image (hero_car.jpg) loads
- [ ] Headline uses a serif font (Playfair Display) — requires internet for Google Fonts
- [ ] "Browse Cars" and "View Fleet" buttons navigate to Catalogue
- [ ] Quick-search "Search" button navigates to Catalogue (REQUIRES Step 2 fix to pass filter values)

### Catalogue view (requires Step 1 fix — afterCatalogueRender wired)
- [ ] All 6 cars appear on first load
- [ ] Typing in search box filters the grid in real time
- [ ] "Hatchback" category shows only VW Golf GTI
- [ ] "Manual" transmission shows only VW Golf GTI
- [ ] "Available Only" hides VW Sharan MPV (available: false)
- [ ] "Price: Low to High" puts VW Golf GTI first (Rs 2,800/day)
- [ ] "Price: High to Low" puts Mercedes-AMG GT-R first (Rs 15,000/day)
- [ ] "Top Rated" puts Audi A8 first (rating 4.9)
- [ ] "Reset" clears all filters and shows all 6 cars
- [ ] Searching with no results shows the empty state with "Clear Filters" button
- [ ] Clicking heart on a car toggles filled/empty and persists on navigation away and back

### Booking modal
- [ ] Clicking "Book Now" when not logged in shows warning toast and opens auth modal
- [ ] After login, clicking "Book Now" opens modal with correct car name and image
- [ ] Location dropdown shows 10 options
- [ ] Setting return date before pickup date shows validation error
- [ ] Valid dates show price breakdown (days × rate, 5% fee, total)
- [ ] Clicking "Confirm Booking" shows success toast and navigates to My Bookings
- [ ] Clicking outside the modal closes it

### My Bookings
- [ ] 2 booking cards visible (Audi A8 upcoming, VW Golf GTI completed)
- [ ] "Cancel" button visible only on the upcoming booking
- [ ] Clicking Cancel shows a placeholder toast
- [ ] Filter tabs show/hide bookings by status (REQUIRES Step 3 fix)

### Auth modal
- [ ] "Sign In" button opens modal on login tab
- [ ] "Get Started" button opens modal on register tab
- [ ] Login: invalid email shows inline error
- [ ] Login: password shorter than 6 chars shows inline error
- [ ] Valid login closes modal and updates topbar + sidebar with user name
- [ ] Register: all 5 fields validated, password confirmation checked
- [ ] Valid registration shows welcome toast with entered name
- [ ] Password eye toggle works on both password fields
- [ ] Clicking backdrop closes the modal

### Mobile layout (untested — verify manually at 768px)
- [ ] Sidebar hidden at <=768px; hamburger button visible top-left
- [ ] Tapping hamburger slides in sidebar with dark overlay
- [ ] Tapping overlay closes sidebar
- [ ] Hero search bar stacks vertically on mobile

---

## 12. Handoff Instructions for Incoming AI Agent

### First actions — in this exact order
1. **Read this entire HANDOFF.md** before touching any file.
2. **Run `list_dir` on the project root** to confirm the file tree matches Section 2.
3. **Read each JS file** (`app.js`, `catalogue.js`, `home.js`, `bookings.js`, `auth.js`, `data.js`) to understand the current exact state. Do not assume the code matches anything written in a previous chat.
4. **Do not rebuild, redesign, or overwrite working features.** Phase 1 is complete and functional (except for the known bugs listed in Section 8).
5. **Do not add a frontend framework** (React, Vue, Next.js, Tailwind CSS) unless the project owner explicitly requests it.

### Critical integration rules before editing
- `app.js navigate()` is the only place that renders views. Any view that needs post-render setup must add a call inside `navigate()` after line 43 (where `innerHTML` is set).
- Script load order in `index.html` must remain: data → home → catalogue → bookings → auth → app. `app.js` must stay last because it depends on all view renderers existing in the global scope.
- CSS has both new token names (`--copper`, `--sp-4`) and legacy aliases (`--clr-accent`, `--space-4`). Do not remove either set — inline styles in JS files use both.
- `window.CARS` is the single source of truth for car data in Phases 1–5. All views and modals look up cars by ID from this array.

### Design rules to preserve
- Colour palette: Ink `#101820` / Cream `#F5F3EE` / Copper `#D97736`. Introduce no new brand colours without explicit approval.
- Do NOT add neon colours, heavy drop shadows, gradient buttons, or large border-radius values. This is an editorial-luxury design.
- Typography: DM Sans for all body text. Playfair Display for the hero headline (`hero-title`) ONLY.
- Use CSS custom properties for all spacing in new rules (`--sp-4`, `--sp-6`, etc.) — not raw pixel values.

### Recommended first code edit
Fix Bug B1. In `app.js`, inside the `navigate()` function, immediately after `contentEl.innerHTML = VIEW_RENDERERS[viewName]();` (line 43), add:
```js
if (viewName === 'catalogue' && typeof window.afterCatalogueRender === 'function') {
  window.afterCatalogueRender();
}
```
This single one-line addition makes the entire Catalogue page functional — car grid appears and all 5 filter controls become live.

### What NOT to do
- Do not delete `data.js` — it is the data layer for Phases 1–5.
- Do not rename CSS token variables — both naming conventions are actively used.
- Do not add `<script type="module">` — the project uses classic scripts with global window scope by design.
- Do not commit a real `.env` file.
- Do not claim a feature works without verifying it by reading the code end-to-end or testing it in a browser.
- Do not create a new `package.json` in the project root — backend dependencies belong in `backend/package.json` only.

---

*File location: `MAIN/HANDOFF.md` — project root directory*  
*No application code was modified during the creation of this document.*
