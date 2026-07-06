I have a full-stack project. It has exactly two folders at the top level, 
named exactly:

- backend
- frontend

Inside "backend", package.json already exists with express, cors, and 
better-sqlite3 already installed. Do not create package.json. Do not run npm 
install. Just write the code file.

Inside "frontend", a React + Vite project is already scaffolded. 
frontend/src/main.jsx already exists and contains:
  import App from './App.jsx'
  import './index.css'
Do NOT generate main.jsx, index.css, package.json, vite.config.js, or 
index.html. 

Generate EXACTLY 5 files, and only these 5 files, at these exact paths:

1. backend/index.js
2. frontend/src/App.jsx
3. frontend/src/App.css
4. postman/Habit-Tracker-API.postman_collection.json
5. postman/Habit-Tracker.postman_environment.json

============================================================
FILE 1: backend/index.js
============================================================

Line 1 must be this exact comment: // backend/index.js

1. Use require() syntax (CommonJS), not import/export. Import express, cors, 
   and better-sqlite3.

2. Create the Express app. Call app.use(cors()). Call app.use(express.json()).

3. Connect to a database file named exactly "data.db" using: 
   const db = new Database('data.db');

4. Create two tables if they do not exist:

   TABLE "habits":
   - id: INTEGER, PRIMARY KEY, AUTOINCREMENT
   - name: TEXT, NOT NULL (e.g. "Drink 2L water")
   - created_at: TEXT, NOT NULL (ISO timestamp, set when the habit is created)

   TABLE "checkins":
   - id: INTEGER, PRIMARY KEY, AUTOINCREMENT
   - habit_id: INTEGER, NOT NULL (references the habits table's id column — 
     do not enforce a real SQL foreign key constraint, just store the number)
   - date: TEXT, NOT NULL (the date checked in, format "YYYY-MM-DD")
   - checked_at: TEXT, NOT NULL (exact ISO timestamp of when the check-in 
     was recorded)
   There must be a UNIQUE constraint across (habit_id, date) together, so the 
   same habit cannot be checked in twice on the same calendar date. Add this 
   using: UNIQUE(habit_id, date) inside the CREATE TABLE statement.

