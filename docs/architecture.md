# {{project_name}}: Architecture and Design Document

**Version:** 0.0.1
**Audience:** Python-first web developers building server-rendered interfaces with HTMX
**Authoring mode:** HTML-first, progressive enhancement, copy and paste components

---

## 1. Purpose

{{project_name}} is an HTML-first component library that delivers copy and paste snippets powered by HTMX for rapid UI construction. The library targets Python frameworks first (Django, FastAPI, Flask), while remaining backend agnostic. The goal is to provide a set of accessible, unopinionated building blocks that work without a JavaScript bundler, can be themed with design tokens, and define predictable server contracts.

## 2. Goals

1. Copy and paste components that work out of the box with HTMX, no bundler required.
2. First-class documentation and adapters for Django, FastAPI, and Flask.
3. Accessible by default, including keyboard flows, focus management, and ARIA roles.
4. Theming through CSS variables and an optional Tailwind preset. No lock-in.
5. Predictable server contracts: each component declares expected endpoints, inputs, events, and return shapes.
6. Ownership of source: the CLI copies files into the application so teams can modify freely.

## 3. Non-Goals

1. A full reactive component framework. {{project_name}} enhances server-rendered HTML instead of replacing it.
2. Shipping a runtime JavaScript framework. The baseline is HTMX and optional Hyperscript.
3. Locking teams into a specific CSS utility framework. Tailwind support is an add-on.

## 4. Core Principles

* **HTML first:** render everything on the server, apply interactivity through HTMX attributes.
* **Progressive enhancement:** components function without client scripting, then enhance behavior when HTMX is present.
* **Accessibility by default:** roles, names, focus management, and keyboard interaction are designed in from the start.
* **Stable contracts:** the markup API and server contract are explicit and versioned.
* **Learn by viewing source:** every example page exposes the exact HTML that gets pasted.

---

## 5. High-Level Architecture

### 5.1 Package Layout

```
{{package_name}}/
  packages/
    {{package_name}}-core/                # CSS tokens, resets, low-level utilities
    {{package_name}}-components/          # Copy and paste HTML snippets with HTMX attributes
    {{package_name}}-hyperscript/         # Optional interactions as <script type="text/hyperscript">
    {{package_name}}-tailwind-preset/     # Optional Tailwind preset mapped to CSS variables
    adapters/
      {{package_name}}-django/            # Template tags, CSRF helpers, messages-to-toasts, pagination helpers
      {{package_name}}-fastapi/           # Template utilities, partial helpers, HX-Trigger helpers
      {{package_name}}-flask/             # Blueprint with helpers matching the FastAPI adapter
  tools/
    {{package_name}}-cli/                 # `{{package_name}} add <component>` copies component files into the app
  docs/
    site/                        # Documentation site with live examples and source view
  examples/
    fastapi-demo/
    django-demo/
    flask-demo/
```

### 5.2 Runtime Overview

* Components are HTML fragments with HTMX attributes and CSS classes.
* The server returns partial templates that replace targets or apply out of band swaps.
* Optional Hyperscript snippets handle focus, keyboard navigation, and small behaviors without a bundler.
* The adapters provide ergonomics for CSRF, flash messages, and HTMX-aware rendering.

### 5.3 Data Flow

1. A user action triggers an HTMX request via `hx-get` or `hx-post`.
2. The server endpoint validates input, executes domain logic, and returns an HTML fragment.
3. HTMX swaps the fragment into the target element. Optional out of band fragments update shared zones such as a toast container.
4. Optional custom events inform other components about state changes.

---

## 6. Styling and Theming

### 6.1 Design Tokens

A small set of CSS variables controls colors, spacing, radii, shadows, and transitions. These are defined in `{{package_name}}-core` and can be overridden on `:root` or on a theme scope.

```css
:root {
  --{{package_name}}-color-background: #0b0b0c;
  --{{package_name}}-color-foreground: #e8e8ea;
  --{{package_name}}-color-muted: #a0a0a7;
  --{{package_name}}-color-accent: #6aa1ff;
  --{{package_name}}-radius-medium: 12px;
  --{{package_name}}-shadow-1: 0 1px 2px rgba(0,0,0,.2);
  --{{package_name}}-shadow-2: 0 8px 30px rgba(0,0,0,.35);
  --{{package_name}}-spacing-1: .25rem;
  --{{package_name}}-spacing-2: .5rem;
  --{{package_name}}-spacing-3: .75rem;
  --{{package_name}}-spacing-4: 1rem;
  --{{package_name}}-focus-ring: 0 0 0 3px rgba(106,161,255,.5);
}
```

