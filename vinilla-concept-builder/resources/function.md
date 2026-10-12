---
name: demo-app-js
description: Use when building a clickable concept demo as a vanilla JS web app with realistic filler data. Pairs with the web-app-styling skill for visuals.
---

# Clickable Concept Demo: Vanilla JS

A concept demo exists to make someone believe in an idea in a few minutes of clicking. It has to *feel* like a real product: realistic data, working navigation, actions that visibly do something. It does not need a backend, auth, validation edge cases, or production architecture.

Optimize for: **believable, clickable, easy to change, zero setup.**

## Ground rules

- **No frameworks, no build step.** Plain HTML + CSS + one `app.js` (split into a `data.js` only if the data gets long). It must open by double-clicking `index.html` or loading in any static host.
- **No external APIs.** Everything is local filler data. Nothing should break offline.
- **Every visible control does something.** If a button can't really work, make it show a believable result (a toast, a state change, a modal) rather than nothing. Dead clicks kill the illusion.
- **Fake it at the data layer, not the UI layer.** The UI should read and write a single state object as if it were real, so the demo can be swapped to a real API later with little change.

## File structure

```
index.html     # shell: nav, <main id="app">, toast container, script tags
styles.css     # follows the web-app-styling skill
data.js        # seed data + generators (optional; inline in app.js if short)
app.js         # state, router, render functions, actions, events
```

Order inside `app.js`, top to bottom:

1. Utilities (`$`, `escape`, formatters, `sleep`)
2. State (seeded from data)
3. Actions (functions that change state)
4. Views (functions that return HTML strings)
5. Router
6. Event handling (one delegated listener)
7. Boot

## Filler data: make it believable

Filler data is half of what makes a demo convincing. Lorem ipsum and "User 1, User 2" immediately read as fake.

- **Use realistic, varied names, companies, and content** that fit the domain. Mix name origins and lengths. Include a few long values to prove the layout handles them.
- **Make numbers plausible and uneven.** `$12,480.50`, `$940.00`, `$1,203,775.10`, not `$100, $200, $300`. Real distributions have outliers.
- **Use relative, recent dates** computed from today, so the demo never looks stale: "2 hours ago", "Yesterday", "Oct 3".
- **Include every status** the UI supports (active, pending, failed, archived) so all the styles get exercised.
- **Make it deterministic.** Use a seeded random so the demo looks the same every time someone presents it.
- **Have a hero record.** One item with the richest, most interesting data, placed where the presenter will click first.
- **Right amount:** 8–30 items per list. Enough to scroll and feel real, few enough to hand-write the important ones.

```js
// Seeded random: same data on every load
function mulberry32(seed) {
  return () => {
    seed |= 0; seed = (seed + 0x6D2B79F5) | 0;
    let t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}
const rand = mulberry32(42);
const pick = (arr) => arr[Math.floor(rand() * arr.length)];
const between = (min, max) => min + rand() * (max - min);

const daysAgo = (n) => new Date(Date.now() - n * 864e5);

// Hand-write the important records, generate the long tail
const SEED_PROJECTS = [
  { id: "p1", name: "Northwind Q4 Rebrand", client: "Northwind Traders", status: "active",
    budget: 48250, spent: 31790.4, owner: "Priya Raman", updated: daysAgo(0.1) },
  { id: "p2", name: "Harbor & Pine — Checkout Redesign", client: "Harbor & Pine Outfitters", status: "at_risk",
    budget: 22000, spent: 21480, owner: "Marcus Okafor", updated: daysAgo(1) },
];

const OWNERS = ["Priya Raman", "Marcus Okafor", "Lena Fischer", "Tomás Alvarez", "Aiko Tanaka", "Sam Whitfield"];
const CLIENTS = ["Lumen Health", "Brightline Freight", "Otter Coffee Co.", "Meridian Credit Union", "Fieldstone Labs"];
const STATUSES = ["active", "active", "active", "paused", "at_risk", "done"]; // weighted

function generateProjects(n) {
  return Array.from({ length: n }, (_, i) => {
    const budget = Math.round(between(4000, 90000) / 50) * 50;
    return {
      id: `p${i + 3}`,
      name: `${pick(["Website", "Onboarding", "Brand", "Mobile app", "Analytics"])} ${pick(["refresh", "build", "audit", "phase 2", "migration"])}`,
      client: pick(CLIENTS),
      status: pick(STATUSES),
      budget,
      spent: Math.round(budget * between(0.05, 1.05) * 100) / 100,
      owner: pick(OWNERS),
      updated: daysAgo(between(0.5, 40)),
    };
  });
}
```

