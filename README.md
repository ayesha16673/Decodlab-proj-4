# Yummy Restaurant — Full Stack Development Journey (Projects 1–4)

**DecodeLabs Industrial Training Kit — Batch 2026**

This repository contains the final, integrated version of the Yummy
Restaurant web application, built across four milestone projects in the
DecodeLabs Full Stack Development track. Each project had its own official
brief (goal, key requirements, key skills); this document explains what each
one asked for and how this codebase satisfies it.

---

## Project 1 — Responsive Frontend Interface

**Official goal:** Create a responsive frontend interface for a simple web
application.

**Key requirements:**
- Use HTML, CSS, and basic JavaScript
- Responsive layout for different screen sizes
- Clean and user-friendly UI

**Key skills:** Frontend development, responsive design, UI basics

**What was built:** The Yummy Restaurant site — header/nav, hero section,
about, menu tabs, gallery, testimonials, reservation and contact forms,
footer — using plain HTML5, CSS3 (including media queries for
tablet/desktop breakpoints), and vanilla JavaScript for interactive behavior
(mobile nav toggle, image slider, menu category switching, animated
counters). No frameworks were used, per the brief's "master the fundamentals
first" approach. At this stage the site was static: forms did not talk to
any server, and menu content was written directly into the HTML.

---

## Project 2 — Backend API Development

**Official goal:** Develop a simple backend API to handle application
logic.

**Key requirements:**
- Create API endpoints (GET / POST)
- Handle user input and responses
- Validate basic data

**Key skills:** Backend development, server-side logic, API concepts

**What was built:** A backend using Node's built-in `http` module (no
framework yet, per the brief), exposing endpoints for bookings, contact
messages, and the menu. Data was held in in-memory JavaScript arrays/objects
— there was no database at this stage, so data reset whenever the server
restarted. This project introduced the request/response cycle, basic input
validation (e.g. required fields on the booking and contact forms), and the
GET/POST distinction that the frontend would later depend on.

---

## Project 3 — Database Integration

**Official goal:** Connect the backend with a database to store and
retrieve data.

**Key requirements:**
- Design a simple database schema
- Perform basic CRUD operations
- Ensure proper data handling

**Key skills:** Databases, CRUD operations, data storage

**What was built:** The backend was rebuilt on **Express**, connected to
**MongoDB** through **Mongoose**. This is where the application gained:
- Three schemas with validation — `Booking`, `Message`, `MenuItem`
- Full CRUD on bookings (create, read, **update**, delete — Project 2 only
  had create/read/delete)
- `express-mongo-sanitize` middleware, guarding against NoSQL injection on
  every request
- A one-time `seed.js` script that migrated the old hardcoded menu data into
  a real `menu_items` collection

By the end of Project 3, the backend could do everything the brief asked —
but the frontend still hadn't been updated to use any of it for displaying
data. The menu section in `index.html`, in particular, was still the same
static markup from Project 1.

---

## Project 4 — Frontend & Backend Integration (this submission)

**Official goal:** Integrate frontend with backend APIs.

**Key requirements:**
- Send requests from frontend to backend
- Display dynamic data on UI
- Handle basic errors and responses

**Key skills:** API integration, asynchronous requests, full stack flow

**What was already in place:** The booking and contact forms (built in
Project 2, carried through Project 3) already used `fetch()` with
`async/await` and try/catch to send POST requests to the backend — so two
of the three requirements were technically already met before this project
started.

**What was missing, and what this project adds:** The menu section had
never been connected to `GET /api/menu`, despite the backend serving it
since Project 3. This project closes that gap:

```javascript
async function loadMenu() {
  try {
    const response = await fetch(`${API_BASE_URL}/api/menu`);
    if (!response.ok) throw new Error(`HTTP Error! Status: ${response.status}`);

    const result = await response.json();
    if (!result.success) throw new Error(result.message);

    categories.forEach(category => renderMenuCategory(category, result.data[category]));
  } catch (err) {
    // show a retry-able error message instead of a blank page
  } finally {
    // hide the loading indicator either way
  }
}
```