### 6.2 Utility Classes

`{{package_name}}-core` includes a light utility sheet for common layout patterns: stack, cluster, grid, visually hidden, and focus ring helpers. Teams can keep these or replace them.

### 6.3 Tailwind Preset

`{{package_name}}-tailwind-preset` maps the CSS variables to Tailwind theme tokens. Teams using Tailwind can opt in without rewriting components.

---

## 7. Component Specification Format

Each component ships with:

* `component.html` (trigger markup that lives in your page)
* `component.partial.html` (server-rendered fragment that gets returned by endpoints)
* `component.hyperscript.html` (optional behavior)
* `README.md` (contract and accessibility guidance)

Each component must document:

* **Inputs:** data attributes, form fields, and ARIA attributes.
* **Endpoints:** routes, HTTP methods, expected parameters, and return fragments.
* **Events:** custom events emitted or consumed.
* **Accessibility:** roles, names, keyboard interactions, and focus management.
* **States:** loading, error, empty, and success visuals.
* **Theming hooks:** tokens and classes involved.

### 7.1 Example: Modal Component

**Trigger Markup**

```html
<button
  class="{{package_name}}-button"
  hx-get="/modal/example"
  hx-target="#modal-root"
  hx-trigger="click"
  hx-swap="innerHTML">
  Open Modal
</button>

<div id="modal-root" aria-live="polite"></div>
```

**Returned Partial**

```html
<div
  id="{{package_name}}-modal"
  role="dialog"
  aria-modal="true"
  aria-labelledby="{{package_name}}-modal-title"
  class="{{package_name}}-modal"
  _="on load call #{{package_name}}-modal.focus()">
  <div class="{{package_name}}-modal__backdrop"
       hx-trigger="click"
       hx-get="/modal/close"
       hx-target="#modal-root"
       hx-swap="innerHTML"></div>

  <div class="{{package_name}}-modal__panel" tabindex="-1">
    <header class="{{package_name}}-modal__header">
      <h2 id="{{package_name}}-modal-title" class="{{package_name}}-heading-2">Example modal</h2>
      <button class="{{package_name}}-icon-button"
              aria-label="Close"
              hx-get="/modal/close"
              hx-target="#modal-root"
              hx-swap="innerHTML">×</button>
    </header>

    <div class="{{package_name}}-modal__body">
      <p>Content here</p>
      <form
        hx-post="/modal/submit"
        hx-target="#modal-root"
        hx-swap="innerHTML">
        <input name="email" type="email" required class="{{package_name}}-input" />
        <button class="{{package_name}}-button {{package_name}}-button--accent" type="submit">Submit</button>
      </form>
    </div>
  </div>
</div>
```

**Server Endpoints**

* `GET /modal/example` returns the modal partial
* `GET /modal/close` returns an empty string
* `POST /modal/submit` returns a success partial or validation errors

**Accessibility**

* Focus sent to the panel on open, Escape closes via a visible close button or an optional key handler in Hyperscript.
* Backdrop has a descriptive label on the dialog container, not on the backdrop itself.

---

## 8. Python Adapters

### 8.1 Django Adapter

* Template tag `{% {{package_name}}_toast_container %}` injects a single toast root.
* Template tag `{% {{package_name}}_csrf_headers %}` prints `hx-headers` for CSRF.
* Middleware converts `django.contrib.messages` into out of band toast fragments.
* Pagination helper generates `hx-get` links targeting a table body.

### 8.2 FastAPI Adapter

* `HTMXPartial` response helper for fragments, with optional event triggers via `HX-Trigger` headers.
* Jinja2 template utilities for deciding layout versus partial based on the `HX-Request` header.

### 8.3 Flask Adapter

* Blueprint with helpers matching FastAPI semantics.

---

## 9. CLI

### 9.1 Purpose

The CLI copies component files into the application, initializes tokens and utilities, and optionally installs a Tailwind preset.

### 9.2 Commands

