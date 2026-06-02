# ERP Feedback v1 — Bug Fix & Polish Plan

**Date:** 2026-06-02  
**Status:** Fix Plan Complete — Awaiting Implementation Approval  
**Scope:** 19 reported discrepancies from teammate implementation  
**Files Affected:** `js/dashboard.js`, `js/clients.js`, `js/users.js`, `js/pendingChanges.js`, `js/billing.js`, `js/disbursement.js`, `js/workflow.js`, `js/reports.js`, `js/transmittal.js`, `js/dms.js`, `css/styles.css`, `js/app.js`  

---

## Summary of Issues

| # | Module | Issue | Root Cause |
|---|--------|-------|------------|
| 1 | Dashboard | WR Due for Week doesn't update | Widget only in `renderEntityScoped()`, missing from `renderConsolidated()` |
| 2 | Global UI | Save/Cancel not consistently top-right | Clients, Disbursement, Transmittal forms have save at bottom |
| 3 | Admin Gate | Staff rejected submissions disappear | `renderMyPendingSection()` only queries `status='pending'`; rejected section never rendered |
| 4 | Clients | Missing Related Companies & Contact Details columns | `renderList()` table headers don't include these fields |
| 5 | Billing | Invoice PDF format incorrect | `printVoucher()` uses generic layout, missing BIR-compliant sections |
| 6 | Billing & Disbursement | Payment details missing from list/board/card views | Table/board/list renderers don't include payment info columns |
| 7 | Billing | Voucher print overlay/CSS issues | Popup window styling is minimal; no `@media print` CSS for in-page voucher |
| 8 | Billing | Payment History table overflows card | 6-column table inside narrow `invoice-detail` container |
| 9 | Billing | Board view horizontally scrolls | `.board-view` CSS uses flex without proportional constraints |
| 10 | Operations | Board view horizontally scrolls | Same CSS issue as #9 |
| 11 | Disbursement | Who requested / who handled payment unclear | Detail view doesn't prominently show both roles; form lacks clarity |
| 12 | Disbursement & Transmittal | View toggle in wrong bar | View-mode toggle sits in `actions-bar` instead of `filters-bar` |
| 13 | Disbursement | No proper PDF generation | `openPrintVoucher()` exists but format is basic; needs structured voucher PDF |
| 14 | Disbursement | Can't create recurring template | `renderTemplates()` has no "New Template" button or form |
| 15 | Operations | Time logs not bound to task table rows | Time logs live only in `renderTaskActivity()` section below table |
| 16 | Operations | Documentation staff can't see all WRs | Staff filter blocks all non-assigned WRs; needs `dms:handover` exception |
| 17 | Reports | Month filter in Monthly Pending doesn't work | `type="month"` input may have browser compatibility/event issues |
| 18 | Operations | Admin can't view attached files | Task documents show metadata but no clickable link to open file |
| 19 | Operations | Task documents not in task table rows | Documents and comments isolated in Task Activity section |

---

## Implementation Roadmap

### Phase 1: Global UI Consistency & CSS Foundation
**Files:** `css/styles.css`, `js/app.js`  
**Issues:** #2, #9, #10, #12

| # | Task | File | Details |
|---|------|------|---------|
| 1.1 | Add `.board-view` no-scroll CSS | `css/styles.css` | `display:flex; gap:12px; width:100%; overflow-x:hidden;` on container. Columns: `flex:1; min-width:0; max-height:calc(100vh - 200px); overflow-y:auto; overflow-x:hidden;`. Cards: `word-wrap:break-word;` |
| 1.2 | Fix `getPreferredViewMode` | `js/app.js` | Line 182: `stored === 'list' || stored === 'grid' || stored === 'table' || stored === 'board'` — `grid` should not be a valid mode for modules that use table/board/list. Add validation per module. |
| 1.3 | Move save button to top-right | `js/clients.js`, `js/disbursement.js`, `js/transmittal.js` | Mirror Billing pattern: both Save and Cancel in `form-actions-top` inside `form-header-bar`. Remove duplicate from bottom. |
| 1.4 | Move view-mode toggle to filters bar | `js/disbursement.js`, `js/transmittal.js` | Relocate `view-mode-toggle` from `actions-bar` to `filters-bar` or below it. |

### Phase 2: Dashboard & Reports Fixes
**Files:** `js/dashboard.js`, `js/reports.js`  
**Issues:** #1, #17

| # | Task | File | Details |
|---|------|------|---------|
| 2.1 | Add widgets to consolidated dashboard | `js/dashboard.js` | Copy `Upcoming Disbursements` and `Work Requests Due This Week` cards from `renderEntityScoped()` into `renderConsolidated()`. Filter by active entity context or show combined. |
| 2.2 | Fix month filter in Reports | `js/reports.js` | Replace `type="month"` with a `<select>` dropdown of 12 months + current/previous year, or ensure `type="month"` event fires reliably across browsers. Add explicit `monthInput.addEventListener('input', ...)` in addition to `'change'`. |

