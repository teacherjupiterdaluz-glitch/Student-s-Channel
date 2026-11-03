---

## 2. Technical `docs/ARCHITECTURE.md`
*(Save in `docs/ARCHITECTURE.md`)*

```markdown
# Collab LMS — Technical Architecture Document

## 1. Architectural Patterns

### 1.1 Service-Oriented File Architecture (SOFA)
Google Apps Script runs all `.gs` files in a flat global namespace without ES modules. To prevent naming collisions and file loading order bugs:
1. Files are prefixed numerically (`01_`, `02_`, etc.) so core dependencies load first.
2. Business logic is organized into single-responsibility services.
3. Private functions end with a trailing underscore (`_`) to prevent client exposure via `google.script.run`.

### 1.2 In-Memory Repository Pattern (`02_DbRepository.gs`)
Opening Google Sheets via `SpreadsheetApp.openById()` incurs significant network latency (~800ms–1500ms).
- `02_DbRepository.gs` implements request-level memoization via `_tableRecordsCache`.
- A tab is read **once** into memory during an execution context. Subsequent queries in that same execution resolve in **0ms**.
- Multi-row updates are committed atomically in memory and flushed to disk using `SpreadsheetApp.flush()` within an exclusive `LockService` block.

### 1.3 The Composite Payload / Single Check Roll (`04_BootstrapService.gs`)
To prevent client-side network waterfalls, the client requests `getStudentBootstrapContext(token)` on boot. 
The server collects:
- Student identity, level (CEFR), and professional track.
- Next scheduled class and Google Meet link (14-day calendar window).
- Open/completed PDP deliverables.
- Unread feedback counts and Evolution status.
- Contextual Ebook bookmark ("Never Be Lost").

---

## 2. Client-Side Lifecycle & Sandboxed Execution

### 2.1 The Two-Window Architecture
Apps Script Web Apps run inside a sandboxed `<iframe>` on `*.googleusercontent.com` wrapped by `script.google.com`.
- Form actions must use `onsubmit="return false;"` and buttons must use `type="button"`.
- Forms must never submit via native `GET` or use `action="javascript:void(0)"`, which triggers cross-origin frame navigation security violations.

### 2.2 Component Gating (Custom Elements)
The application relies on Web Components (Cally Calendar `<calendar-date>`).
Initialization logic must wait for custom elements to upgrade:
```javascript
Promise.all([
  customElements.whenDefined('calendar-date'),
  customElements.whenDefined('calendar-month')
]).then(() => {
  restoreWorkspace();
});
