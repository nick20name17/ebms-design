# HTML Design File Guidelines

You are helping a designer create and iterate on HTML files for the EBMS Client Platform — a team review tool where reviewers leave comments anchored to specific page elements.

## What You're Building

Single self-contained `.html` files — typically e-commerce pages, production management interfaces, or marketing sites. These files are rendered inside an iframe on the review platform, where team members click on elements to leave comments.

Every visible element MUST have a `data-comment` attribute with a unique value. This is how the platform anchors comments to elements. Missing `data-comment` = element cannot be reviewed.

---

## File Structure

Every HTML file follows this exact structure:

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Project Name — Page</title>

        <!-- Google Fonts only -->
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
        <link
            href="https://fonts.googleapis.com/css2?family=..."
            rel="stylesheet" />

        <style>
            /* Reset */
            *,
            *::before,
            *::after {
                box-sizing: border-box;
                margin: 0;
                padding: 0;
            }

            /* Design tokens as CSS custom properties */
            :root {
                /* -- Colors -- */
                /* -- Typography -- */
                /* -- Spacing -- */
            }

            /* All styles here — no external CSS files */
        </style>
    </head>
    <body>
        <!-- All markup here -->

        <script>
            /* All scripts here — no external JS files */
        </script>
    </body>
</html>
```

### Rules

- **One file = everything.** HTML, CSS, and JS all live in a single `.html` file. No external CSS or JS files.
- **Google Fonts only** for custom typefaces. Always include `preconnect` hints.
- **Libraries are OK** if loaded from CDN and useful (e.g., GSAP for animation, Swiper for carousels). Keep it pragmatic — don't add dependencies without a reason.
- **CSS custom properties** in `:root` for all design tokens (colors, fonts, spacing). This keeps the design system readable and easy to adjust during review.

---

## data-comment Attribute Convention

### The Rule

Every element that is visible or interactive MUST have a unique `data-comment` attribute.

### Naming Pattern

Use `section-element` or `section-element-variant` format:

```
header-logo
header-nav
header-nav-link-home
header-nav-link-cart

hero-heading
hero-subheading
hero-cta-primary
hero-cta-secondary
hero-image

products-grid
products-card-1
products-card-1-image
products-card-1-title
products-card-1-price
products-card-2
products-card-2-image
...

footer-copyright
footer-social-instagram
footer-social-twitter
```

### What Gets a data-comment

| Element type | Gets data-comment? | Example |
| --- | --- | --- |
| Headings, paragraphs, labels | ✅ Yes | `data-comment="hero-heading"` |
| Buttons, links, inputs | ✅ Yes | `data-comment="hero-cta-primary"` |
| Images, icons, SVGs | ✅ Yes | `data-comment="hero-image"` |
| Cards, list items | ✅ Yes | `data-comment="products-card-1"` |
| Section/wrapper containers | ✅ Yes | `data-comment="hero-section"` |
| Layout-only wrappers with no visual meaning | ❌ No | Generic flex/grid containers that are purely structural |
| `<html>`, `<head>`, `<body>` | ❌ No | — |
| `<script>`, `<style>`, `<meta>`, `<link>` | ❌ No | — |

### Numbered Items

For repeated elements (cards, list items, table rows), use sequential numbering:

```html
<div data-comment="products-card-1">...</div>
<div data-comment="products-card-2">...</div>
<div data-comment="products-card-3">...</div>
```

Children inherit the parent's prefix:

```html
<div data-comment="products-card-1">
    <img data-comment="products-card-1-image" />
    <h3 data-comment="products-card-1-title">...</h3>
    <span data-comment="products-card-1-price">...</span>
</div>
```

### Uniqueness

No two elements in the same file may share a `data-comment` value. Before committing, mentally verify there are no duplicates.

---

## SPA Navigation

**SPA is the default.** All pages/views go into a single `.html` file unless the user explicitly asks for separate HTML files. This keeps state shared across views and simplifies navigation.

Follow this pattern:

```html
<!-- Navigation triggers -->
<button data-comment="nav-link-home" onclick="navigate('home')">Home</button>
<button data-comment="nav-link-catalog" onclick="navigate('catalog')">
    Catalog
</button>