```
{{package_name}} add <component-name>
{{package_name}} add table --features pagination,sorting
{{package_name}} theme init --preset minimal
{{package_name}} doctor
```

### 9.3 Behavior

* Verifies destination paths and writes files under `templates/{{package_name}}` and `static/{{package_name}}` by default.
* Never hides files. The application owns and edits all copied files.
* Prints follow-up instructions for wiring endpoints and optional adapter helpers.

---

## 10. Events and Swap Conventions

* Loading indicators: use `hx-indicator=".{{package_name}}-loading"` on a component root that contains a spinner element.
* Error handling: fragments can mark error roots with `data-{{package_name}}-error="true"` so styles can switch to an error state.
* Out of band zones: `#{{package_name}}-toasts` for toasts, `#{{package_name}}-dialog-root` for modals, `#{{package_name}}-announcer` for live regions.
* Custom events: components may emit `{{package_name}}:success`, `{{package_name}}:error`, or more specific events.

---

## 11. Accessibility Requirements

* All interactive elements are keyboard reachable and operable.
* Dialogs manage focus and restore focus to the trigger on close.
* Menus and lists use roving tabindex when appropriate.
* All form controls have programmatic names and visible labels.
* Announce dynamic updates through `aria-live` regions when content changes non-modally.

---

## 12. Security Considerations

* CSRF documentation per framework, plus helpers for headers on HTMX requests.
* Validation on server endpoints for state-changing requests, even when triggered by HTMX.
* Rate limits for high-frequency triggers, such as typeahead and command palette.
* Clear guidance on safely rendering untrusted content in fragments.

---

## 13. Versioning and Stability

* Semantic versioning for all packages.
* `{{package_name}}-components` treats any change to HTML structure, required attributes, or data attributes as a breaking change that bumps the major version.
* Adapters maintain minor version compatibility within a major series.

---

## 14. Testing and Quality

* Example servers in `examples/` for FastAPI, Django, and Flask, each rendering every component.
* Playwright-based snapshot tests for visual regressions.
* `axe-core` automated accessibility checks on example pages.
* Contract tests that render fragments with sample context to validate required fields and states.

---

## 15. Initial Component Catalog

Each entry includes a summary, expected endpoints, events, and accessibility notes. The manifest below encodes these details for scaffolding.

### 15.1 Buttons

* **Summary:** Primary, secondary, icon, and split buttons.
* **Endpoints:** None, buttons act as triggers.
* **Events:** Emit `{{package_name}}:click` as an optional custom event.
* **Accessibility:** Visible focus, `aria-pressed` for toggle variants.

### 15.2 Inputs

* **Summary:** Text input, textarea, select, and a combobox pattern.
* **Endpoints:** Validation endpoint optional.
* **Events:** `{{package_name}}:validate` optional.
* **Accessibility:** Labeled controls, `aria-describedby` for help text and errors.

### 15.3 Modal

* **Summary:** Dialog with focus trap, backdrop, and close controls.
* **Endpoints:** `GET /modal/example`, `GET /modal/close`, `POST /modal/submit`.
* **Events:** `{{package_name}}:modal:open`, `{{package_name}}:modal:close`.
* **Accessibility:** `role="dialog"`, `aria-modal="true"`, managed focus.

### 15.4 Drawer

* **Summary:** Edge panel for settings and filters.
* **Endpoints:** Open, close, and optional save.
* **Events:** `{{package_name}}:drawer:open`, `{{package_name}}:drawer:close`.
* **Accessibility:** `role="dialog"` with labeled header.

### 15.5 Dropdown Menu

* **Summary:** Button-triggered menu with keyboard navigation.
* **Endpoints:** None required, content is server-rendered or static.
* **Events:** `{{package_name}}:menu:select`.
* **Accessibility:** `role="menu"`, roving tabindex, arrow key navigation.

### 15.6 Tabs

* **Summary:** Tablist with per-tab content via `hx-get`.
* **Endpoints:** `GET /tabs/<tab>`.
* **Events:** `{{package_name}}:tab:change`.
* **Accessibility:** `role="tablist"`, `role="tab"`, `aria-controls`, and `aria-selected`.

### 15.7 Table

* **Summary:** Sortable, pageable table with row actions.
* **Endpoints:** `GET /table?page=<n>&sort=<field>:<dir>`.
* **Events:** `{{package_name}}:table:sorted`, `{{package_name}}:table:paged`.
* **Accessibility:** Proper table semantics, captions, and summaries.