## State

One plain object. Every view reads from it; every action writes to it and then re-renders.

```js
const initialState = () => ({
  projects: [...SEED_PROJECTS, ...generateProjects(18)],
  filter: "all",
  query: "",
  selectedId: null,
  loading: false,
});

let state = initialState();

function setState(patch) {
  state = { ...state, ...patch };
  render();
}
```

**Persistence is optional.** If edits should survive a refresh, save to `localStorage` wrapped in try/catch (it can throw or be empty), and always provide a way back to the seed. Dates must be revived after `JSON.parse`. For most demos, *not* persisting is better: every presentation starts clean.

```js
function save() { try { localStorage.setItem("demo", JSON.stringify(state)); } catch {} }
function load() {
  try {
    const s = JSON.parse(localStorage.getItem("demo"));
    if (s) s.projects.forEach(p => (p.updated = new Date(p.updated)));
    return s;
  } catch { return null; }
}
```

## Fake actions with fake latency

Instant results feel fake. A short delay plus a loading state makes actions feel real and shows off the loading design.

```js
const sleep = (ms) => new Promise(r => setTimeout(r, ms));

async function createProject(fields) {
  setState({ loading: true });
  await sleep(450 + rand() * 300);           // 450–750ms feels like a network call
  const project = { id: `p${Date.now()}`, status: "active", spent: 0, updated: new Date(), ...fields };
  setState({ loading: false, projects: [project, ...state.projects] });
  toast(`Created “${project.name}”`);
  navigate(`#/projects/${project.id}`);
}