<!-- Views -->
<div id="view-home" class="view active" data-comment="view-home">...</div>
<div id="view-catalog" class="view" data-comment="view-catalog">...</div>
```

```css
.view {
    display: none;
    opacity: 0;
    transition: opacity 0.4s ease;
}
.view.active {
    display: block;
    opacity: 1;
}
```

```js
function navigate(page) {
    document.querySelectorAll(".view").forEach((v) => {
        v.classList.remove("active");
        setTimeout(() => {
            if (!v.classList.contains("active")) v.style.display = "none";
        }, 400);
    });
    const target = document.getElementById("view-" + page);
    target.style.display = "block";
    requestAnimationFrame(() =>
        requestAnimationFrame(() => target.classList.add("active")),
    );

    document.querySelectorAll('[data-comment^="nav-link-"]').forEach((link) => {
        link.classList.toggle(
            "active",
            link.dataset.comment === "nav-link-" + page,
        );
    });

    window.scrollTo({ top: 0, behavior: "smooth" });
}
```

Each view's elements still need unique `data-comment` values. Prefix with the view name:

```
home-hero-heading       (not just "hero-heading")
catalog-filter-price    (not just "filter-price")
```

---

## CSS Conventions

### Design Tokens

Always define tokens in `:root`. Group them clearly:

```css
:root {
    /* Colors */
    --color-primary: #...;
    --color-secondary: #...;
    --color-bg: #...;
    --color-text: #...;
    --color-muted: #...;
    --color-border: #...;

    /* Typography */
    --font-heading: "Font Name", serif;
    --font-body: "Font Name", sans-serif;
    --font-mono: "Font Name", monospace;

    /* Spacing (if useful) */
    --space-xs: 8px;
    --space-sm: 16px;
    --space-md: 32px;
    --space-lg: 64px;
}
```

### Selectors

Use `data-comment` attribute selectors for styling when it makes sense for unique elements:

```css
header[data-comment="header-root"] { ... }
```

Class selectors are fine for shared styles:

```css
.btn-primary { ... }
.card { ... }
```

Use whichever is cleaner. For one-off sections, `data-comment` selectors keep HTML minimal. For reused components, classes are better.

### Responsive Design

Add media queries when the project requires responsive behavior. Not every project needs them — confirm before adding.

Use Tailwind-aligned breakpoints so responsive behavior carries over to the final React + Tailwind build:

```css
:root {
    /* Breakpoints (reference — use in media queries) */
    /* sm: 640px, md: 768px, lg: 1024px, xl: 1280px, 2xl: 1536px */
}

/* Mobile first — base styles are mobile, then scale up */
.products-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 16px;
}

