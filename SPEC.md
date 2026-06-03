# cycle-request — specification & user guide

A reference for what **cycle-request** does and how its pieces fit together. For
download and install steps, see the [README](README.md). For the engineering-level
design (architecture, ADRs, build), see the
[source repository](https://github.com/MrParkerZ7/project-cycle-request).

> Applies to **v0.0.5**. Feature notes tagged *(new in vX.Y.Z)* indicate when a
> capability first shipped.

---

## 1. Overview

cycle-request is a desktop HTTP/API client in the Postman / Insomnia family, built
around one principle: **the collection is a folder of plain JSON files you own**, not
a record in someone's cloud. That makes API definitions first-class citizens of your
git repository — reviewable in PRs, diffable per request, branchable per feature.

It reads and writes **Postman's `.postman_collection.json` format natively**, so it
interoperates with existing Postman assets without a one-way import.

## 2. Core concepts

| Concept | What it is |
|---------|-----------|
| **Workspace** | A folder you open. May contain a single collection, or several collections side by side (a multi-collection workspace). |
| **Collection** | A `.postman_collection.json` file — the top-level container for folders + requests, plus collection-level auth, scripts, and variables. |
| **Folder** | A grouping inside a collection. Carries its own auth + pre/test scripts that descendant requests can inherit. |
| **Request** | A single HTTP call — method, URL, params, headers, body, auth, and optional pre-request/test scripts and docs. |
| **Environment** | A named set of variables (e.g. `dev`, `staging`, `prod`) selected at send time. |

## 3. On-disk format

- Collections are stored as **Postman v2.1 JSON** (`.postman_collection.json`),
  serialized deterministically so git diffs stay small and meaningful.
- cycle-request-specific metadata that Postman has no field for round-trips
  losslessly inside an `_x_cycle_request` extension object — open the file in
  Postman and back without data loss.
- Editing a collection's or folder's settings in the app writes straight back into
  the same JSON file in place.

## 4. Requests

- **Methods** — GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, and **arbitrary custom
  verbs** (type your own, e.g. PURGE / MKCOL / PROPFIND).
- **URL & query params** — edit the URL directly or via a structured parts editor
  (protocol / host / port / path / query / fragment); `{{var}}` placeholders survive
  intact.
- **Headers** — per-row enable/disable + description metadata.
- **Body types** — none · raw text (language-aware) · JSON · `x-www-form-urlencoded`
  · `multipart/form-data` (including **file** fields) · GraphQL (query + variables).
- **Docs** — a per-request description/Docs tab.
- **Transport behaviors** — Postman's `protocolProfileBehavior` is preserved, with
  selected behaviors applied at send time (e.g. response-size limiting; strict-SSL,
  cookie, and redirect controls via a per-request client rebuild).

## 5. Variables & environments

**Scopes**, from broadest to narrowest — narrower wins:

```
collection variables  →  environment variables  →  request-local variables
```

- **`{{var}}` substitution** applies to URL, headers, body, and inside scripts —
  identical syntax to Postman.
- **Secrets vs shared values** — commit non-secret env values in env files; keep
  secrets in a gitignored per-user local file (and/or the OS keychain). Secrets never
  land in the committed collection.
- **"Current value" persistence** — when a script sets a variable at runtime
  (`pm.environment.set(...)`), that captured value persists across runs in a
  gitignored local store, layered above the committed file — mirroring Postman's
  initial-value / current-value split.
- **Inline preview** — `{{var}}` chips in the URL bar and body editor show the
  resolved value + scope on hover, are editable in place, and Ctrl/Cmd-click jumps to
  the defining environment row.

## 6. Authorization

Supported schemes: **None · Basic · Bearer · API-key** (header or query) **· OAuth 2**.

- **Inherit from parent** — set auth once on a collection or folder; child requests
  default to **Inherit** and resolve the nearest non-inherit ancestor's auth at send
  time (request → folder chain → collection). An explicit *No Auth* stops the chain.
- Auth is injected after variable substitution and never overwrites an
  `Authorization` header you set by hand.

## 7. Scripting

Pre-request and test scripts run in a **sandboxed JavaScript runtime** with a
Postman-compatible `pm.*` API:

- `pm.environment.*` / `pm.variables.*` — get/set variables (current-value aware).
- `pm.test(name, fn)` — register a named assertion.
- `pm.expect(...)` — assertions with the full chai chain
  (`.to.be.a('string').and.to.have.length.greaterThan(...)`, etc.).
- `pm.response.*` — `json()`, `text()`, `code`, `responseTime`, and related accessors.
- `console.log / info / warn / error` — captured and shown in the **Console** tab,
  grouped per request and color-coded by level.

Scripts compose across levels: **collection → folder(s) → request** for pre-request
(outer-to-inner) and the reverse for tests (inner-to-outer).

## 8. Sending & responses

- **Response viewer** — status, time, size, headers, and a body pane with
  dependency-free syntax highlighting and pretty-printing for JSON / XML / HTML.
- **Cookie jar** — a shared cookie store across requests, with a one-click Clear.
- **History** — every send is logged to a re-sendable history list; re-sending fires
  the exact request snapshot as originally sent.

## 9. Automate (collection runner)

- **Saved run configs** — per-collection, remembered between sessions.
- **Persisted run history** — stored alongside the collection (in a `*.cycle-runs/`
  sibling) so past runs are reviewable.
- **Live progress** — pass/fail counts and per-request status as the run proceeds,
  with an **All / Passed / Failed** filter.
- **PDF report** — export a run as a paginated PDF with a KPI grid, analytics tables
  (by status / method / slowest-5 / failure breakdown), and a per-request detail list.

## 10. Import & export

| Direction | Formats |
|-----------|---------|
| **Import** | Postman v2.1 · Insomnia v4 · HAR · cURL |
| **Export** | Postman v2.1 (whole collection) · cURL (per request, copy-as-cURL) |

When you open a folder that contains a foreign export (e.g. a Postman or HAR file)
but no cycle-request collection, the app offers to import it in place.

## 11. User interface

- **Welcome / browse screen** — open a workspace by path or folder picker, with an
  IDE-style **Recent** list (click to reopen · pin · remove · clear). *(new in v0.0.5)*
- **Project Files sidebar** — a VS Code-style file tree of the workspace, with a
  right-click context menu for file actions; resizable, stacked collapsible sections.
- **Tabbed editors** — request and collection/folder settings open as reorderable,
  persisted tab strips (drag to reorder; order is remembered).
- **Collection/folder settings** — edit Overview (description), Authorization,
  Scripts, and (for collections) Variables in a main-area tabbed editor.
- **Themes** — 17 dark + light themes; per-window choice persists.
- **Density** — Standard / Compact / Ultra-compact spacing; persists.
- **Settings dialog** — General, Themes, Shortcuts, Data, Add-ons, Certificates,
  Proxy, Update, About panels.

## 12. Data & file locations

| Data | Where it lives |
|------|----------------|
| Collections | Wherever you save them — your repo/workspace (`*.postman_collection.json`). |
| App settings (theme, density, recent folders, last workspace) | `settings.json` in the OS app-data directory (Electron `userData`). |
| Shared env values | Committed env file(s) in the workspace. |
| Secrets / local overrides | Gitignored per-user local env file (and/or OS keychain). |
| Script-captured "current values" | Gitignored local current-value store next to the env file. |
| Run history | A `*.cycle-runs/` folder beside the collection. |

Settings such as the **Recent folders** list, active theme, and density are stored in
`settings.json` and survive restarts.

## 13. Platform support

| Platform | Architecture | Artifacts |
|----------|--------------|-----------|
| Windows  | x64 | NSIS installer `.exe` + portable `.zip` |
| macOS    | Apple Silicon (arm64) | `.dmg` + portable `.zip` |
| Linux    | x64 (glibc) | `.AppImage` + portable `.zip` |

**System requirements**

- **Windows** 10 / 11 (x64).
- **macOS** on Apple Silicon (recent macOS). *No Intel (x64) build yet.*
- **Linux** x64 with glibc; the `.AppImage` needs FUSE — if unavailable, use the
  portable `.zip`.
- No separate runtime, .NET, or system webview required — Chromium is bundled.

## 14. Known limitations & roadmap

- **Code signing / notarization** — builds are currently unsigned; expect an
  "unidentified developer" prompt (see the README for the per-OS bypass). Signing is
  planned.
- **Auto-update** — `latest*.yml` metadata is published, but the in-app updater isn't
  wired yet; update by downloading a newer release.
- **macOS Intel** and additional Linux packaging (`.deb` / `.rpm`) are not built yet.
- **Client certificates (mTLS)** are deferred; HTTP(S) proxy support is available.

## 15. Links

- **Source & engineering docs** — <https://github.com/MrParkerZ7/project-cycle-request>
- **Changelog** — <https://github.com/MrParkerZ7/project-cycle-request/blob/main/CHANGELOG.md>
- **Downloads / releases** — [latest release](../../releases/latest)
- **License** — MIT ([LICENSE](LICENSE))