### 15.8 Toasts

* **Summary:** Global toast queue using out of band swaps.
* **Endpoints:** Any server event can return out of band toast fragments.
* **Events:** `{{package_name}}:toast`.
* **Accessibility:** `aria-live="assertive"` with timeouts announced.

### 15.9 Command Palette

* **Summary:** Input with server-backed results and keyboard selection.
* **Endpoints:** `POST /palette/search`.
* **Events:** `{{package_name}}:palette:select`.
* **Accessibility:** `role="dialog"` with listbox semantics for results.

### 15.10 Stepper

* **Summary:** Multi-step wizard with back and next actions.
* **Endpoints:** `GET /stepper/<step>`, `POST /stepper/<step>`.
* **Events:** `{{package_name}}:stepper:change`.
* **Accessibility:** Announce step changes and disabled states.

### 15.11 Infinite List

* **Summary:** Lazy loading list using `hx-trigger="revealed"` on a sentinel.
* **Endpoints:** `GET /list?cursor=<id>`.
* **Events:** `{{package_name}}:list:append`.
* **Accessibility:** Maintain position, announce new content.

### 15.12 Form with Validation Pattern

* **Summary:** Server-validated form that replaces only invalid groups.
* **Endpoints:** `POST /form/validate`, `POST /form/submit`.
* **Events:** `{{package_name}}:form:invalid`, `{{package_name}}:form:submitted`.
* **Accessibility:** Field-level error messages with `aria-describedby`.

---

## 16. Scaffolding Manifest

The manifest declares the component catalog, file outputs, and contracts. The CLI consumes this file to scaffold a project or add components.

