**CAHSI – GMiS – Student Generative AI Session**
**Activity: Reddit Clone in Flask**
**October 2025**

**Purpose**

Build a tiny Reddit-like site in Flask that supports voting, posting, and hiding
items—first in memory, then persisted to SQLite—while using an LLM as a thought
companion (LPDP), not a code vending machine.

**What you’ll practice**

- Setting up a small Flask project and running it locally.
- Server-side HTML forms and request/response flow (POST → redirect → GET).
- Mutating in-memory state safely and rendering sorted views.
- Incremental feature development with clear acceptance tests.
- Finding and describing bugs in others’ code.
- Adding persistence with sqlite3 and reasoning about data models.
- Using the **LPDP prompts** to clarify, plan, debug, and reflect.

**Tools & setup**

- Work in **pairs** (Driver/Navigator; swap every ~15 minutes).
- Stack: **Python 3.10+**, **Flask**, **uv**, any editor (VS Code recommended).
- Basic HTML forms; no JS required.

**Generative AI policy (read carefully)**

- Use an LLM for **questions, planning, tiny nudges** (≤8 lines if code is essential; explain line‑by‑line).
- **Do not** paste full solutions. You must own the design and tests.
- Keep a **prompt/decision log** (3–5 lines total): one suggestion you kept + evidence, and one you rejected + why.

---

## Part 1: Getting set up (15 minutes)

**Requirements**

1. Install prerequisites

- Python 3.10+
- uv (package manager)

Install uv (macOS/Linux):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2. Download the code

```bash
git clone git@github.com:SamLau95/gmis-flask-reddit.git
cd gmis-flask-reddit
```

3. Create the virtual environment and install dependencies

```bash
uv sync
```

4. Run the app

Run using the Flask CLI:

```bash
uv run flask --app app run --debug
```

Open `http://127.0.0.1:5000/` and confirm you see the dog links.

---

## Part 2: Add features (45 minutes)

You’ll add three features by mutating the `dog_links` variable. All actions
should update state on the server, then **redirect** back to `/` so the page
refreshes with the latest data.

### Feature A: Upvotes and downvotes

**Goal**: Users can upvote or downvote a link. After voting, the page reloads and items are always shown from highest score to lowest.

**Behavior**

- When the user clicks the up arrow or down arrow, the app should update that post’s `score` in `dog_links`.
- After updating, redirect to the homepage.
- On the homepage posts should always be ordered by `score` descending.

### Feature B: Submit a new post

**Goal**: Users can add a new post with `title` and `url`. New posts start with **1 upvote by default**.

**Behavior**

- Add a form to submit a new post.
- Validate minimally: a non-empty `title` and a `url` that starts with `http` is fine.
- Append to `dog_links` with `score = 1`.
- Redirect back to `/`, then sort by `score` descending.

**Acceptance tests**

1. Submitting a valid post adds it to the list and displays score `1`.
2. The post is placed relative to others based on its score (1).
3. Empty title or invalid URL is rejected with a friendly message (no crash).

### Feature C: Hide a post

**Goal**: Users can hide any post. Hidden posts are not shown in the main list; instead they appear at the very bottom of the page under a section titled **“Hidden posts”**.

**Behavior**

- Add a "Hide" control for each post; toggling sets a simple flag on that item in `dog_links` (e.g., `hidden: True`).
- The main feed shows only non-hidden posts, sorted by `score` descending.
- At the bottom, render a separate section for **Hidden posts**.
- Within Hidden posts, you may sort by score descending or leave original order—state your choice.

**Acceptance tests**

1. Hiding a post removes it from the main list immediately and places it under “Hidden posts.”
2. Non-hidden posts remain sorted by `score` descending.
3. Page reload after any action reflects the correct grouping and order.

---

## Part 3: Bug‑finding competition (25 minutes)

1. **Swap projects** with another pair. Sit at their computer.
2. **Find as many bugs as possible** (logic, UI, validation, sorting edge cases).
3. **Document each bug** with steps, expected vs. actual, and a screenshot if helpful.
4. **Share your findings** with the original team. Discuss overlaps and prevention ideas.

---

