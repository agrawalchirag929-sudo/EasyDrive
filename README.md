# EasyDrive — Premium Car Rental Booking Platform

A full-stack car rental web application built with HTML5, CSS3, Vanilla JavaScript, Node.js, Express, and MongoDB.

## Project Structure

```
MAIN/
├── frontend/               # Static frontend (HTML, CSS, JS)
│   ├── index.html          # Single-page app entry point
│   ├── assets/
│   │   ├── css/
│   │   │   └── style.css   # Global design system & component styles
│   │   ├── js/
│   │   │   ├── app.js      # SPA router & init
│   │   │   ├── data.js     # Mock car data
│   │   │   ├── home.js     # Home dashboard view
│   │   │   ├── catalogue.js # Car catalogue view
│   │   │   ├── bookings.js # My bookings view
│   │   │   └── auth.js     # Auth modal logic
│   │   └── images/         # Car & UI images
├── backend/                # Node.js / Express API (Phase 3+)
│   ├── src/
│   │   ├── config/         # DB & env config
│   │   ├── controllers/    # Route controllers
│   │   ├── middleware/     # Auth, error handling
│   │   ├── models/         # Mongoose models
│   │   └── routes/         # Express routers
│   ├── server.js
│   ├── package.json
│   └── .env.example
└── README.md
```

## Phase Status

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Frontend UI foundation & layout | ✅ Complete |
| 2 | Catalogue, search, filters & booking form | 🔜 Next |
| 3 | Express server, MongoDB, models | ⏳ Pending |
| 4 | Auth — registration, login, JWT | ⏳ Pending |
| 5 | Booking engine & availability | ⏳ Pending |
| 6 | Frontend–backend integration | ⏳ Pending |
| 7 | Responsiveness, security, deployment | ⏳ Pending |

## Running Phase 1 (Frontend Only)

Simply open `frontend/index.html` in any modern browser. No server required for Phase 1.

```bash
# Or use VS Code Live Server / any static server:
npx serve frontend
```