5. Write a helper function named calculateStreak(habitId) that:
   - Queries all rows from checkins where habit_id = habitId, ordered by 
     date descending.
   - Starting from today's date (calculated fresh each time the function 
     runs, as "YYYY-MM-DD"), counts how many consecutive calendar days in a 
     row (going backwards from today) have a check-in. 
   - If there is no check-in for today AND no check-in for yesterday, the 
     streak is 0.
   - If today has no check-in but yesterday does, still count the streak 
     starting from yesterday backwards (so a user who hasn't checked in yet 
     today doesn't lose their streak until the day is over).
   - Return the final count as a plain integer.
   - Add a comment above this function explaining this exact logic in your 
     own words.

6. Create exactly these 6 routes, in this exact order:

   ROUTE A — POST /habits
   - Purpose: create a new habit.
   - Read "name" from req.body. If missing or an empty string after 
     trimming, respond 400 with { "error": "name is required" }. Stop.
   - Insert into habits with created_at = new Date().toISOString().
   - Respond 201 with the full new habit row as JSON, plus an extra field 
     "streak": 0 (since a brand new habit has no check-ins yet).

   ROUTE B — GET /habits
   - Purpose: list all habits along with each one's current streak.
   - Select all rows from habits, ordered by created_at ascending (oldest 
     first).
   - For each habit, call calculateStreak(habit.id) and attach the result 
     as a "streak" field on that habit object.
   - Respond 200 with a JSON array of habit objects (each including id, 
     name, created_at, streak).

   ROUTE C — POST /habits/:id/checkin
   - Purpose: mark a habit as done for a specific date (defaults to today).
   - Read "date" from req.body. If not provided, use today's date 
     (calculated as "YYYY-MM-DD").
   - Check that the habit with this id exists in the habits table. If not, 
     respond 404 with { "error": "Habit not found" }. Stop.
   - Try to insert a new row into checkins with habit_id = the id from the 
     URL, date = the resolved date, checked_at = new Date().toISOString().
   - If the insert fails because of the UNIQUE constraint (habit already 
     checked in for that date), catch that specific error and respond 409 
     with { "error": "Already checked in for this date" }. Do not let the 
     server crash.
   - If the insert succeeds, respond 201 with the new checkin row AND an 
     updated "streak" field (call calculateStreak(id) again after inserting).

   ROUTE D — GET /habits/:id/checkins
   - Purpose: return all check-in dates for one habit, for rendering a 
     calendar/history view.
   - Confirm the habit exists; if not, respond 404 with 
     { "error": "Habit not found" }.
   - Select all rows from checkins where habit_id matches, ordered by date 
     descending.
   - Respond 200 with a JSON array of just the date strings, e.g. 
     ["2026-07-06", "2026-07-05", "2026-07-03"]. Do not include the full row 
     objects, just an array of date strings.

   ROUTE E — DELETE /habits/:id/checkin/:date
   - Purpose: undo a check-in for a specific date (in case the user 
     misclicked).
   - Delete the row from checkins where habit_id matches the :id URL param 
     AND date matches the :date URL param exactly.
   - Respond 200 with { "message": "Checkin removed" }.

   ROUTE F — DELETE /habits/:id
   - Purpose: delete a habit entirely, along with all of its check-in history.
   - First delete all rows from checkins where habit_id matches.
   - Then delete the row from habits where id matches.
   - Respond 200 with { "message": "Habit <id> and its checkins deleted" } 
     where <id> is the actual id value.

7. At the bottom, start the server on port 5000 (hardcoded, not from an 
   environment variable). console.log exactly: 
   "Server running on http://localhost:5000"

8. Add a one-sentence comment above every route and above both table 
   definitions.

============================================================
FILE 2: frontend/src/App.jsx
============================================================

Line 1 must be this exact comment: // frontend/src/App.jsx

Single functional component named App, default exported at the bottom. 
Import useState and useEffect from 'react'. Import './App.css' at the top. 
Define: const API_URL = 'http://localhost:5000';

Render, top to bottom, in this exact order:

1. <h1>🔥 Habit Tracker</h1>

2. A "New Habit" card:
   - A single text input, placeholder "e.g. Drink 2L water", and an "Add 
     Habit" button next to it (same row, not stacked).
   - Pressing Enter in the input OR clicking the button should submit.
   - If the input is empty or whitespace-only when submitted, do nothing 
     (no error message needed for this one — just silently ignore).
   - On successful submit: send POST to `${API_URL}/habits` with 
     { name: <trimmed input value> }. Clear the input. Re-fetch the full 
     habit list.

3. A list of habit cards, one per habit, each rendered as a separate 
   <div className="habit-card"> containing:
   - The habit's name as a heading (e.g. <h3>).
   - The current streak, shown as: "🔥 <streak> day streak" if streak > 0, 
     or the exact text "No streak yet — check in today!" if streak is 0.
   - A "Check In" button. If the habit has ALREADY been checked in for 
     today (you will need to fetch each habit's checkin dates separately — 
     see step 4 below — to know this), this button must instead show the 
     text "✅ Checked in today" and be disabled (not clickable). Otherwise 
     it shows "Check In" and is clickable.
   - Clicking "Check In" sends POST to 
     `${API_URL}/habits/<habit id>/checkin` with an empty JSON body {}. 
     After it succeeds, re-fetch that habit's checkin dates AND the full 
     habit list (to update the streak number).
   - A small calendar-style history: render the last 7 calendar days 
     (today and the 6 days before it) as 7 small square boxes in a row. 
     For each of the 7 days, if that date exists in the habit's checkin 
     dates, the box should visually indicate "done" (see CSS file for exact 
     styling); if not, it should look empty/inactive. Each box should show 
     the day-of-month number inside it (e.g. "6", "5", "4"...).
   - A "Delete Habit" button (small, at the bottom-right corner of the 
     card). Clicking it sends DELETE to `${API_URL}/habits/<habit id>` 
     with no confirmation dialog needed. After success, re-fetch the full 
     habit list.

4. Data fetching logic:
   - On mount, fetch GET `${API_URL}/habits` and store the array in state 
     (call this state "habits").
   - For EACH habit in that array, also fetch GET 
     `${API_URL}/habits/<id>/checkins` (the array of date strings) and 
     store these in a SEPARATE state object keyed by habit id, e.g. 
     { 3: ["2026-07-06", "2026-07-04"], 7: ["2026-07-05"] }. Call this 
     state "checkinsByHabit".
   - Write one function, refreshAll(), that does both of the above (fetch 
     habits, then fetch each habit's checkins) and updates both state 
     variables. Call refreshAll() once on mount, and again after any 
     add/checkin/delete action succeeds — do not write separate one-off 
     fetch logic scattered across handlers; always call refreshAll() to 
     keep it simple and consistent.
   - If there are no habits at all, show the exact text 
     "No habits yet. Add one above to get started!" instead of the habit 
     card list.
   - While the very first load is happening, show the exact text 
     "Loading your habits..." instead of anything else in that section.

5. Wrap every fetch() in try/catch. On error, console.error() the error and 
   leave existing state unchanged (do not crash the page).

============================================================
FILE 3: frontend/src/App.css
============================================================

Line 1 must be this exact comment: /* frontend/src/App.css */

Style these elements using plain CSS only (no frameworks, no CSS-in-JS):
- Page: max-width around 640px, centered with margin: 0 auto, comfortable 
  padding, plain system font.
- .habit-card: white card, light gray 1px border, rounded corners (10px), 
  padding ~18px, margin-bottom ~16px between cards, position: relative (so 
  the delete button can be absolutely positioned in a corner if you choose).
- The streak text: make it stand out — larger font size than normal body 
  text (around 1.1rem), a warm color (e.g. an orange/red tone) when streak 
  > 0.
- The 7-day history boxes: display them in a horizontal row using flexbox 
  or grid, each box roughly 32px by 32px, small rounded corners, centered 
  text showing the day number. "Done" boxes get a solid, saturated 
  background color with white text. "Not done" boxes get a light gray 
  background with gray text. Add a small gap (4-6px) between boxes.
- The "Check In" button: solid accent-colored background, white text, 
  rounded, pointer cursor. When disabled (already checked in), it should 
  look visually muted/grayed out AND change its background to a muted 
  green tone to reinforce "already done" (do not just rely on the disabled 
  attribute's default browser styling).
- The "Delete Habit" button: small, subtle (e.g. gray text, no heavy 
  background), positioned in the bottom-right of the card, red text on hover.
- The "New Habit" input + button row: input takes up remaining width 
  (flex: 1), button sits to its right, both vertically aligned, consistent 
  height between them.

============================================================
FILE 4: postman/Habit-Tracker-API.postman_collection.json
============================================================

This must be valid Postman Collection v2.1 schema JSON, AND it must include 
every one of these fields exactly — collections missing these fields have 
failed to import in the VS Code Postman extension before, so do not omit 
any of them even though the official schema marks some as optional:

- Top-level "info" object must contain: "_postman_id" (any valid-looking 
  UUID string you generate), "name", "description", "schema" (the exact 
  string "https://schema.getpostman.com/js