## Part 4: Feature request — persistence with SQLite (35 minutes)

Right now `dog_links` resets on restart (e.g. if you stop the Flask app and restart it, the dog links will reset back to its original state). Add persistence with `sqlite3` so posts, votes, and hidden state survive restarts.

**Goal**

- On server start, load posts from a SQLite database.
- When users vote/submit/hide, update the database.
- On page load, query posts (grouped into visible/hidden), and render visible posts sorted by score descending.

**Suggested schema (modify as you see fit)**

- `posts(id INTEGER PRIMARY KEY, title TEXT NOT NULL, url TEXT NOT NULL, score INTEGER NOT NULL, hidden INTEGER NOT NULL DEFAULT 0, created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP)`

**Behavior**

- If the DB is empty on first run, **seed** it from the current `dog_links` contents.
- Replace all in‑memory mutations with SQL `INSERT/UPDATE` and read fresh rows before rendering.
- Preserve sorting by `score` descending for the main list; place hidden posts in the bottom section.

**Acceptance tests**

1. After voting/adding/hiding, restart the app; the state is preserved.
2. New posts are stored with score `1` by default.
3. Hidden state persists across restarts.
4. Visible posts always appear sorted by `score` descending.

---

## LPDP: use the LLM as a thought companion (copy/paste)

**Learner Context Card (paste/fill at the start):**

```
Course & level:
Task summary (1–2 lines): Build a server‑rendered Flask app for voting, posting, hiding.
Language/stack: Python 3.10, Flask, sqlite3
Comfort (1–5): decomposition __ / syntax __ / debugging __ / testing __
What I already tried:
Biggest confusion:
Constraints (perf/libs/style): no JS required; server‑side forms; readable code
Allowed help: no full solutions; ≤8 lines/code nudge; prefer questions
Goal for this session (120 min):
```

**Conversation contract (paste after the card):**

Be my thought companion.

1. Ask 3–5 diagnostic questions before ideas.
2. Offer 2–3 options tied to MY constraints.
3. No full code until I have checks; ≤8 lines if code is essential, explain line‑by‑line.
4. After each step, ask for evidence (checks/outputs) before moving on.
5. End each turn with: “Next two choices you could make are: …”

**Short prompts to use during the build:**

- **Clarify:** “Ask 4 questions to remove ambiguity in my vote/submit/hide routes and sorting rules.”
- **Plan options:** “Suggest 2–3 ways to identify posts (index vs. id vs. composite key) and the trade‑offs.”
- **Test‑first:** “Help me list 5 acceptance checks for voting, posting, hiding, and sorting.”
- **Micro‑nudge:** “Give me ≤8 lines of pseudocode for POST‑redirect‑GET flow for voting.”
- **Debug:** “Hiding then voting does something weird—ask 2 isolation questions and suggest 2 tiny probes.”
- **Refactor:** “Offer 2 small refactors to reduce duplication across my vote/hide handlers.”
- **Reflect:** “Interview me with 3 questions so I can explain one decision, one trade‑off, and one thing I’d test next.”

---

## Design clarifications

- **Ordering rule:** Visible posts are sorted by `score` descending; ties can be broken arbitrarily.
- **Hidden posts:** Render in a separate “Hidden posts” section at the bottom. They do not appear in the main list at the top of the homepage.
- **Identifiers:** Choose a stable identifier for actions (array index is OK for Part 2; use `id` in Part 4).
- **No JS needed:** Keep interactions as simple HTML forms.
- **Validation:** Be kind to users—reject empty titles or clearly malformed URLs with a helpful message.
- **Restart behavior:** In Part 2, state resets on restart. In Part 4, state must persist.

---

## Run (any time)

```bash
uv sync
uv run flask --app app run --debug
```

Or:

```bash
uv run python app.py
```

Open `http://127.0.0.1:5000/` to see the site.

---

## Optional stretch (only if time allows)

- Unhide a post from the Hidden section.
- Prevent negative scores or display them differently.
- Add delete or edit for posts.
- Add pagination if the list grows long.
- Add simple unit tests for your sort/visibility logic.
- CSS polish and accessibility checks (labels, focus order, contrast).