@media (min-width: 640px) {   /* sm */
    .products-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {  /* lg */
    .products-grid { grid-template-columns: repeat(3, 1fr); }
}

@media (min-width: 1280px) {  /* xl */
    .products-grid { grid-template-columns: repeat(4, 1fr); }
}
```

**Rules:**
- **Mobile first** — base CSS is the mobile layout, use `min-width` to scale up (matches Tailwind's approach)
- **Use only these breakpoints:** `640px` (sm), `768px` (md), `1024px` (lg), `1280px` (xl), `1536px` (2xl)
- **Comment the breakpoint name** next to each media query for readability
- Not every project needs all breakpoints — use only what the design requires

---

## JavaScript Conventions

- Vanilla JS by default. Libraries only when they save significant effort.
- No frameworks (React, Vue, etc.) — these are static HTML review files.
- Keep JS at the bottom inside a single `<script>` tag.
- SPA navigation logic (if applicable) should follow the pattern above.
- Interactive elements (forms, modals, accordions) should work in the preview but don't need real backend connections. Mock the behavior.

---

## State & Data Management

These prototypes behave like real apps — full CRUD, filters, selections, form state — but with no backend. All data lives in memory and resets on page reload. That's fine.

### The Store

Use this vanilla JS store (Zustand-like pattern, no dependencies):

```js
function createStore(initialState) {
    let state = structuredClone(initialState);
    const listeners = new Set();
    return {
        get: () => state,
        set(updater) {
            state = typeof updater === 'function'
                ? updater(state)
                : { ...state, ...updater };
            listeners.forEach(fn => fn(state));
        },
        subscribe(fn) {
            listeners.add(fn);
            return () => listeners.delete(fn);
        }
    };
}
```

### Rules

- **In-memory only.** No `localStorage`, `sessionStorage`, or cookies. Data resets on reload — this is intentional.
- **One store per app.** Define it once at the top of the `<script>` block with all initial data (seed/mock data).
- **Store survives SPA navigation.** Since views are just toggled divs, the store stays alive across all views — no need to persist or restore state between pages.
- **Seed with realistic mock data.** Pre-populate the store with enough entries to make the UI look real (e.g., 5–10 users, a few products, etc.).
- **Full CRUD should work.** If the design shows a user list with add/edit/delete — all of those actions should actually work against the store and the UI should update.
- **Subscribe to re-render.** After mutating the store, the UI must reflect the change. Use `store.subscribe()` to trigger render functions.

### Example Usage

```js
// Define store with seed data
const store = createStore({
    users: [
        { id: 1, name: 'Anna Lee', email: 'anna@example.com' },
        { id: 2, name: 'Mark Chen', email: 'mark@example.com' },
    ],
    selectedUserId: null,
});

// Create
function addUser(name, email) {
    store.set(s => ({
        users: [...s.users, { id: Date.now(), name, email }]
    }));
}

// Update
function updateUser(id, updates) {
    store.set(s => ({
        users: s.users.map(u => u.id === id ? { ...u, ...updates } : u)
    }));
}

// Delete
function deleteUser(id) {
    store.set(s => ({
        users: s.users.filter(u => u.id !== id),
        selectedUserId: s.selectedUserId === id ? null : s.selectedUserId
    }));
}

// UI sync
function renderUsers() {
    const { users } = store.get();
    document.querySelector('[data-comment="users-list"]').innerHTML =
        users.map(u => `<div data-comment="users-item-${u.id}">${u.name}</div>`).join('');
}

store.subscribe(renderUsers);
renderUsers(); // initial render
```

### What Belongs in the Store

| Data type | In store? | Example |
| --- | --- | --- |
| Entity collections (CRUD) | ✅ Yes | `users`, `products`, `orders` |
| UI selection state | ✅ Yes | `selectedUserId`, `activeTab` |
| Form drafts | ✅ Yes | `editingUser: { name: '...' }` |
| Filters / search query | ✅ Yes | `searchTerm`, `filterStatus` |
| SPA current page | ❌ No | Handled by `navigate()` |
| Pure visual toggles | ❌ No | Modal open/close — use class toggles |

---

## UI States

Every data-driven view should account for all possible states — not just the "happy path" with data. These are real design decisions that carry over to the React build.

### Required States

For any view that displays data from the store, implement these four states:

**1. Loading state** — shown briefly on initial render or when navigating to a view. Use CSS skeleton placeholders:

```css
.skeleton {
    background: linear-gradient(90deg, var(--color-border) 25%, transparent 50%, var(--color-border) 75%);
    background-size: 200% 100%;
    animation: skeleton-pulse 1.5s ease-in-out infinite;
    border-radius: 4px;
}

@keyframes skeleton-pulse {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
}
```

```js
// Simulate loading on view navigation
function navigate(page, params = {}) {
    // ... existing logic ...
    showLoadingState(page);
    setTimeout(() => renderView(page), 300); // brief fake delay
}

function showLoadingState(page) {
    const container = document.getElementById('view-' + page);
    container.innerHTML = getSkeletonHTML(page);
}
```

Create skeleton layouts that roughly match the real content shape — a few rectangular bars for text, larger blocks for cards/images. Maps directly to shadcn's `skeleton` component later.

**2. Empty state** — shown when a collection has zero items. Always include a message and an action:

```html
<div data-comment="users-empty-state" data-component="card" class="empty-state">
    <div data-comment="users-empty-icon" class="empty-state-icon">👤</div>
    <h3 data-comment="users-empty-heading">No users yet</h3>
    <p data-comment="users-empty-text">Get started by adding your first user.</p>
    <button data-comment="users-empty-cta" data-component="button" onclick="openAddUserForm()">
        Add User
    </button>
</div>
```

```css
.empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 64px 24px;
    text-align: center;
    color: var(--color-muted);
}

.empty-state-icon {
    font-size: 48px;
    margin-bottom: 16px;
}
```

**3. Data state** — the normal view with data. This is what you'd build anyway.

**4. Error state** — shown when an action fails. Since we have no real API, use this for form validation errors:

```html
<div data-comment="users-form-error" data-component="alert" class="error-message" style="display: none;">
    <span data-comment="users-form-error-text">Please fill in all required fields.</span>