```yaml
# {{package_name}}.manifest.yaml
version: 1
library:
  name: "{{project_name}}"
  description: "HTML-first, HTMX-powered copy and paste components for Python-first apps"
  packages:
    - {{package_name}}-core
    - {{package_name}}-components
    - {{package_name}}-hyperscript
    - {{package_name}}-tailwind-preset
    - {{package_name}}-django
    - {{package_name}}-fastapi
    - {{package_name}}-flask
  tokens_file: "static/{{package_name}}/{{package_name}}-core.css"
  docs_site: "docs/site"

components:
  - key: button
    title: "Buttons"
    summary: "Primary, secondary, icon, and split buttons"
    files:
      - templates/{{package_name}}/button.html
      - static/{{package_name}}/button.css
      - docs/components/button.md
    endpoints: []
    events:
      emits: ["{{package_name}}:click"]
      listens: []
    accessibility:
      notes: ["Visible focus", "aria-pressed for toggle variants"]

  - key: input
    title: "Inputs"
    summary: "Text input, textarea, select, and combobox"
    files:
      - templates/{{package_name}}/input.html
      - static/{{package_name}}/input.css
      - docs/components/input.md
    endpoints:
      - method: POST
        path: "/input/validate"
        description: "Optional server validation for demo purposes"
    events:
      emits: ["{{package_name}}:validate"]
      listens: []
    accessibility:
      notes: ["Labels required", "aria-describedby for help and errors"]

  - key: modal
    title: "Modal"
    summary: "Dialog with focus trap and backdrop"
    files:
      - templates/{{package_name}}/modal.html
      - templates/{{package_name}}/modal.partial.html
      - templates/{{package_name}}/modal.hyperscript.html
      - static/{{package_name}}/modal.css
      - docs/components/modal.md
    endpoints:
      - method: GET
        path: "/modal/example"
        description: "Return the modal partial"
      - method: GET
        path: "/modal/close"
        description: "Close the modal by clearing the root"
      - method: POST
        path: "/modal/submit"
        description: "Handle form submit and return success or errors"
    events:
      emits: ["{{package_name}}:modal:open", "{{package_name}}:modal:close"]
      listens: []
    accessibility:
      notes: ["role=dialog with aria-modal", "focus sent to panel on open"]

  - key: drawer
    title: "Drawer"
    summary: "Edge panel for settings and filters"
    files:
      - templates/{{package_name}}/drawer.html
      - templates/{{package_name}}/drawer.partial.html
      - static/{{package_name}}/drawer.css
      - docs/components/drawer.md
    endpoints:
      - method: GET
        path: "/drawer/open"
      - method: GET
        path: "/drawer/close"
    events:
      emits: ["{{package_name}}:drawer:open", "{{package_name}}:drawer:close"]
      listens: []
    accessibility:
      notes: ["role=dialog", "label with heading"]

  - key: dropdown
    title: "Dropdown Menu"
    summary: "Button-triggered menu with keyboard navigation"
    files:
      - templates/{{package_name}}/dropdown.html
      - static/{{package_name}}/dropdown.css
      - docs/components/dropdown.md
    endpoints: []
    events:
      emits: ["{{package_name}}:menu:select"]
      listens: []
    accessibility:
      notes: ["role=menu with roving tabindex", "arrow key navigation"]

  - key: tabs
    title: "Tabs"
    summary: "Tablist with per-tab content"
    files:
      - templates/{{package_name}}/tabs.html
      - templates/{{package_name}}/tabs.partial.html
      - static/{{package_name}}/tabs.css
      - docs/components/tabs.md
    endpoints:
      - method: GET
        path: "/tabs/{tabKey}"
        description: "Return content for a given tab key"
    events:
      emits: ["{{package_name}}:tab:change"]
      listens: []
    accessibility:
      notes: ["role=tablist and role=tab", "aria-selected and aria-controls"]

  - key: table
    title: "Table"
    summary: "Sortable, pageable table with row actions"
    files:
      - templates/{{package_name}}/table.html
      - templates/{{package_name}}/table.partial.html
      - static/{{package_name}}/table.css
      - docs/components/table.md
    endpoints:
      - method: GET
        path: "/table"
        description: "Accepts page and sort query parameters"
    events:
      emits: ["{{package_name}}:table:sorted", "{{package_name}}:table:paged"]
      listens: []
    accessibility:
      notes: ["caption recommended", "scope attributes on headers"]

  - key: toast
    title: "Toasts"
    summary: "Global toast queue using out of band swaps"
    files:
      - templates/{{package_name}}/toast.root.html
      - templates/{{package_name}}/toast.item.html
      - static/{{package_name}}/toast.css
      - docs/components/toast.md
    endpoints: []
    events:
      emits: ["{{package_name}}:toast"]
      listens: []
    accessibility:
      notes: ["aria-live=assertive on root", "dismiss buttons with labels"]

  - key: palette
    title: "Command Palette"
    summary: "Search input with server-backed results"
    files:
      - templates/{{package_name}}/palette.html
      - templates/{{package_name}}/palette.partial.html
      - static/{{package_name}}/palette.css
      - docs/components/palette.md
    endpoints:
      - method: POST
        path: "/palette/search"
        description: "Return result list items"
    events:
      emits: ["{{package_name}}:palette:select"]
      listens: []
    accessibility:
      notes: ["role=dialog with listbox semantics", "announce result counts"]

  - key: stepper
    title: "Stepper"
    summary: "Multi-step wizard"
    files:
      - templates/{{package_name}}/stepper.html
      - templates/{{package_name}}/stepper.partial.html
      - static/{{package_name}}/stepper.css
      - docs/components/stepper.md
    endpoints:
      - method: GET
        path: "/stepper/{stepKey}"
      - method: POST
        path: "/stepper/{stepKey}"
    events:
      emits: ["{{package_name}}:stepper:change"]
      listens: []
    accessibility:
      notes: ["announce step changes", "disable next when invalid"]

  - key: infinite-list
    title: "Infinite List"
    summary: "Lazy loading list triggered on reveal"
    files:
      - templates/{{package_name}}/infinite-list.html
      - templates/{{package_name}}/infinite-list.partial.html
      - static/{{package_name}}/infinite-list.css
      - docs/components/infinite-list.md
    endpoints:
      - method: GET
        path: "/list"
        description: "Accepts cursor query parameter"
    events:
      emits: ["{{package_name}}:list:append"]
      listens: []
    accessibility:
      notes: ["maintain scroll position", "announce new content"]

  - key: form-validated
    title: "Form with Validation"
    summary: "Server-validated form that replaces invalid groups"
    files:
      - templates/{{package_name}}/form.html
      - templates/{{package_name}}/form.partial.html
      - static/{{package_name}}/form.css
      - docs/components/form.md
    endpoints:
      - method: POST
        path: "/form/validate"
      - method: POST
        path: "/form/submit"
    events:
      emits: ["{{package_name}}:form:invalid", "{{package_name}}:form:submitted"]
      listens: []
    accessibility:
      notes: ["associate errors with fields via aria-describedby"]
```

