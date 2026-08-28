# PapperFit — Gym & Training App

**Track your workouts, plan routines and visualize your training volume week by week.**

PapperFit is a training companion app for gym-goers: build routines organized in blocks, walk through a session step by step, log every set (reps + weight) and get meaningful feedback with an estimated 1RM and a 2D muscle map that shows your weekly training volume.

---

## Overview

This repository is a **presentation copy** of the project (the working code lives in a private repository). Its purpose is to show the product concept, features and technical design.

- **Type:** Responsive web application (SPA)
- **Frontend:** React 18 · TypeScript · Vite · Tailwind CSS
- **Backend as a Service:** Supabase (PostgreSQL) 
- **Auth:** Email + password with OTP verification, and Google OAuth
- **Deploy:** Vercel

---

## Problem it solves

Most gym-goers train without records: they don't know what they did last week, how much they lifted, or whether they are actually progressing.

PapperFit solves this by making the whole cycle digital:

1. **Plan** — define routines with exercises and target sets/reps.
2. **Execute** — log every set with real reps and weight while training.
3. **Measure** — see volume, personal bests and estimated 1RM, plus a visual breakdown of volume per muscle group.

---

## Features

### Routines & planning
- Create and delete routines (name + description).
- Organize each routine into **blocks** (e.g. "Chest & Triceps").
- Add exercises from the exercise library to a block with **target sets, reps and weight** (defaults: 3 × 10 @ 0 kg).
- Adjust targets inline from the routine detail screen.

### Live workout session
- Start a session from a routine; all blocks and exercises are flattened into a **step-by-step flow**.
- Each step shows the block, exercise, set number and rep goal; exercises with attached media are displayed as a full-screen background.
- Log **reps and weight** per set; progress bar + animated transitions between steps.
- On finish, the session is marked as completed and the **total volume** (Σ reps × weight) is stored.

### Exercise library
- Catalog of reusable exercises with muscles targeted, movement pattern and optional image/video.
- Per-exercise aggregated stats: total sets, max weight and total volume (computed from all logged sets).

### Progress & stats
- **Estimated 1RM** using the Epley formula `weight × (1 + reps / 30)`, with a %-based rep/weight table (100/90/80/70).
- **Training history** per exercise grouped by session (sets, max weight, volume, set-by-set detail).
- **2D muscle map** (SVG): weekly volume per muscle group with a tier scale (S/A/B/C/D/F).

### Authentication
- Register with email + 6-digit OTP verification and resend support.
- Login with email/password or **Continue with Google**.
- Password reset request and token-based reset.
- Protected routes + safe redirect handling after login.

---

## Tech stack

| Layer    | Technology |
|----------|------------|
| UI       | React 18, TypeScript, Tailwind CSS, shadcn/ui (Radix primitives), Framer Motion |
| Data/Auth| Supabase via Base44 SDK (entities + auth), TanStack React Query |
| Routing  | React Router |
| Build    | Vite |
| Deploy   | Vercel |

---

## Database schema

All entities are accessed through the Base44 SDK (`base44.entities.*`).

```
Routine 1───* Block 1───* RoutineExercise *───1 Exercise
   │                                                  │
   1                                                  1
   │                                                  │
   * WorkoutSession 1───* SetLog ──────────────────────*
```

| Table            | Columns (key fields) |
|------------------|----------------------|
| `Routine`        | `id`, `name`, `description`, `created_date` |
| `Block`          | `id`, `routine_id` → Routine, `name`, `order` |
| `RoutineExercise`| `id`, `routine_id`, `block_id`, `exercise_id` → Exercise, `exercise_name`, `target_sets`, `target_reps`, `target_weight`, `order` |
| `Exercise`       | `id`, `name`, `muscles`, `movement_pattern`, `media_url` |
| `WorkoutSession` | `id`, `routine_id` → Routine, `routine_name`, `started_at`, `finished_at`, `status` (`in_progress`/`completed`), `total_volume` |
| `SetLog`         | `id`, `session_id` → WorkoutSession, `exercise_id`, `exercise_name`, `set_number`, `reps`, `weight`, `created_date` |

---

## Key data flows

1. **Create a routine** → `INSERT Routine { name, description }` → add blocks → add exercises with targets (`RoutineExercise`).
2. **Start a session** → `INSERT WorkoutSession { status: "in_progress" }`; the UI builds an ordered step list from the routine's blocks/exercises.
3. **Log a set** → `INSERT SetLog { session_id, exercise_id, exercise_name, set_number, reps, weight }`.
4. **Finish a session** → aggregate all `SetLog` of the session, compute `total_volume = Σ(reps × weight)` and `UPDATE WorkoutSession { status: "completed", finished_at, total_volume }`.
5. **1RM & history** → computed on the fly from `SetLog` (never stored).
6. **Weekly muscle map** → last-7-days `SetLog` are mapped to muscle groups (via each exercise's `muscles` field) and counted as sets per group; the SVG map colors each group by its tier.

---

## Screenshots

Add captures in a `screenshots/` folder and reference them here. Representative screenshots for this project:

- `screenshots/routines.png` — Routines list (main screen)
- `screenshots/routine-detail.png` — Routine detail with blocks and target sets/reps
- `screenshots/session.png` — Live workout session (logging one set)
- `screenshots/exercises.png` — Exercise library
- `screenshots/muscle-map.png` — 2D muscle map with weekly volume (distinctive feature)
- `screenshots/login.png` — Login / registration

Example:

```
![Routines](screenshots/routines.png)
```

---

## Getting started (setup notes)

1. `npm install`
2. Create a `.env` with the project credentials (Supabase / Base44):
   ```
   VITE_SUPABASE_URL=...
   VITE_SUPABASE_ANON_KEY=...
   ```
3. `npm run dev` → open the local URL.

> Configuration values are intentionally omitted from this repository.

---

## Notes

- Table and column names reflect the real schema used by the application.
- This README-only repository was created to showcase the project in a portfolio. The full source code is kept private.