Specifically, `frontend/script.js` and `frontend/index.html` were updated so
that:

1. **Send requests from frontend to backend** — `loadMenu()` fires a
   `GET /api/menu` request as soon as the page loads.
2. **Display dynamic data on UI** — the response (grouped by `starters`,
   `breakfast`, `lunch`, `dinner`) is rendered into the DOM using
   `createElement` and `textContent` (never `innerHTML`, so data from the
   database can't be interpreted as markup), replacing the old hardcoded
   items.
3. **Handle basic errors and responses** — `response.ok` is checked before
   trusting the body, a loading message shows while the request is in
   flight, a visible error message with a **Retry** button appears on
   failure, and a `finally` block guarantees the loading indicator always
   clears.

**Also added — `frontend/admin.html`:** a small, independent dashboard page
that calls `GET /api/bookings` and `GET /api/contact` and renders both as
tables, using the same loading/error/retry pattern as the menu. This makes
the data already being stored (since Project 3) visible without needing
Postman or the Atlas web console. It does not touch or affect `index.html`.

---

## Project Structure

```
backend/
  config/db.js         MongoDB connection setup (Project 3)
  models/               Mongoose schemas — Booking, Message, MenuItem (Project 3)
  routes/                CRUD route handlers for /api/bookings, /api/contact, /api/menu
  data/menu.js           Original static menu data (Project 2), used only for seeding
  seed.js                One-time script to load menu data into MongoDB (Project 3)
  server.js              Express app entry point (Project 3)
  .env.example           Template for required environment variables

frontend/
  index.html             Main site — Project 1 layout, Project 4 dynamic menu
  admin.html              Standalone dashboard for bookings & messages (Project 4)
  script.js               All frontend logic, including the fetch/render code
  style.css, responsive.css   Project 1 styling
  assets/                 Images
```

---

## Running the Project

### Backend
```bash
cd backend
npm install
```

Create a `.env` file (copy `.env.example`) and add your MongoDB connection
string:
```
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@<cluster-url>/yummy_restaurant
```

Seed the menu collection (first time only):
```bash
npm run seed
```

Start the server:
```bash
npm run dev
```

Expected output:
```
✅ MongoDB connected successfully
✅ Yummy Backend API running at http://localhost:5000
```

### Frontend
Open `frontend/index.html` with a live server (e.g. the VS Code "Live
Server" extension) while the backend above is running. To view submitted
bookings and messages, open `frontend/admin.html` the same way.

---

## API Endpoints

| Method | Endpoint            | Introduced In | Description                    |
|--------|----------------------|----------------|---------------------------------|
| GET    | /api/menu            | Project 3      | All menu items, grouped by category |
| GET    | /api/menu/:category  | Project 3      | Items for a single category     |
| POST   | /api/bookings        | Project 2      | Create a booking                |
| GET    | /api/bookings        | Project 2      | List all bookings               |
| GET    | /api/bookings/:id    | Project 2      | Get one booking                 |
| PUT    | /api/bookings/:id    | Project 3      | Update a booking (full CRUD)     |
| DELETE | /api/bookings/:id    | Project 2      | Cancel/delete a booking          |
| POST   | /api/contact         | Project 2      | Submit a contact message         |
| GET    | /api/contact         | Project 2      | List all contact messages        |

---

## Notes for Reviewers

- `.env` is intentionally excluded from this submission for security reasons.
  `.env.example` shows the required format — add your own MongoDB URI to run
  the project locally.
- `node_modules/` is excluded as well; run `npm install` inside `backend/`
  before starting the server.
- Project 4 is listed as an optional milestone in the DecodeLabs brief; the
  core certification requirements were already met by Project 3. This
  submission completes it in full regardless.