### Phase 3: Admin Review Gate Fix
**Files:** `js/users.js`, `js/pendingChanges.js`  
**Issue:** #3

| # | Task | File | Details |
|---|------|------|---------|
| 3.1 | Show rejected submissions to Staff | `js/users.js` | In `renderMyPendingSection()`, add a second table for `PendingChanges.getRejectedForUser(Auth.user.id)`. Display rejection reason. Add "Resubmit" button that clones the snapshot into a new pending change. |
| 3.2 | Prevent accidental delete on reject | `js/users.js` | Line 527: `PendingChanges.delete(pc.id)` on Withdraw — keep this, but add confirmation. The bug is that REJECTED items aren't shown, not that they're deleted. |

### Phase 4: Clients Module Polish
**Files:** `js/clients.js`  
**Issue:** #4

| # | Task | File | Details |
|---|------|------|---------|
| 4.1 | Add Related Companies column | `js/clients.js` | In `renderList()` table headers, add `Related Companies`. Render as comma-separated relation labels or "—" if empty. |
| 4.2 | Add Contact Details column | `js/clients.js` | In `renderList()` table headers, add `Contact Details`. Render as comma-separated `type: value` pairs or "—". |

### Phase 5: Billing Module — PDFs, Vouchers, Payments, Layout
**Files:** `js/billing.js`, `css/styles.css`  
**Issues:** #5, #6, #7, #8

| # | Task | File | Details |
|---|------|------|---------|
| 5.1 | Rewrite `printVoucher()` for BIR-compliant invoice PDF | `js/billing.js` | Per NotebookLM: include Seller header (name, TIN+branch, address), Buyer info (name, TIN, address for B2B ≥₱1,000), line items with qty/unit cost/total, subtotal, **no VAT line** (hidden), `total = subtotal`, and footer: *"This document is not valid for claim of input tax."* Use `window.open()` with inline print stylesheet. |
| 5.2 | Create voucher-format print (no company details) | `js/billing.js` | New method `printVoucherNoHeader()` — strips seller header/logo, keeps itemized list + amount + pass-through disclaimer. For government fee pass-throughs. |
| 5.3 | Add `@media print` CSS for in-page voucher | `css/styles.css` | Hide nav, header, actions. Show only `.invoice-detail` content. `.bir-footer` display becomes `block` in print. Prevent overlay. |
| 5.4 | Show payment details in list/board views | `js/billing.js` | `refreshTable()`: add `Paid` and `Balance` columns. `refreshBoard()`: show `balance` on card. `refreshListCompact()`: show paid amount. |
| 5.5 | Fix Payment History table overflow | `js/billing.js` | Wrap payment history table in a `div` with `overflow-x:auto;` or reduce columns. Condense `Recorded By` + `Collected By` into one "Processor" column if space is tight. |
| 5.6 | Add payment details to billing card view | `js/billing.js` | In `refreshBoard()`, show `Paid: X / Total: Y` on each card. |

### Phase 6: Disbursement Module — Completeness
**Files:** `js/disbursement.js`, `css/styles.css`  
**Issues:** #6, #11, #12, #13, #14

| # | Task | File | Details |
|---|------|------|---------|
| 6.1 | Clarify requester vs payment handler in detail | `js/disbursement.js` | In `renderDetail()`, add explicit labeled rows: **"Requested By"** (employee name) and **"Payment Handled By"** (from release dialog). Show in a dedicated meta block. |
| 6.2 | Show payment details in list/board views | `js/disbursement.js` | `renderTableView()`: add `Payment Method` and `Handled By` columns for Released items. `renderBoardView()`: show method on card if released. |
| 6.3 | Implement proper disbursement PDF/voucher | `js/disbursement.js` | Per NotebookLM: Voucher No, Date of Request, Date of Disbursement, Payee/Vendor, Payment Method, Description, GL Account (optional), Client/Project Code, Amount, "Prepared By" + "Approved By" signature lines. Use `window.open()` with structured table layout. |
| 6.4 | Add "New Disbursement Template" form | `js/disbursement.js` | In `renderTemplates()`, add a **"New Template"** button that shows an inline form: Name, Category, Amount, Fund Source, Schedule, Description. On save, insert into `disbursementTemplates`. |
| 6.5 | Move view toggle to filters bar | `js/disbursement.js` | Done in Phase 1.4. |

### Phase 7: Operations Module — Task Documents, Time Logs, Visibility
**Files:** `js/workflow.js`  
**Issues:** #10, #15, #16, #18, #19