</div>
```

```css
.error-message {
    padding: 12px 16px;
    background: color-mix(in srgb, var(--color-error) 10%, transparent);
    border: 1px solid var(--color-error);
    border-radius: 6px;
    color: var(--color-error);
    font-size: 14px;
}
```

### Render Pattern

Structure render functions to handle all states:

```js
function renderUserList() {
    const { users, isLoading } = store.get();
    const container = document.querySelector('[data-comment="users-list-container"]');

    if (isLoading) {
        container.innerHTML = renderUserListSkeleton();
        return;
    }

    if (users.length === 0) {
        container.innerHTML = renderUserListEmpty();
        return;
    }

    container.innerHTML = users.map(u => renderUserCard(u)).join('');
}
```

### Form Validation

Validate before store actions. Show inline errors next to fields:

```js
function validateUserForm(data) {
    const errors = {};
    if (!data.name.trim()) errors.name = 'Name is required';
    if (!data.email.trim()) errors.email = 'Email is required';
    else if (!data.email.includes('@')) errors.email = 'Invalid email address';
    return { valid: Object.keys(errors).length === 0, errors };
}

function handleAddUser() {
    const data = getFormData();
    const { valid, errors } = validateUserForm(data);

    clearFormErrors();
    if (!valid) {
        showFormErrors(errors); // display inline error messages
        return;
    }

    addUser(data);
    closeForm();
}
```

This maps directly to react-hook-form + zod validation in the final React build.

---

## React-Ready Conventions

These prototypes will eventually be implemented as React apps (Vite + React Query + TanStack Router + shadcn/ui + Tailwind). The rules below make that conversion straightforward — follow them now so the HTML prototypes map cleanly to the final stack.

### Render Functions = Future Components

Every distinct UI piece should have its own render function. Name them like React components — `PascalCase` with a `render` prefix:

```js
function renderUserCard(user) { ... }
function renderUserList() { ... }
function renderUserForm() { ... }
function renderSidebar() { ... }
function renderDashboardStats() { ... }
```

Each render function should be self-contained: it receives data (or reads from the store) and returns/injects HTML. One render function = one future React component.

### Views = Future Routes

SPA views map directly to TanStack Router routes. Name and structure them accordingly:

| SPA view ID | Future route |
| --- | --- |
| `view-dashboard` | `/dashboard` |
| `view-users` | `/users` |
| `view-user-detail` | `/users/$userId` |
| `view-settings` | `/settings` |

If a view needs a parameter (like a user ID), pass it through the `navigate()` function and store it in the store:

```js
function navigate(page, params = {}) {
    // ... existing navigation logic ...
    if (params.id) store.set({ selectedId: params.id });
}

