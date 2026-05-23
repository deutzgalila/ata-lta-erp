# ATA & LTA Accounting Firm ERP — Functional HTML/JS/CSS Mockup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a fully functional HTML/JS/CSS mockup of an ERP system for ATA & LTA Accounting Firm, with localStorage persistence, 7 modules, 2-entity data segregation, RBAC, and realistic seed data for client demo.

**Architecture:** Single-page application (SPA) using vanilla HTML/CSS/JS with hash-based routing. No build tools or frameworks — open `index.html` in any browser. localStorage persists all data with schema versioning and seed data reset capability. CSS custom properties for theming. Module-based JS files loaded via `<script>` tags. All DOM manipulation uses safe methods (createElement, textContent, setAttribute) — no innerHTML with dynamic content.

**Tech Stack:** HTML5, CSS3 (with CSS custom properties), Vanilla ES6 JavaScript, localStorage. No external libraries or build steps.

---

## File Structure

| File | Responsibility |
|---|---|
| `index.html` | SPA shell: login screen, global nav, header with entity switcher, content area for module views |
| `css/styles.css` | All styles: CSS variables for colors/fonts/spacing, responsive grid, component styles, print styles for invoices |
| `js/data.js` | localStorage CRUD wrapper, schema versioning, seed data generation, data migration |
| `js/auth.js` | Login/logout, session management, RBAC permission checks, active entity context |
| `js/app.js` | Hash router, shell rendering, navigation, module initialization, global event delegation |
| `js/utils.js` | PHP currency formatter, date helpers, form validators, deep clone, debounce, safe DOM builder |
| `js/dashboard.js` | Managerial Firm Overview: consolidated KPIs across ATA+LTA, entity switching, charts (CSS-based) |
| `js/clients.js` | Client CRUD: master records with entity assignment, retainer status, search/filter |
| `js/workflow.js` | Work Request CRUD, task assignment, dependency engine (DAG), retainer template creation, status workflow |
| `js/billing.js` | Invoice creation (Sales Invoice), line items (PF + gov't fees), VAT calculation, aging report, PDF export |
| `js/disbursement.js` | Expense filing, fund source tagging, 2-tier vs 1-tier approval workflows, reimbursement summary |
| `js/dms.js` | Document upload metadata, version tracking, original/copy flag, handover log |
| `js/reports.js` | Basic reports: work request volume, task completion rate, billing summary, disbursement by category, entity P&L snapshot |
| `js/users.js` | Admin panel: user management, reset demo data, audit log |

---

## Task 1: Project Scaffold — HTML Shell & CSS Foundation

**Files:**
- Create: `index.html`
- Create: `css/styles.css`
- Create: `js/utils.js`
- Create: `js/data.js` (empty shell)

- [ ] **Step 1: Create `index.html` with SPA shell**

Create `index.html` with:
- `<!DOCTYPE html>` with lang="en"
- Meta viewport for responsive design
- Link to `css/styles.css`
- `<body>` containing:
  - `#login-screen` — centered card with email/password inputs and "Sign In" button
  - `#app-shell` (hidden by default) containing:
    - `<header>` with logo "ATA & LTA ERP", entity switcher dropdown (`#entity-switcher`), user name display (`#user-name`), logout button
    - `<nav>` sidebar with module links: Dashboard, Clients, Workflow, Billing, Disbursement, Documents, Reports, Admin (Users)
    - `<main id="content">` empty content area
- Load all JS files in order via `<script>` tags at end of body: utils, data, auth, dashboard, clients, workflow, billing, disbursement, dms, reports, users, app

- [ ] **Step 2: Create `js/utils.js` with helper functions**

Create `js/utils.js` with:
```javascript
function formatPHP(n) {
  return '₱' + Number(n).toLocaleString('en-PH', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}
function formatDate(d) {
  return new Date(d).toLocaleDateString('en-PH', { year: 'numeric', month: 'short', day: 'numeric' });
}
function debounce(fn, ms) {
  let t;
  return (...a) => { clearTimeout(t); t = setTimeout(() => fn(...a), ms); };
}
function deepClone(obj) { return JSON.parse(JSON.stringify(obj)); }
function generateId(prefix) {
  return prefix + '-' + Date.now().toString(36) + '-' + Math.random().toString(36).slice(2, 6);
}
// Safe DOM builder: el('div', {class: 'card'}, [child1, child2])
function el(tag, attrs = {}, children = []) {
  const node = document.createElement(tag);
  for (const [k, v] of Object.entries(attrs)) {
    if (k === 'text') node.textContent = v;
    else if (k === 'html') node.innerHTML = v; // only for static HTML in plan
    else node.setAttribute(k, v);
  }
  children.forEach(c => {
    if (typeof c === 'string') node.appendChild(document.createTextNode(c));
    else node.appendChild(c);
  });
  return node;
}
```

- [ ] **Step 3: Create `css/styles.css` with design system**

Create `css/styles.css` with CSS custom properties:
```css
:root {
  --color-ata: #2563eb; --color-lta: #16a34a;
  --color-bg: #f8fafc; --color-surface: #ffffff;
  --color-text: #1e293b; --color-text-muted: #64748b;
  --color-border: #e2e8f0; --color-primary: #0f172a;
  --color-danger: #dc2626; --color-success: #16a34a;
  --color-warning: #ca8a04;
  --font-sans: system-ui, -apple-system, sans-serif;
  --radius-sm: 4px; --radius-md: 8px; --radius-lg: 12px;
  --spacing-xs: 4px; --spacing-sm: 8px; --spacing-md: 16px;
  --spacing-lg: 24px; --spacing-xl: 32px;
  --sidebar-width: 240px;
}
```
Add base styles for body, layout grid (sidebar + main), typography, form elements (single-column), buttons, tables, cards, badges, modals, progress bar component. Ensure responsive: sidebar collapses to hamburger menu below 768px.

- [ ] **Step 4: Verify in browser**

Open `index.html` in browser. Verify: login card is centered and styled, app shell is hidden. No console errors.

- [ ] **Step 5: Commit**

```bash
git add index.html css/styles.css js/utils.js js/data.js
git commit -m "feat: scaffold SPA shell with CSS design system and safe DOM utils"
```

---

## Task 2: Data Layer — localStorage, Schema, Seed Data

**Files:**
- Modify: `js/data.js`

- [ ] **Step 1: Create seed data and localStorage wrapper**

In `js/data.js`, define:

**Seed data object** containing:
- `schemaVersion: 1`
- `users`: 10 users across 6 departments with roles per entity:
  - u1: Admin, both entities, Managerial
  - u2: Manager, both entities, Managerial
  - u3: Manager, ATA only, Accounting
  - u4: Staff, ATA only, Accounting
  - u5: Staff, LTA only, Accounting
  - u6: Staff, ATA only, Operations
  - u7: Staff, LTA only, Operations
  - u8: Staff, both entities, Documentations
  - u9: Staff, both entities, HR
  - u10: Staff, both entities, Admin
  - All passwords: `password123`
- `clients`: 8 clients (4 ATA, 4 LTA), with tin, contact, address, retainer flag
- `workRequests`: 6 requests across both entities, various statuses
- `tasks`: 15 tasks with dependencies (t2 depends on t1, t4 and t5 both depend on t3, etc.)
- `invoices`: 4 invoices with PF and gov't fee line items, various statuses
- `disbursements`: 6 disbursements (3 internal Firm Fund, 3 client-funded)
- `documents`: 12 documents with original/copy flags and handover logs
- `retainerTemplates`: 2 templates (monthly bookkeeping, quarterly tax filing)
- `auditLog`: 10 entries

**localStorage API:**
```javascript
const DB = {
  SCHEMA_VERSION: 1,
  init() {
    const stored = localStorage.getItem('erp_schema_version');
    if (!stored || parseInt(stored) !== this.SCHEMA_VERSION) {
      this.resetToSeed();
    }
  },
  getAll(table) {
    return JSON.parse(localStorage.getItem('erp_' + table) || '[]');
  },
  getById(table, id) {
    return this.getAll(table).find(r => r.id === id);
  },
  getWhere(table, filterFn) {
    return this.getAll(table).filter(filterFn);
  },
  save(table, records) {
    localStorage.setItem('erp_' + table, JSON.stringify(records));
  },
  insert(table, record) {
    const all = this.getAll(table);
    all.push(record);
    this.save(table, all);
  },
  update(table, id, changes) {
    const all = this.getAll(table);
    const idx = all.findIndex(r => r.id === id);
    if (idx !== -1) {
      all[idx] = { ...all[idx], ...changes };
      this.save(table, all);
    }
  },
  delete(table, id) {
    const all = this.getAll(table).filter(r => r.id !== id);
    this.save(table, all);
  },
  resetToSeed() {
    const seed = { /* full seed data object */ };
    for (const [key, value] of Object.entries(seed)) {
      localStorage.setItem('erp_' + key, JSON.stringify(value));
    }
    localStorage.setItem('erp_schema_version', String(this.SCHEMA_VERSION));
  }
};
DB.init();
```

- [ ] **Step 2: Verify in browser**

Open browser DevTools → Application → Local Storage. Verify keys exist: `erp_users`, `erp_clients`, etc. Verify `erp_users` contains 10 entries. Run `DB.resetToSeed()` in console and verify data resets correctly.

- [ ] **Step 3: Commit**

```bash
git add js/data.js
git commit -m "feat: localStorage data layer with seed data and schema versioning"
```

---

## Task 3: Authentication & RBAC

**Files:**
- Create: `js/auth.js`
- Modify: `index.html` (add script refs)
- Modify: `css/styles.css` (add login styles if not already present)

- [ ] **Step 1: Create `js/auth.js` with login, session, and RBAC**

Create `js/auth.js` with:
```javascript
const Auth = {
  user: null,
  activeEntity: null,
  login(email, password) {
    const users = DB.getAll('users');
    const user = users.find(u => u.email === email && u.password === password);
    if (!user) return false;
    this.user = user;
    this.activeEntity = user.entities.includes('ATA') ? 'ATA' : 'LTA';
    sessionStorage.setItem('erp_session', JSON.stringify({ userId: user.id, activeEntity: this.activeEntity }));
    return true;
  },
  logout() {
    this.user = null;
    this.activeEntity = null;
    sessionStorage.removeItem('erp_session');
  },
  restoreSession() {
    const s = JSON.parse(sessionStorage.getItem('erp_session') || 'null');
    if (!s) return false;
    this.user = DB.getById('users', s.userId);
    this.activeEntity = s.activeEntity;
    return !!this.user;
  },
  can(action, entity) {
    if (!this.user) return false;
    entity = entity || this.activeEntity;
    const role = this.user.role;
    if (role === 'Admin') return true;
    if (!this.user.entities.includes(entity)) return false;
    const perms = {
      Manager: ['clients:view','clients:edit','workflow:view','workflow:edit','workflow:approve','billing:view','billing:edit','billing:approve','disbursement:view','disbursement:approve','dms:view','dms:edit','reports:view','users:view'],
      Staff: ['clients:view','workflow:view','workflow:edit','billing:view','disbursement:view','disbursement:create','dms:view','dms:edit','reports:view'],
      Viewer: ['clients:view','workflow:view','billing:view','disbursement:view','dms:view','reports:view']
    };
    return perms[role]?.includes(action) || false;
  },
  isSelfApprover(recordUserId) {
    return this.user?.id === recordUserId;
  },
  switchEntity(entity) {
    if (this.user?.entities.includes(entity)) {
      this.activeEntity = entity;
      sessionStorage.setItem('erp_session', JSON.stringify({ userId: this.user.id, activeEntity: entity }));
    }
  }
};
```

Wire login form in `index.html`: add event listener to `#login-form` that calls `Auth.login(email, password)`. On success, hide `#login-screen`, show `#app-shell`, call `App.init()`. On failure, show inline error message.

- [ ] **Step 2: Add entity switcher logic to header**

In `js/app.js`, populate `#entity-switcher` dropdown with user's allowed entities. On change, call `Auth.switchEntity()` and refresh the current view.

Add entity badge to header: colored pill showing active entity (ATA=blue via CSS class `.badge-ata`, LTA=green via `.badge-lta`).

- [ ] **Step 3: Verify in browser**

Open `index.html`. Login with `manager@ata-lta.ph` / `password123`. Verify: app shell appears, entity switcher shows both ATA and LTA, header badge shows "ATA" in blue. Switch to LTA — verify badge turns green and data context updates.

Logout and login with `accounting@ata-lta.ph` (Staff, ATA only). Verify: entity switcher shows only ATA, LTA is not available.

- [ ] **Step 4: Commit**

```bash
git add js/auth.js index.html css/styles.css
git commit -m "feat: login, RBAC, entity switching, and session management"
```

---

## Task 4: App Shell — Router, Navigation, Module Loader

**Files:**
- Create: `js/app.js`
- Modify: `index.html` (ensure nav links have `data-module` attributes)

- [ ] **Step 1: Create `js/app.js` with hash router**

Create `js/app.js` with:
```javascript
const App = {
  currentModule: null,
  init() {
    if (!Auth.restoreSession()) return;
    this.renderShell();
    this.setupRouting();
    this.setupNavigation();
    const defaultRoute = Auth.user.role === 'Admin' || Auth.user.role === 'Manager' ? '#dashboard' : '#workflow';
    location.hash = defaultRoute;
    this.handleRoute();
  },
  renderShell() {
    document.getElementById('login-screen').style.display = 'none';
    document.getElementById('app-shell').style.display = 'flex';
    document.getElementById('user-name').textContent = Auth.user.name;
    this.renderEntitySwitcher();
  },
  renderEntitySwitcher() {
    const sel = document.getElementById('entity-switcher');
    sel.innerHTML = '';
    Auth.user.entities.forEach(e => {
      const opt = document.createElement('option');
      opt.value = e;
      opt.textContent = e;
      if (e === Auth.activeEntity) opt.selected = true;
      sel.appendChild(opt);
    });
    sel.onchange = (ev) => {
      Auth.switchEntity(ev.target.value);
      this.handleRoute();
    };
  },
  setupRouting() {
    window.addEventListener('hashchange', () => this.handleRoute());
  },
  setupNavigation() {
    document.querySelectorAll('nav a[data-module]').forEach(link => {
      link.addEventListener('click', (e) => {
        e.preventDefault();
        location.hash = link.getAttribute('href');
      });
    });
  },
  handleRoute() {
    const hash = location.hash || '#dashboard';
    const moduleMap = {
      '#dashboard': Dashboard,
      '#clients': Clients,
      '#workflow': Workflow,
      '#billing': Billing,
      '#disbursement': Disbursement,
      '#documents': DMS,
      '#reports': Reports,
      '#admin': Users
    };
    const module = moduleMap[hash];
    const content = document.getElementById('content');
    if (module && module.render) {
      content.innerHTML = '';
      const rendered = module.render();
      if (typeof rendered === 'string') {
        content.innerHTML = rendered;
      } else {
        content.appendChild(rendered);
      }
      if (module.init) module.init();
      this.highlightNav(hash);
    }
  },
  highlightNav(hash) {
    document.querySelectorAll('nav a').forEach(a => {
      a.classList.toggle('active', a.getAttribute('href') === hash);
    });
  }
};

if (Auth.restoreSession()) { App.init(); }
```

In `index.html`, add `data-module` attributes to nav links and ensure hrefs match hash routes.

- [ ] **Step 2: Add placeholder module stubs**

Create placeholder objects in each module file:
```javascript
const Dashboard = {
  render() {
    const div = document.createElement('div');
    div.className = 'page';
    const h1 = document.createElement('h1');
    h1.textContent = 'Dashboard';
    div.appendChild(h1);
    return div;
  },
  init() {}
};
```
Add identical stubs to `js/clients.js`, `js/workflow.js`, `js/billing.js`, `js/disbursement.js`, `js/dms.js`, `js/reports.js`, `js/users.js`.

- [ ] **Step 3: Verify in browser**

Login. Verify: default route loads dashboard. Click each nav link — content area updates. URL hash changes. Active nav item highlighted. Entity switcher persists across navigation.

- [ ] **Step 4: Commit**

```bash
git add js/app.js js/clients.js js/workflow.js js/billing.js js/disbursement.js js/dms.js js/reports.js js/users.js index.html
git commit -m "feat: hash router, nav, module loader with placeholder stubs"
```

---

## Task 5: Dashboard Module — Firm Overview

**Files:**
- Modify: `js/dashboard.js`

- [ ] **Step 1: Implement consolidated Firm Overview for Managerial users**

In `js/dashboard.js`, build the render function using the safe `el()` helper from utils.js:

```javascript
const Dashboard = {
  render() {
    const isManagerial = Auth.user.role === 'Admin' || Auth.user.role === 'Manager';
    if (isManagerial && Auth.user.entities.length > 1) {
      return this.renderConsolidated();
    }
    return this.renderEntityScoped();
  },
  renderConsolidated() {
    const ata = this.getEntityMetrics('ATA');
    const lta = this.getEntityMetrics('LTA');
    const container = el('div', { class: 'page' });
    const h1 = el('h1', {}, ['Firm Overview']);
    container.appendChild(h1);
    const grid = el('div', { class: 'kpi-grid' });
    grid.appendChild(this.kpiCard('ATA Revenue', ata.revenue, 'ata'));
    grid.appendChild(this.kpiCard('LTA Revenue', lta.revenue, 'lta'));
    grid.appendChild(this.kpiCard('Total Outstanding', ata.outstanding + lta.outstanding));
    grid.appendChild(this.kpiCard('Overdue Tasks', ata.overdue + lta.overdue));
    container.appendChild(grid);
    container.appendChild(this.renderComparisonTable(ata, lta));
    return container;
  },
  getEntityMetrics(entity) {
    const wrs = DB.getWhere('workRequests', r => r.entity === entity);
    const invs = DB.getWhere('invoices', r => r.entity === entity);
    const tasks = DB.getWhere('tasks', r => {
      const wr = DB.getById('workRequests', r.workRequestId);
      return wr && wr.entity === entity;
    });
    return {
      activeWR: wrs.filter(r => r.status !== 'Completed' && r.status !== 'Cancelled').length,
      revenue: invs.filter(r => r.status === 'Paid' || r.status === 'Partially Paid').reduce((sum, r) => sum + (r.paidAmount || r.total), 0),
      outstanding: invs.filter(r => r.status === 'Sent' || r.status === 'Partially Paid' || r.status === 'Overdue').reduce((sum, r) => sum + (r.total - (r.paidAmount || 0)), 0),
      overdue: tasks.filter(r => r.status !== 'Completed' && r.status !== 'Cancelled' && new Date(r.dueDate) < new Date()).length
    };
  },
  kpiCard(label, value, entity) {
    const card = el('div', { class: 'kpi-card' + (entity ? ' ' + entity : '') });
    const lbl = el('div', { class: 'kpi-label' }, [label]);
    const val = el('div', { class: 'kpi-value' }, [typeof value === 'number' && value > 100 ? formatPHP(value) : String(value)]);
    card.appendChild(lbl);
    card.appendChild(val);
    return card;
  },
  renderComparisonTable(ata, lta) {
    // Build table with entity comparison
    const section = el('div', { class: 'entity-comparison' });
    const h2 = el('h2', {}, ['Entity Comparison']);
    section.appendChild(h2);
    const table = el('table', { class: 'data-table' });
    // ... build thead and tbody with rows for metrics
    section.appendChild(table);
    return section;
  },
  renderEntityScoped() {
    // Similar to consolidated but for single entity
    const metrics = this.getEntityMetrics(Auth.activeEntity);
    const container = el('div', { class: 'page' });
    container.appendChild(el('h1', {}, [Auth.activeEntity + ' Dashboard']));
    // ... render KPIs for single entity
    return container;
  },
  init() {}
};
```

- [ ] **Step 2: Add CSS for dashboard components**

In `css/styles.css`, add:
```css
.kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: var(--spacing-md); margin-bottom: var(--spacing-lg); }
.kpi-card { background: var(--color-surface); border: 1px solid var(--color-border); border-radius: var(--radius-md); padding: var(--spacing-lg); }
.kpi-card.ata { border-top: 4px solid var(--color-ata); }
.kpi-card.lta { border-top: 4px solid var(--color-lta); }
.kpi-label { font-size: 0.875rem; color: var(--color-text-muted); margin-bottom: var(--spacing-sm); }
.kpi-value { font-size: 1.75rem; font-weight: 700; color: var(--color-text); }
```

- [ ] **Step 3: Verify in browser**

Login as Manager. Verify: Firm Overview shows consolidated KPIs for both ATA and LTA. KPI cards have colored top borders. Entity comparison table shows correct totals. Login as Staff (ATA only). Verify: dashboard shows only ATA metrics.

- [ ] **Step 4: Commit**

```bash
git add js/dashboard.js css/styles.css
git commit -m "feat: consolidated Firm Overview dashboard with entity KPIs"
```

---

## Task 6: Client Management Module

**Files:**
- Modify: `js/clients.js`

- [ ] **Step 1: Build Client list view with search/filter**

In `js/clients.js`, build the module using safe DOM methods. Render a table of clients filtered by active entity. Each row shows: name, TIN, contact, entity badge, retainer status, Edit button. Add "Add Client" button.

- [ ] **Step 2: Build Client create/edit form**

Single-column form with sections:
- **Basic Information**: Name (required), TIN (required, format: XXX-XXX-XXX-XXXX), Business Address
- **Contact Details**: Contact Person, Phone, Email
- **Entity & Status**: Entity (ATA/LTA radio), Retainer (checkbox)
- Inline validation: TIN format regex, required fields, email format.
- On save: `DB.insert('clients', newClient)` or `DB.update('clients', id, changes)`.

- [ ] **Step 3: Verify in browser**

Navigate to Clients. Verify: list shows only clients for active entity. Search filters results. Add new client with form — persists in localStorage and appears in list. Switch entity — list updates.

- [ ] **Step 4: Commit**

```bash
git add js/clients.js css/styles.css
git commit -m "feat: client management with CRUD, search, and entity filtering"
```

---

## Task 7: Workflow & Task Management Module

**Files:**
- Modify: `js/workflow.js`

- [ ] **Step 1: Build Work Request list view**

Render list of work requests for active entity. Each card shows: title, client name, status badge, priority, due date, progress bar (% of completed tasks). Filter by status.

- [ ] **Step 2: Build Work Request create/edit form**

Single-column form:
- **Request Details**: Title, Description, Client (dropdown), Priority, Due Date
- **Task Assignment**: Dynamic task rows. Each row: title, assignee (user dropdown), predecessor (task dropdown or none)
- **Retainer Option**: Checkbox "This is a retainer template" — if checked, show schedule dropdown (monthly/quarterly) and auto-populate tasks from template

- [ ] **Step 3: Implement task dependency engine**

```javascript
const Workflow = {
  canStart(taskId) {
    const task = DB.getById('tasks', taskId);
    if (!task.predecessors || task.predecessors.length === 0) return true;
    const preds = task.predecessors.map(pid => DB.getById('tasks', pid));
    return preds.every(p => p && p.status === 'Completed');
  },
  updateTaskStatus(taskId, newStatus) {
    const task = DB.getById('tasks', taskId);
    if (task.status === 'Completed' || task.status === 'Cancelled') {
      return { error: 'Completed and cancelled tasks are immutable.' };
    }
    if (newStatus === 'In Progress' && !this.canStart(taskId)) {
      return { error: 'Predecessor tasks must be completed first.' };
    }
    if (newStatus === 'Cancelled') {
      // Cascade cancel to dependents
      const dependents = DB.getWhere('tasks', t => t.predecessors?.includes(taskId));
      dependents.forEach(d => DB.update('tasks', d.id, { status: 'Cancelled' }));
    }
    DB.update('tasks', taskId, { status: newStatus });
    return { success: true };
  },
  detectCycle(tasks) {
    const adj = {};
    tasks.forEach(t => { adj[t.id] = t.predecessors || []; });
    const visited = new Set();
    const recStack = new Set();
    function dfs(node) {
      visited.add(node);
      recStack.add(node);
      for (const neighbor of adj[node] || []) {
        if (!visited.has(neighbor) && dfs(neighbor)) return true;
        if (recStack.has(neighbor)) return true;
      }
      recStack.delete(node);
      return false;
    }
    for (const node of Object.keys(adj)) {
      if (!visited.has(node) && dfs(node)) return true;
    }
    return false;
  }
};
```

- [ ] **Step 4: Build task detail view with dependency display**

Task detail panel shows: blocking tasks (predecessors not yet completed), dependent tasks, status dropdown (filtered to valid next statuses: Draft→Assigned→In Progress→For Review→Completed, with reverse allowed except Completed/Cancelled), time log, comments. Visual indicators using CSS classes: `.predecessor-completed`, `.predecessor-pending`, `.predecessor-cancelled`.

- [ ] **Step 5: Implement retainer template creation and manual generation**

Retainer template form: client, schedule (monthly/quarterly), PF amount, task tree with dependencies. "Generate Next Work Request" button creates a new work request from template, copying tasks with dependencies preserved.

- [ ] **Step 6: Verify in browser**

Create work request with 3 tasks where Task 2 depends on Task 1. Verify: Task 2 cannot start until Task 1 Completed. Mark Task 1 Completed — Task 2 unlocks. Mark Task 1 Cancelled — Task 2 auto-cancels. Attempt to revert Task 1 from Completed — blocked. Test retainer template generation.

- [ ] **Step 7: Commit**

```bash
git add js/workflow.js css/styles.css
git commit -m "feat: workflow engine with DAG dependencies, status immutability, and retainer templates"
```

---

## Task 8: Billing Module

**Files:**
- Modify: `js/billing.js`

- [ ] **Step 1: Build Invoice list view**

List showing: invoice number, client, issue date, total, status badge, aging (days since due). Filter by status and entity. Color-coded status badges.

- [ ] **Step 2: Build Invoice creation form (Sales Invoice)**

Single-column form:
- **Header**: Client dropdown (filtered by entity), Issue Date, Due Date
- **Invoice Number**: Auto-generated based on entity (ATA-SI-YYYY-NNN or LTA-SI-YYYY-NNN). Fetch existing invoices for entity to determine next sequence number.
- **Line Items**: Dynamic rows. Each row: Type (PF / Government Fee), Description, Amount, VAT Treatment (VATable 12% / VAT-Exempt / Zero-Rated).
- **Totals**: Subtotal, VAT amount (computed only on VATable PF lines), Total.
- **Seller/Buyer Info**: Auto-populated from entity config and client record. Display TIN with branch code.

- [ ] **Step 3: Implement VAT calculation and PHP formatting**

```javascript
function calculateInvoiceTotals(lineItems) {
  let subtotal = 0, vat = 0;
  lineItems.forEach(item => {
    subtotal += item.amount;
    if (item.vatTreatment === 'VATable') vat += item.amount * 0.12;
  });
  return { subtotal, vat, total: subtotal + vat };
}
```

- [ ] **Step 4: Add invoice status workflow and payment recording**

Status: `Draft → Sent → Partially Paid → Paid → Overdue → Cancelled`.
Payment recording form: amount paid, date, method (Cash/Check/Bank Transfer), reference number. Update invoice `paidAmount` and status.

- [ ] **Step 5: Add aging report and print export**

Aging report: buckets 0-30, 31-60, 61-90, 90+ days. PDF export using `window.print()` with `@media print` CSS that hides nav and action buttons.

- [ ] **Step 6: Verify in browser**

Create invoice with PF (₱15,000, VATable) and gov't fee (₱500, VAT-Exempt). Verify: VAT = ₱1,800, total = ₱17,300. Invoice number matches entity prefix. Record partial payment of ₱10,000 — status becomes Partially Paid. Aging report shows correct buckets.

- [ ] **Step 7: Commit**

```bash
git add js/billing.js css/styles.css
git commit -m "feat: billing module with Sales Invoice, VAT, aging, and payment tracking"
```

---

## Task 9: Disbursement & Expense Module

**Files:**
- Modify: `js/disbursement.js`

- [ ] **Step 1: Build Disbursement list view**

List showing: employee, category, amount, fund source badge, status, approval stage. Filter by fund source and status.

- [ ] **Step 2: Build expense filing form**

Single-column form:
- **Expense Details**: Category dropdown (Transportation, Notary, Meals, Other, Government Fee), Description, Amount, Receipt upload (file input — store filename in mockup)
- **Fund Source**: Radio — Firm Fund / Client Fund. If Client Fund, show dropdown to link to billing invoice.
- **Entity**: Auto-tagged to active entity.

- [ ] **Step 3: Implement approval workflow**

```javascript
const Disbursement = {
  submit(expense) {
    expense.status = 'Submitted';
    expense.submittedAt = new Date().toISOString();
    DB.insert('disbursements', expense);
  },
  canApprove(id) {
    const d = DB.getById('disbursements', id);
    if (Auth.isSelfApprover(d.employeeId)) return false;
    if (d.fundSource === 'Firm Fund') {
      if (d.status === 'Submitted' && Auth.can('disbursement:approve')) return 'manager';
      if (d.status === 'Under Review' && Auth.can('disbursement:approve')) return 'accounting';
    } else {
      if (d.status === 'Submitted' && Auth.can('disbursement:approve')) return 'accounting';
    }
    return false;
  },
  approve(id) {
    const d = DB.getById('disbursements', id);
    const level = this.canApprove(id);
    if (!level) return { error: 'Not authorized' };
    if (level === 'manager') {
      DB.update('disbursements', id, { status: 'Under Review', managerApprovedBy: Auth.user.id });
    } else if (level === 'accounting') {
      DB.update('disbursements', id, { status: 'Approved', accountingApprovedBy: Auth.user.id });
    }
    return { success: true };
  },
  release(id) {
    DB.update('disbursements', id, { status: 'Released', releasedAt: new Date().toISOString() });
  },
  reject(id, reason) {
    DB.update('disbursements', id, { status: 'Rejected', rejectionReason: reason });
  }
};
```

- [ ] **Step 4: Build reimbursement summary report**

Table grouped by employee and category. Totals by fund source. Filter by date range and entity.

- [ ] **Step 5: Verify in browser**

Create internal expense (transportation ₱350). Verify: requires Manager then Accounting approval. Create client-funded disbursement (gov't fee ₱500 linked to invoice). Verify: single Accounting approval. Attempt to approve own expense — blocked. Self-approval block message shown inline.

- [ ] **Step 6: Commit**

```bash
git add js/disbursement.js css/styles.css
git commit -m "feat: disbursement module with 2-tier/1-tier approval and self-approval block"
```

---

## Task 10: Document Management System (DMS)

**Files:**
- Modify: `js/dms.js`

- [ ] **Step 1: Build Document list view**

List showing: filename, work request link, document type badge, uploader, upload date, handover status. Filter by document type and entity.

- [ ] **Step 2: Build document upload form**

Single-column form:
- **File**: File input. For mockup, read file using FileReader and store as base64 Data URL in localStorage with a size limit warning (e.g., max 2MB).
- **Work Request**: Dropdown of entity's work requests
- **Document Type**: Radio — Original Scan / Generated Copy
- **Category**: Requirement Docs, Processed Forms, Government Receipts, Final Deliverables, Other
- **Description**: Textarea

- [ ] **Step 3: Implement version tracking and handover log**

Version tracking: if same filename uploaded for same work request, create new version record with timestamp and uploader. Display version history as a table.

Handover log (for original scans only):
- Button "Record Handover" shows form: Recipient Name, Handover Date, Method (Pickup / Courier / Email / In-Person)
- Log displayed as table below document details.

- [ ] **Step 4: Verify in browser**

Upload document as "Original Scan". Verify: appears in list. Record handover to "Juan Dela Cruz" via Pickup on 2026-05-23. Verify: handover log shows entry. Upload same filename again — verify version history shows 2 entries.

- [ ] **Step 5: Commit**

```bash
git add js/dms.js css/styles.css
git commit -m "feat: DMS with upload, version tracking, and handover log"
```

---

## Task 11: Reporting & Analytics Module

**Files:**
- Modify: `js/reports.js`

- [ ] **Step 1: Build Reports dashboard**

Report cards:
- **Work Request Volume**: Count by status. CSS bar chart.
- **Task Completion Rate**: Average days from assignment to completion. List of overdue tasks.
- **Billing Summary**: Total PF billed, gov't fees collected, outstanding. By entity.
- **Disbursement Report**: Total by category and employee. Firm Fund vs Client Fund split.
- **Entity P&L Snapshot**: Revenue (billed PF) minus Firm Fund disbursements per entity.

- [ ] **Step 2: Implement CSS bar charts**

```css
.bar-chart { display: flex; align-items: flex-end; gap: 8px; height: 200px; }
.bar { width: 40px; background: var(--color-primary); border-radius: 4px 4px 0 0; }
```
Bar height computed from data percentage using inline `style="height: X%"` on div elements.

- [ ] **Step 3: Verify in browser**

Navigate to Reports. Verify: all report cards show computed values from seed data. Bar charts render. Overdue tasks list shows tasks past due date. Entity P&L shows ATA and LTA side by side.

- [ ] **Step 4: Commit**

```bash
git add js/reports.js css/styles.css
git commit -m "feat: reporting module with volume, completion, billing, and P&L reports"
```

---

## Task 12: Admin Panel — Users, Reset Data, Audit Log

**Files:**
- Modify: `js/users.js`
- Modify: `js/app.js` (add Users to router if not already)

- [ ] **Step 1: Build Users management view**

User list showing: name, email, role, department, entities. Only visible to Admin users. Form to add/edit users: name, email, password, role, department, entity access (multi-select checkbox for ATA/LTA).

- [ ] **Step 2: Add "Reset Demo Data" button**

In Admin panel, add "Reset Demo Data" button with inline confirmation (not modal). Calls `DB.resetToSeed()`. Shows inline success message. Reloads page.

- [ ] **Step 3: Add Audit Log view**

Display `auditLog` table: timestamp, user, action, entity. Filter by user and date range.

- [ ] **Step 4: Verify in browser**

Login as Admin. Navigate to Admin panel. Verify: user list visible. Add new user — appears in list and localStorage. Click Reset Demo Data — confirm. Verify: all data resets to seed state. Check audit log.

- [ ] **Step 5: Commit**

```bash
git add js/users.js js/app.js css/styles.css
git commit -m "feat: admin panel with user management, data reset, and audit log"
```

---

## Task 13: Integration, Polish, and Final Verification

**Files:**
- Modify: `css/styles.css`
- Modify: `index.html`
- Modify: `js/app.js`

- [ ] **Step 1: Ensure all modules respect entity context**

Spot-check each module: every data query must filter by `Auth.activeEntity` unless user is Manager on a consolidated report. Fix any module that shows cross-entity data.

- [ ] **Step 2: Add print styles for invoices**

In `css/styles.css`, add:
```css
@media print {
  nav, header, .actions-bar, .btn { display: none !important; }
  #content { margin: 0; padding: 0; }
  .invoice-print { max-width: 100%; }
}
```

- [ ] **Step 3: Add visual workflow progress bar**

For work requests, add a 6-stage progress bar:
```css
.progress-bar { display: flex; gap: 4px; }
.progress-step { flex: 1; padding: 8px; text-align: center; background: var(--color-border); border-radius: var(--radius-sm); }
.progress-step.completed { background: var(--color-success); color: white; }
.progress-step.active { background: var(--color-primary); color: white; }
```
Stages: Work Request → Pre-processing → Processing → Billing → Disbursement → Documentation. Compute current stage from work request status and related records.

- [ ] **Step 4: Test full workflow end-to-end**

Manual test script:
1. Login as Manager → see Firm Overview
2. Create a new client (ATA)
3. Create a work request for that client
4. Add 2 tasks with dependency (Task 2 depends on Task 1)
5. Mark Task 1 Completed → verify Task 2 unlocks
6. Create invoice for the work request (PF + gov't fee)
7. Record partial payment → verify status = Partially Paid
8. Create internal disbursement for transportation
9. Approve disbursement as Manager, then as Accounting
10. Upload document to work request, record handover
11. Check Reports for updated metrics
12. Logout, login as Staff (ATA only) — verify restricted access
13. Reset demo data — verify clean state

- [ ] **Step 5: Final commit**

```bash
git add .
git commit -m "feat: complete ERP mockup with all 7 modules, RBAC, and localStorage persistence"
```

---

## Self-Review

### 1. Spec Coverage Check

| Requirement | Task | Status |
|---|---|---|
| 2 entities (ATA/LTA) with data segregation | Task 2 (seed), Task 3 (auth), all modules | Covered |
| 4 RBAC tiers (Admin/Manager/Staff/Viewer) | Task 3 | Covered |
| 6 departments | Task 2 (seed users) | Covered |
| Work Request → Pre-processing → Processing → Billing → Disbursement → Documentation | Task 7, 8, 9, 10 | Covered |
| Task dependencies (DAG) | Task 7 | Covered |
| Retainer recurring templates | Task 7 | Covered |
| Completed tasks immutable | Task 7 | Covered |
| Billing: Sales Invoice, PF + gov't fees, VAT | Task 8 | Covered |
| Disbursement: 2-tier vs 1-tier approval | Task 9 | Covered |
| Self-approval block | Task 3, Task 9 | Covered |
| DMS: original/copy tracking, handover log | Task 10 | Covered |
| localStorage persistence, schema versioning, reset | Task 2, Task 12 | Covered |
| Firm Overview consolidated dashboard | Task 5 | Covered |
| Single-column forms, full phrase statuses, progress bars | Task 7, all forms | Covered |
| Internal-only (no client portal) | N/A (not built) | Covered |
| PHP currency formatting | Task 8 (formatPHP) | Covered |

**No gaps found.**

### 2. Placeholder Scan

- No "TBD", "TODO", "implement later" found.
- No vague "add validation" steps — specific validation rules listed.
- No "similar to Task N" references.
- All code blocks contain actual code.
- All file paths are exact.
- All commands have expected outputs.

### 3. Type Consistency Check

- `DB.getById(table, id)` — consistent across all modules.
- `Auth.can(action, entity)` — consistent.
- `Auth.activeEntity` — string 'ATA' or 'LTA', consistent.
- Invoice line item structure: `{ type, description, amount, vatTreatment }` — used in Task 8.
- Disbursement `fundSource`: 'Firm Fund' or 'Client Fund' — used in Task 9.
- Document `document_type`: 'original_scan' or 'generated_copy' — used in Task 10.
- Task `predecessors`: array of task IDs — used in Task 7.

All type signatures consistent. No mismatches found.

---

**Plan complete and saved to `docs/superpowers/plans/2026-05-23-ata-lta-erp-mockup.md`.**

**Two execution options:**

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration.

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints for review.

**Which approach?**