async function archiveProject(id) {
  const prev = state.projects;
  setState({ projects: prev.map(p => p.id === id ? { ...p, status: "archived" } : p) }); // optimistic
  toast("Project archived", { action: "Undo", onAction: () => setState({ projects: prev }) });
}
```

Patterns worth faking because they sell the concept:
- Create, edit, delete/archive, with **undo** on destructive actions
- Search and filter that actually filter the list
- Sorting by clicking column headers
- A detail view for every list item
- One "magic moment" for the concept (the feature the demo is really about) — give it the most polish, including a slightly longer fake "processing" state

## Rendering

Views are functions that return HTML strings. Re-render the whole `<main>` on every change; at demo scale this is fast and keeps the code simple.

**Always escape** any data you interpolate, including filler (it will eventually contain an `&` or a user's typed input).

```js
const esc = (s) => String(s ?? "").replace(/[&<>"']/g, c => ({ "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;" }[c]));
const money = (n) => n.toLocaleString("en-US", { style: "currency", currency: "USD" });
const ago = (d) => {
  const s = (Date.now() - d) / 1000;
  if (s < 3600) return `${Math.max(1, Math.round(s / 60))}m ago`;
  if (s < 86400) return `${Math.round(s / 3600)}h ago`;
  if (s < 172800) return "Yesterday";
  return d.toLocaleDateString("en-US", { month: "short", day: "numeric" });
};

function ProjectList() {
  const items = state.projects
    .filter(p => state.filter === "all" || p.status === state.filter)
    .filter(p => p.name.toLowerCase().includes(state.query.toLowerCase()));

  if (!items.length) return EmptyState("No projects match", "Try a different filter or search term.");

  return `
    <table class="table">
      <thead><tr><th>Project</th><th>Owner</th><th class="num">Spent</th><th>Updated</th></tr></thead>
      <tbody>
        ${items.map(p => `
          <tr data-action="open" data-id="${p.id}" tabindex="0">
            <td>${esc(p.name)}<div class="muted">${esc(p.client)}</div></td>
            <td>${esc(p.owner)}</td>
            <td class="num">${money(p.spent)}</td>
            <td class="muted">${ago(p.updated)}</td>
          </tr>`).join("")}
      </tbody>
    </table>`;
}
```

Write a view for every state, not just the happy path: list, detail, empty, loading, and a not-found for bad routes.

Keep focus and scroll sane on re-render: if an input had focus (e.g. search), restore focus and cursor position after rendering, or update that part of the DOM separately instead of re-rendering it.

## Routing

Use hash routes so back/forward and refresh work, and the presenter can deep-link to a specific screen.

```js
const routes = [
  [/^#\/?$/,                 () => ProjectsPage()],
  [/^#\/projects\/([\w-]+)$/, (id) => ProjectDetailPage(id)],
  [/^#\/settings$/,          () => SettingsPage()],
];

function navigate(hash) { location.hash = hash; }

function render() {
  const hash = location.hash || "#/";
  for (const [re, view] of routes) {
    const m = hash.match(re);
    if (m) { document.getElementById("app").innerHTML = view(...m.slice(1)); updateNav(hash); return; }
  }
  document.getElementById("app").innerHTML = NotFound();
}

window.addEventListener("hashchange", () => { render(); window.scrollTo(0, 0); });
```

## Events

One delegated listener per event type on `document`. Elements declare intent with `data-action`; no inline `onclick`, no re-binding after render.

```js
const actions = {
  open:    (el) => navigate(`#/projects/${el.dataset.id}`),
  filter:  (el) => setState({ filter: el.dataset.value }),
  archive: (el) => archiveProject(el.dataset.id),
  "reset-demo": () => { state = initialState(); render(); toast("Demo reset"); },
};

document.addEventListener("click", (e) => {
  const el = e.target.closest("[data-action]");
  if (el && actions[el.dataset.action]) { e.preventDefault(); actions[el.dataset.action](el, e); }
});

document.addEventListener("keydown", (e) => {
  if (e.key === "Enter" && e.target.matches("tr[data-action]")) e.target.click();
  if (e.key === "Escape") closeModal();
});

document.addEventListener("input", (e) => {
  if (e.target.matches("[data-bind='query']")) setState({ query: e.target.value });
});

document.addEventListener("submit", (e) => {
  if (e.target.matches("[data-form='new-project']")) {
    e.preventDefault();
    createProject(Object.fromEntries(new FormData(e.target)));
  }
});
```

## Feedback components

Small, shared helpers that make the demo feel finished:

- **Toast:** bottom corner, auto-dismiss after ~3s, optional action button (for undo)
- **Modal:** for create/edit and confirmations; Escape and backdrop click close it; focus the first input on open
- **Skeleton/loading state:** shown while `state.loading` is true
- **Empty state:** short headline, one line of help, one action button

## Demo-specific affordances

- A **"Reset demo"** action tucked in settings or a footer, so the presenter can start clean between runs
- Stable **deep links** to the key screens (e.g. `#/projects/p1`) for jumping straight to the good part
- Optional `?scenario=empty` or `?scenario=error` URL param that swaps the seed data, to show those states on demand

## What not to do

- No lorem ipsum, "Test User", "Item 1/2/3", `example@example.com`, or round fake numbers
- No `alert()` / `confirm()` dialogs — use real UI
- No buttons that do nothing; no `href="#"` links that jump to top
- No hardcoded dates that will look old next month
- No `innerHTML` with unescaped data
- No frameworks, bundlers, TypeScript, or package installs for a click-through demo
- Don't over-engineer: no component class hierarchies, stores, or event buses

## Final check

- [ ] Opens with no build step and no console errors
- [ ] Every button, link, and row does something visible
- [ ] Data looks real at a glance, including a long name and an outlier number
- [ ] Back/forward and refresh keep you on the right screen
- [ ] Empty, loading, and not-found states reachable and styled
- [ ] Destructive actions have undo or confirmation
- [ ] Keyboard works: Tab, Enter on rows, Escape closes modals
- [ ] "Reset demo" returns everything to the seed state
