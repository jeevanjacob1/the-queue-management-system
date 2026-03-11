# Queue Management System — Implementation Plan

A model project by **Jeevan A Jacob** — a full-stack clinic/office queue management system to eliminate long waiting lines. Users join queues online and track their position in real time, while admins manage the queue via a dashboard.

---

## Proposed Changes

### Backend

#### [NEW] `backend/package.json`
Initialize Node.js project; dependencies: `express`, `better-sqlite3`, `bcryptjs`, `jsonwebtoken`, `cors`, `ws`, `dotenv`, `uuid`

#### [NEW] `backend/server.js`
Express server with:
- JWT middleware
- WebSocket server (ws) for real-time queue updates
- Route registration for auth, queues, clinics, admin, users

#### [NEW] `backend/database/schema.sql`
SQLite schema:
- `users` (id, name, email, password_hash, role, created_at)
- `clinics` (id, name, description, avg_service_time, status, created_at)
- `queue_tickets` (id, user_id, clinic_id, ticket_number, status, created_at, called_at, completed_at)

#### [NEW] `backend/database/db.js`
SQLite connection via `better-sqlite3`, creates tables if not exists, seeds default admin.

#### [NEW] `backend/middleware/auth.js`
JWT verification middleware; sets `req.user` with role.

#### [NEW] `backend/routes/auth.js`
- `POST /auth/register` — validates, hashes password, inserts user, returns JWT
- `POST /auth/login` — verifies credentials, returns JWT

#### [NEW] `backend/routes/queues.js`
- `GET /queues` — with optional `?clinic_id=` filter
- `POST /queues` — join queue, auto-generates ticket number
- `PUT /queues/:id` — update status
- `DELETE /queues/:id` — cancel/delete ticket

#### [NEW] `backend/routes/clinics.js`
- `GET /clinics`
- `POST /clinics` (admin only)
- `PUT /clinics/:id` (admin only)
- `DELETE /clinics/:id` (admin only)

#### [NEW] `backend/routes/admin.js`
- `POST /admin/queue/next` — calls next waiting ticket
- `POST /admin/queue/reset` — resets all tickets for a clinic

#### [NEW] `backend/routes/users.js`
- `GET /users/profile` — get own profile
- `PUT /users/profile` — update profile
- `GET /users/history` — own queue history

---

### Frontend

#### [NEW] `frontend/index.html`
Landing page: hero section, feature highlights, "Join Queue" CTA, navbar.

#### [NEW] `frontend/login.html`
Combined login form with tab to switch to Register.

#### [NEW] `frontend/user-dashboard.html`
- Clinic selector
- Join queue button
- Active ticket card (position, estimated wait time, ticket number)
- Queue history table

#### [NEW] `frontend/admin-dashboard.html`
- Live queue table (waiting, called, completed)
- Call Next / Skip / Complete / Cancel buttons
- Reset queue button
- Clinic management (add/edit/delete)
- Analytics summary (total served today, avg wait time)

#### [NEW] `frontend/display.html`
Public live display screen — shows currently called ticket numbers via WebSocket.

#### [NEW] `frontend/css/style.css`
Full design system: dark mode, glassmorphism cards, gradient accents, responsive grid.

#### [NEW] `frontend/js/config.js`
`API_URL` constant pointing to backend.

#### [NEW] `frontend/js/api.js`
Centralized fetch wrapper with JWT auth header injection.

#### [NEW] `frontend/js/auth.js`
Login/register form handlers, JWT storage in localStorage.

#### [NEW] `frontend/js/user-dashboard.js`
- Load clinics, join queue
- WebSocket for live position updates
- Queue history fetch

#### [NEW] `frontend/js/admin-dashboard.js`
- WebSocket for live queue list
- Call next, skip, complete, cancel, reset
- Clinic CRUD UI

#### [NEW] `frontend/js/display.js`
WebSocket connection, shows called ticket numbers on big display.

---

### Documentation

#### [NEW] `api-docs/README.md`
Full API reference: all endpoints, request/response schemas, auth headers.

#### [NEW] `README.md`
Project overview, setup instructions, how to run locally, deployment guide.

---

## Verification Plan

### Automated (Manual via curl / browser)

1. **Start backend**:
   ```
   cd backend && npm install && node server.js
   ```
   Server should start on `http://localhost:3000`

2. **Register a user**:
   ```
   curl -X POST http://localhost:3000/auth/register \
     -H "Content-Type: application/json" \
     -d '{"name":"Test User","email":"test@test.com","password":"test123"}'
   ```
   Expected: `{ token, user }` with role=user

3. **Login**:
   ```
   curl -X POST http://localhost:3000/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email":"test@test.com","password":"test123"}'
   ```
   Expected: JWT token

4. **Get clinics**:
   ```
   curl http://localhost:3000/clinics
   ```
   Expected: array of clinics (seeded on first run)

5. **Join a queue** (use token from login):
   ```
   curl -X POST http://localhost:3000/queues \
     -H "Authorization: Bearer <token>" \
     -H "Content-Type: application/json" \
     -d '{"clinic_id": 1}'
   ```
   Expected: ticket object with ticket_number and status=waiting

### Browser Tests
- Open `frontend/index.html` in browser → landing page visible
- Click "Join Queue" → redirects to login
- Register → redirected to user dashboard
- Join queue → ticket appears with position and estimated wait
- Admin login (seeded admin: `admin@clinic.com` / `admin123`) → admin dashboard visible
- Click "Call Next" → status changes, WebSocket pushes update to user dashboard
- Open `frontend/display.html` → shows live called ticket

### Manual QA Checklist
- [ ] User can join multiple different clinic queues (one active at a time per clinic)
- [ ] Admin can call next, skip, complete, cancel
- [ ] Admin can reset daily queue
- [ ] Admin can add/edit/delete clinic
- [ ] WebSocket updates propagate to all open browser tabs in < 1 second
