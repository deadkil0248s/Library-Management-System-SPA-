# LibraSolve — Library Management System

A fully client-side library management system built with vanilla HTML, CSS, and JavaScript. No backend, no build tools, no dependencies beyond a Google Fonts import. All data persists across sessions via the browser's `localStorage`.

---

## Features

### Dashboard
The landing page gives a real-time snapshot of the entire library state:
- **Stat cards** showing total books, registered users, available books (as a ratio), cumulative borrow count, and number of currently overdue books.
- **Popular Books panel** — lists the top 5 most-borrowed books ranked by borrow count, each with an inline popularity bar and availability badge.
- **Recent Activity panel** — shows the last 10 logged events (adds, borrows, returns) with timestamps, rendered in reverse-chronological order.

### Books
Full CRUD management of the book catalog:
- **Add a book** by entering a title and a comma-separated list of tags (genres, topics, etc.). Each book is assigned a zero-padded unique ID (e.g. `#001`).
- **Delete a book** — the delete button is revealed on card hover. Removing a book also cleans it out of every user's borrowed set and the overdue tracker automatically.
- **Grid / Table view toggle** — switch between a card grid layout and a compact data table. Both views show availability status, tags, and popularity bar.
- **Sort** the catalog by title A–Z, title Z–A, most popular, or newest first.
- **Tag-based category filter bar** — dynamically built from all unique tags across the catalog. Clicking a chip filters the visible books to that tag only. Clicking a tag on a book card also activates the filter.
- **Search** — live filter by title, tag, or book ID as you type. The global search bar in the topbar also navigates to the Books page and runs a search when Enter is pressed.
- **Availability badge** on every book shows a green "Available" or pink "Borrowed" dot, with a tooltip listing who currently holds the book.
- **Popularity bar** — a relative progress bar showing each book's borrow count proportional to the most-borrowed book in the catalog.

### Users
Management of library members:
- **Register a user** by name. Each user gets an auto-incremented numeric ID and a color-coded avatar generated from their initials.
- **Delete a user** with the remove button on their card.
- **User cards** display the user's name, ID, total books currently borrowed, and a list of the actual book titles they are holding.
- **Avatar colors** cycle through a palette of 8 preset colors assigned by user ID.

### Circulation
Handles the borrow and return workflow:
- **Borrow** — select a user and an available book from dropdowns. The book's borrow count increments and the book is removed from the available list immediately. Attempting to borrow a book that is already out raises a warning.
- **Return** — select a user and one of the books they currently hold. The return book dropdown is dynamically filtered to only show books that specific user has.
- **Transaction History** — a scrollable log of the last 20 borrow/return events with timestamps, displayed in reverse order (newest first).
- **Undo Last Action** — implemented as a stack (`borrowStack`). Pressing Undo pops the most recent transaction and reverses it: an undone borrow removes the book from the user and decrements the count; an undone return re-adds the book to the user's set.

### Recommendations
A tag-overlap scoring algorithm that suggests books a user has not yet read:
- For the selected user, the system collects every tag from books they have previously borrowed (their "interest profile").
- Each unread book is scored: `+0.5` for every tag that overlaps with the user's profile, plus `+0.1 × borrow_count` as a popularity boost.
- The top 5 results are displayed as ranked cards with their computed scores, tags, and availability status. The #1 result is highlighted in gold.

### Fines
Overdue fine calculator:
- Works against the due-date entries tracked in the Utilities section.
- **Configurable rate** — set a custom fine rate per day (default $2.00, adjustable in $0.50 steps). The entire list recalculates live when the rate changes.
- **Fine summary bar** shows total accrued fines, number of currently overdue books, total tracked books, and the single highest fine.
- Each entry in the list shows: book title, due date, days overdue or days remaining, fine amount, and a color-coded status badge (green = on time, yellow = due within 3 days, red = overdue).
- Entries are sorted by fine amount descending so the worst offenders appear first.

### Utilities
Two independent tools for queue management and due-date tracking:

**Book Request Queue**
- A FIFO queue for managing book hold requests.
- Select a book and press "Request" to add it to the back of the queue.
- Press "Process Next" to dequeue the front item (simulating fulfilling the next request in line).
- The queue is visualized as a horizontal chain of labeled tiles connected by arrows, with a pop-in animation when items are added.

**Overdue Tracker**
- A priority queue (sorted array) that tracks return due dates for borrowed books.
- Add a due entry by selecting a book and entering how many days from now it is due.
- Entries are kept sorted by due date (soonest first) at all times.
- Each entry shows the due date and a status badge: safe (green), warning — due in 3 days or fewer (yellow), or overdue (red).
- The Fines page reads directly from this same data structure.

---

## Data Structures Used

| Structure | Where used |
|---|---|
| Array (dynamic list) | Book catalog, user list, activity log |
| Set | Per-user borrowed book IDs (O(1) lookup, add, delete) |
| Stack (LIFO array) | `borrowStack` — transaction history and undo system |
| Queue (FIFO array) | `requestQueue` — book hold request queue |
| Priority Queue (sorted array) | `overduePQ` — due dates sorted soonest-first |
| Hash map (plain object) | Tag-based scoring in the recommendation engine |

---

## Persistence

All state is serialized to `localStorage` under the key `librasync_data` on every write operation. On page load, state is restored from storage. Because `Set` is not JSON-serializable, user borrowed sets are converted to arrays on save and reconstructed as `Set` on load.

Theme preference is stored separately under `librasync_theme`.

---

## Theming

Supports dark and light modes toggled by the switch in the topbar. The active theme is saved to localStorage and restored on reload. All colors are defined as CSS custom properties (variables) on `[data-theme]` attribute selectors, so the entire palette swaps instantly with a single attribute change on `<html>`.

---

## Security

All user-supplied strings are passed through `escapeHtml()` before being inserted into the DOM. This function replaces `&`, `<`, `>`, `"`, and `'` with their HTML entity equivalents, preventing XSS via book titles, user names, or tags.

---

## Responsive Layout

The sidebar collapses off-screen on viewports narrower than 768px and is revealed by a hamburger button. A dark overlay covers the main content when the sidebar is open and dismisses it on click. Stat cards reflow from 5 columns to 2 on tablet and 1 on mobile. Book and user grids also collapse to single-column on small screens.

---

## Project Structure

```
├── app.html       # Main application (HTML + JS)
├── styles.css     # All styles and CSS custom properties
├── index2.html    # Original single-file version
└── README.md
```

---

## Running Locally

Open `app.html` directly in any modern browser. No server, build step, or installation required.