| # | Task | File | Details |
|---|------|------|---------|
| 7.1 | Fix board view CSS | `css/styles.css` | Same as Phase 1.1 — proportional flex columns, internal scroll. |
| 7.2 | Documentation staff sees all WRs | `js/workflow.js` | In `renderList()` Staff filter (line 114-118), add exception: if `Auth.can('dms:handover')`, show all WRs in entity. But disable Add/Edit buttons for non-managers. |
| 7.3 | Inline task documents in task table | `js/workflow.js` | In `renderDetail()` task table, each `<tr>` gets a second `<tr class="task-doc-row">` below it with colspan. This row shows: document metadata + expandable comment thread (if any). Only visible if task has documents. |
| 7.4 | Admin can view attached files | `js/workflow.js` | In the inline document row, add an `<a>` link that opens `URL.createObjectURL()` or shows the file metadata. Since we only store metadata (not Base64), add a note: "File available in DMS" with link to `#documents` filtered by task. Alternatively, if the file was uploaded via DMS, cross-link to the DMS record. |
| 7.5 | Inline time logs in task table | `js/workflow.js` | Add an expandable row per task that shows today's time log (if any). If no log today, show encouraging message: *"No time logged today — log your progress!"* with a quick-add button. |
| 7.6 | Admin comments on task documents | `js/workflow.js` | In the collapsible document row, add a small comment form (Admin-only) and thread. Use the existing `task.comments` schema. Comments should be scoped to the task (not individual documents) for simplicity. |

### Phase 8: DMS & Transmittal Minor Fixes
**Files:** `js/dms.js`, `js/transmittal.js`  
**Issues:** #12 (Transmittal part)

| # | Task | File | Details |
|---|------|------|---------|
| 8.1 | Move Transmittal view toggle to filters bar | `js/transmittal.js` | Done in Phase 1.4. |

---

## Detailed Technical Specifications

### Board View CSS (Fixes #9, #10)

Per NotebookLM research, the board view must:
1. Use `display: flex` on the container
2. Give each column `flex: 1; min-width: 0;` (proportional, no expanding)
3. Set `max-height` on columns with `overflow-y: auto` (internal vertical scroll)
4. Prevent card text overflow with `word-wrap: break-word`

```css
.board-view {
  display: flex;
  gap: var(--spacing-md);
  width: 100%;
  overflow-x: hidden;
}

.board-column {
  flex: 1;
  min-width: 0;
  max-height: calc(100vh - 240px);
  overflow-y: auto;
  overflow-x: hidden;
  background: var(--color-bg);
  border-radius: var(--radius-md);
  padding: var(--spacing-sm);
}

.board-card {
  word-wrap: break-word;
  overflow-wrap: break-word;
}
```

### BIR-Compliant Invoice PDF (Fix #5)

Per NotebookLM / EOPT Act:
- **Header:** Seller registered name, business address, TIN + 4-digit branch code
- **Document:** Sequential serial number (invoiceNumber), date
- **Buyer:** Name, address, TIN (for B2B ≥₱1,000)
- **Line items:** Description, quantity (default 1 for services), unit cost, total cost
- **No VAT breakdown** — but footer must state: *"This document is not valid for claim of input tax."*
- **Totals:** Subtotal = Total (since VAT = 0)

### Disbursement Voucher PDF (Fix #13)

Per NotebookLM:
- **Header:** Voucher Number, Date of Request, Date of Disbursement
- **Payee:** Employee name or Vendor
- **Payment Method:** Cash, Check, Bank Transfer, GCash
- **Expense Details:** Description, Category, Amount
- **Summary:** Total Amount
- **Authorizations:** "Prepared By" name/timestamp + "Approved By" signature line
- **For pass-through:** Strip firm branding, show only payee + amount + "Reimbursable pass-through cost"

### Task Row Expansion (Fix #15, #19)

In the Operations detail task table, each task `<tr>` will be followed by an optional expansion row:

```html
<tr class="task-main">
  <td>Task title</td>...
</tr>
<tr class="task-expand">
  <td colspan="6">
    <div class="task-expand-inner">
      <!-- Documents -->
      <!-- Time log for today -->
      <!-- Comment thread -->
    </div>
  </td>
</tr>
```

---

## Testing Checklist

- [ ] Dashboard consolidated view shows both widgets
- [ ] All form Save/Cancel buttons consistently at top-right
- [ ] Staff sees rejected submissions with reason and can resubmit
- [ ] Clients table shows Related Companies and Contact Details
- [ ] Billing invoice print includes BIR footer
- [ ] Billing voucher print strips seller header
- [ ] Payment History table doesn't overflow on 1366×768
- [ ] Billing/Operations board views fit without horizontal scroll
- [ ] Disbursement list shows payment method and handler for released items
- [ ] Disbursement PDF/voucher has proper format
- [ ] New Disbursement Template form creates templates
- [ ] Documentation staff sees all WRs but can't add/edit
- [ ] Task table rows expand to show documents, time logs, comments
- [ ] Admin can comment on task documents
- [ ] Reports Monthly Pending month filter updates correctly
- [ ] Transmittal view toggle sits below filters

---

## Do Not Proceed

**Awaiting explicit approval before beginning implementation.**