---

## 17. Documentation Site Plan

* Live examples for every component with a “View HTML” and “Copy” button.
* Side-by-side snippets for Django, FastAPI, and Flask endpoints.
* Patterns section: CRUD with HTMX, streaming updates, pagination and sorting, and validation flows.
* Accessibility checklists per component.

---

## 18. Roadmap

* 0.1: Buttons, Inputs, Modal, Toasts, Table, Dropdown Menu.
* 0.2: Tabs, Drawer, Stepper, Infinite List.
* 0.3: Command Palette, Combobox improvements, Tree View.
* 0.4: File Upload pattern, Data Grid enhancements, Date Picker.

---

## 19. Open Questions

* Should the library ship optional Alpine.js shims for teams who prefer it over Hyperscript, or should that remain an external recipe.
* How strict should the adapters be about differentiating HTMX requests from normal submits when deciding which template to render.
* What level of configurability is needed for keyboard bindings in the command palette and menu components.

---

## 20. Appendix: Minimal FastAPI Example

```python
from fastapi import FastAPI, Request, Form
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates("templates")

@app.get("/", response_class=HTMLResponse)
def home(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})

@app.get("/modal/example", response_class=HTMLResponse)
def modal_example(request: Request):
    return templates.TemplateResponse("{{package_name}}/modal.partial.html", {"request": request})

@app.get("/modal/close", response_class=HTMLResponse)
def modal_close():
    return HTMLResponse("")

@app.post("/modal/submit", response_class=HTMLResponse)
def modal_submit(request: Request, email: str = Form(...)):
    html = f"""
    <div id=\"{{package_name}}-toasts\" hx-swap-oob=\"true\">
      <div class=\"{{package_name}}-toast {{package_name}}-toast--success\">Saved {email}</div>
    </div>
    """
    return HTMLResponse(html)
```

---

## 21. Appendix: Token Starter

```css
:root {
  --{{package_name}}-color-background: #0b0b0c;
  --{{package_name}}-color-foreground: #e8e8ea;
  --{{package_name}}-color-muted: #a0a0a7;
  --{{package_name}}-color-accent: #6aa1ff;
  --{{package_name}}-radius-medium: 12px;
  --{{package_name}}-shadow-1: 0 1px 2px rgba(0,0,0,.2);
  --{{package_name}}-shadow-2: 0 8px 30px rgba(0,0,0,.35);
  --{{package_name}}-spacing-1: .25rem;
  --{{package_name}}-spacing-2: .5rem;
  --{{package_name}}-spacing-3: .75rem;
  --{{package_name}}-spacing-4: 1rem;
  --{{package_name}}-focus-ring: 0 0 0 3px rgba(106,161,255,.5);
}

.{{package_name}}-button { padding: var(--{{package_name}}-spacing-2) var(--{{package_name}}-spacing-4); border-radius: var(--{{package_name}}-radius-medium); box-shadow: var(--{{package_name}}-shadow-1); }
.{{package_name}}-button--accent { background: var(--{{package_name}}-color-accent); color: #0b0b0c; }
.{{package_name}}-input { background: #151518; color: var(--{{package_name}}-color-foreground); border: 1px solid #2a2a32; padding: var(--{{package_name}}-spacing-2) var(--{{package_name}}-spacing-3); border-radius: 10px; }
.{{package_name}}-modal { position: fixed; inset: 0; display: grid; place-items: center; }
.{{package_name}}-modal__backdrop { position: absolute; inset: 0; background: rgba(0,0,0,.5); }
.{{package_name}}-modal__panel { position: relative; background: #101014; color: var(--{{package_name}}-color-foreground); border-radius: var(--{{package_name}}-radius-medium); padding: var(--{{package_name}}-spacing-4); box-shadow: var(--{{package_name}}-shadow-2); }
.{{package_name}}-icon-button:focus-visible { outline: none; box-shadow: var(--{{package_name}}-focus-ring); }
.visually-hidden { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }
```