// Usage
navigate('user-detail', { id: 42 });
```

### Store Shape = Future API Cache

Structure store data as if it came from a REST API — this maps directly to React Query cache keys later:

```js
const store = createStore({
    // Each key = a future React Query query key
    // Arrays of objects with `id` — like API responses
    users: [
        { id: 1, name: 'Anna Lee', email: 'anna@example.com', role: 'admin' },
        { id: 2, name: 'Mark Chen', email: 'mark@example.com', role: 'user' },
    ],
    products: [
        { id: 1, title: 'Widget', price: 29.99, status: 'active' },
    ],

    // UI state — this becomes Zustand/local state in React, not React Query
    selectedUserId: null,
    searchTerm: '',
    filterStatus: 'all',
});
```

**Rules for store data shape:**
- Every entity has a numeric or string `id`
- Collections are always arrays (not objects/maps)
- Nest related data only when it's always fetched together (e.g., `user.address`), otherwise keep flat
- Separate server-like data (entities) from UI-only state (selections, filters)

### Action Functions = Future Mutations

CRUD action functions map to React Query mutations. Keep them separate from render logic:

```js
// These become useMutation hooks in React
function addUser(data) { store.set(s => ({ users: [...s.users, { id: Date.now(), ...data }] })); }
function updateUser(id, data) { store.set(s => ({ users: s.users.map(u => u.id === id ? { ...u, ...data } : u) })); }
function deleteUser(id) { store.set(s => ({ users: s.users.filter(u => u.id !== id) })); }
```

Group actions by entity at the top of the script, before render functions:

```
1. createStore() definition
2. Store initialization with seed data
3. Action functions (grouped by entity: user actions, product actions, etc.)
4. Render functions (grouped by view/component)
5. Event handlers and listeners
6. store.subscribe() calls
7. Initial render calls
8. navigate() and SPA logic
```

### shadcn/ui Component Hints

When building UI elements that have a direct shadcn equivalent, add a `data-component` attribute as a hint for the future implementation:

```html
<div data-comment="users-dialog" data-component="dialog">...</div>
<button data-comment="users-add-btn" data-component="button">Add User</button>
<table data-comment="users-table" data-component="table">...</table>
<input data-comment="users-search" data-component="input" />
<div data-comment="users-card-1" data-component="card">...</div>
<select data-comment="users-filter" data-component="select">...</select>
<div data-comment="users-tabs" data-component="tabs">...</div>
```

Common shadcn components to reference: `button`, `input`, `select`, `checkbox`, `radio-group`, `switch`, `slider`, `textarea`, `dialog`, `sheet`, `dropdown-menu`, `popover`, `tooltip`, `table`, `card`, `tabs`, `accordion`, `badge`, `avatar`, `alert`, `toast`, `separator`, `skeleton`, `pagination`, `breadcrumb`, `command`, `calendar`, `date-picker`, `form`.

Only add `data-component` when there's a clear 1:1 match. Don't force it on custom layouts.

---

## Git Commit Messages

Write short, human-readable commit messages in English. One sentence. Describe what changed visually, not technically.

### Good examples

```
Add hero section with heading and CTA buttons
Update product card layout to 3-column grid
Fix footer alignment and add social links
Restyle navigation with new color scheme
Add contact form page with validation states
Replace placeholder images with SVG illustrations
Adjust typography sizes across all sections
Add hover effects to product cards
Build out the full catalog page with filters
Polish mobile layout for the checkout flow
```

### Bad examples

```
feat: implement responsive grid system with CSS custom properties    ← too technical
update styles                                                        ← too vague
WIP                                                                  ← meaningless
fix: resolve z-index stacking context issue in header component      ← not human
```

---

## Pre-Commit Checklist

Before every commit, verify:

1. **Every visible element has a unique `data-comment`** — no missing, no duplicates
2. **File is self-contained** — no external CSS/JS file references (CDN libraries are OK)
3. **Page renders correctly** — open in browser, check all views/pages
4. **No dead code** — remove unused styles, commented-out blocks, placeholder text like "Lorem ipsum" (unless intentional)
5. **Clean formatting** — consistent indentation (2 spaces), organized CSS sections
6. **SPA navigation works** — if multi-view, all views accessible and transitions smooth
7. **Commit message is clear** — one sentence, describes the visual change

---

## When Modifying Existing Files

- **Don't remove `data-comment` attributes** that already exist — comments in the review platform depend on them
- **Don't change `data-comment` values** unless the element fundamentally changed — renaming breaks existing comment anchors
- When adding new elements, follow the existing naming pattern in the file
- If a section is restructured significantly, note it in the commit message so reviewers know their old comments may be displaced

---

## Validation

There is a validator script (`validate-html.sh`) that runs automatically before every commit via a git pre-commit hook. It checks:

- Every visible element has a `data-comment` attribute
- No duplicate `data-comment` values
- File is self-contained (no external local CSS/JS)
- Basic HTML structure (doctype, charset, viewport, title)
- CSS custom properties exist in `:root`
- SPA views have proper `data-comment` and `navigate()` function

**If the validator reports errors, fix them before committing.** Warnings are advisory but should be addressed when possible.

You can also run it manually:

```bash
bash ./validate-html.sh                  # check all HTML files
bash ./validate-html.sh index.html       # check specific file
```

---

## Project Structure

```
project-root/
├── INSTRUCTIONS.md        ← canonical rules (read by all agents)
├── CLAUDE.md              ← Claude Code → reads INSTRUCTIONS.md
├── AGENTS.md              ← OpenAI Codex → reads INSTRUCTIONS.md
├── .github/
│   └── copilot-instructions.md  ← GitHub Copilot → reads INSTRUCTIONS.md
├── .githooks/
│   └── pre-commit         ← validates HTML before every commit
├── validate-html.sh       ← validation script
├── index.html             ← main page (or the only page)
├── catalog.html           ← additional pages if needed
└── ...
```

Each `.html` file is fully independent. No shared assets between files.
