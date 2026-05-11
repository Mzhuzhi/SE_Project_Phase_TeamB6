# Rei Greca — Application Form

> Personal contribution to UniGate Team B6 — Phase III: Software Design

---

## What this component does

This module is the **application form** — the part of UniGate where a logged-in student actually applies to a programme. It is the bridge between the programme catalogue (where the student picks what to apply to) and the database (where the application is stored for the admin staff to review).

**In one sentence:** a three-step form that collects personal and academic information, validates it on both the frontend and the backend, and stores it in the `applications` table linked to the student and the chosen programme.

**Key design choices:**

- **Three-step flow** (personal details → academic history → review) to reduce drop-offs and make a long form feel manageable.
- **Adaptive form** — if the programme is a Master's, an extra Bachelor degree subsection appears in step 2.
- **Double validation** — the frontend validates each step for fast feedback, and the backend validates again because frontend checks can be bypassed.
- **One application per university rule** — a student cannot apply twice to the same university, enforced on both page load and at submission.

---

## Where the code lives

| Layer | File | Purpose |
|-------|------|---------|
| Frontend | `frontend/src/pages/Apply.jsx` | The entire three-step form |
| Backend | `backend/routes/applications.js` | `POST /api/applications` endpoint + server-side validation |
| Database | `backend/models/Application.js` | Sequelize model for the `applications` table |

---
