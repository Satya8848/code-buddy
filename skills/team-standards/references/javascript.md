# JavaScript / Frontend Standards

- Formatter: Prettier (tabs, width 4, print width 110 for Frappe apps). Linter: ESLint.
- `const` by default, `let` when reassigned, never `var`.
- Use `async/await` with `frappe.call` / `frappe.xcall` instead of nested callbacks.
- No direct DOM manipulation where a Frappe form API exists (`frm.set_value`, `frm.toggle_display`, `frm.set_df_property`).
- Guard against undefined (`r.message || []`).
- No `eval`, no `innerHTML` with untrusted data — use `frappe.utils.escape_html`.
- Every user-visible string wrapped in `__()`.
- Vue (v15+): Vue 3 composition API for new components; Frappe UI for standalone portals.
- React Native / Expo apps: TypeScript, functional components + hooks, API base URL and keys from env config, never hardcoded.
- Node version: Frappe v14 → 16; v15 → 18+; v16 → 24+.
