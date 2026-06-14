# Svelte Router SPA

![version](https://img.shields.io/npm/v/svelte-router-spa.svg)
![license](https://img.shields.io/github/license/jorgegorka/svelte-router.svg)
![Code climate](https://img.shields.io/codeclimate/maintainability/jorgegorka/svelte-router.svg)

## What is Svelte Router SPA

Svelte Router adds routing to your Svelte apps. It keeps your routes organized in a single place.

Under the hood it works by registering a **global click handler** that intercepts every same-origin `<a>` click (and a `popstate` handler for back/forward), so plain HTML links work out of the box — you do **not** need a special link component for basic navigation.

It's designed for Single Page Applications (SPA). If you need Server Side Rendering then consider using [Svelte Kit](https://kit.svelte.dev/).

## -- Svelte Router SPA is feature complete. No new features will be added, only bugfixes will be solved --.

## Index

- [Features](#features)
- [Install](#install)
- [Quick start](#quick-start)
- [Realistic project example](#realistic-project-example)
  - [Route definitions](#route-definitions)
  - [App entry](#app-entry)
  - [Layout components](#layout-components)
  - [Page components](#page-components)
- [How route resolution works (the pipeline)](#how-route-resolution-works-the-pipeline)
- [Navigation: `<a>` vs `<Navigate>` vs `navigateTo`](#navigation-a-vs-navigate-vs-navigateto)
  - [Plain `<a>` links (global click handler)](#plain-a-links-global-click-handler)
  - [`<Navigate>` component](#navigate-component)
  - [`navigateTo()` function](#navigateto-function)
  - [Comparison table](#comparison-table)
- [Named params and query strings](#named-params-and-query-strings)
  - [Named params (`:param`)](#named-params-param)
  - [Query strings (`?key=value`)](#query-strings-keyvalue)
  - [Hash / anchors (`#section`)](#hash--anchors-section)
  - [How params accumulate in nested routes](#how-params-accumulate-in-nested-routes)
- [Route guards](#route-guards)
  - [Defining a guard](#defining-a-guard)
  - [Guard evaluation order](#guard-evaluation-order)
  - [Guard + redirect interaction](#guard--redirect-interaction)
  - [Back-button behavior after a guard redirect](#back-button-behavior-after-a-guard-redirect)
- [Redirects](#redirects)
  - [Static redirect (`redirectTo`)](#static-redirect-redirectto)
  - [Guard-based redirect (`onlyIf`)](#guard-based-redirect-onlyif)
  - [Redirect priority](#redirect-priority)
  - [Chained redirects](#chained-redirects)
- [Not Found (404)](#not-found-404)
  - [Default behavior](#default-404-behavior)
  - [Custom 404 component](#custom-404-component)
  - [When does 404 trigger?](#when-does-404-trigger)
- [Route prefix](#route-prefix)
  - [How prefix changes URL parsing](#how-prefix-changes-url-parsing)
  - [Prefix and link interception](#prefix-and-link-interception)
- [Localisation (i18n)](#localisation-i18n)
  - [Mode A — no `lang` option (match any language)](#mode-a--no-lang-option-match-any-language)
  - [Mode B — `lang` option set (match only that language)](#mode-b--lang-option-set-match-only-that-language)
  - [Mode C — language conversion via `navigateTo` / `<Navigate>`](#mode-c--language-conversion-via-navigateto--navigate)
  - [Partial localisation](#partial-localisation)
  - [`defaultLanguage` option](#defaultlanguage-option)
- [Nested routes and layouts](#nested-routes-and-layouts)
  - [Rendering priority inside `<Route>`](#rendering-priority-inside-route)
  - [Auto-index behavior](#auto-index-behavior)
- [TypeScript reference](#typescript-reference)
  - [`Route` (route definition)](#route-route-definition)
  - [`RouterOptions`](#routeroptions)
  - [`CurrentRoute` (runtime route object)](#currentroute-runtime-route-object)
  - [`NavigateProps`](#navigateprops)
  - [Function signatures](#function-signatures)
  - [`activeRoute` Svelte store](#activeroute-svelte-store)
- [API quick reference](#api-quick-reference)
- [Google Analytics](#google-analytics)
- [Credits](#credits)

---

## Features

- Define your routes in a single file
- Layouts: global, per page, or nested to any depth
- Nested routes with automatic index matching
- Named params (`:id`, `:slug`, etc.) accumulated through nesting
- Query string and hash preservation
- Localisation / i18n with per-segment translations
- Guards (`onlyIf`) for protected routes with redirect on failure
- Static redirects (`redirectTo`) and chained redirects
- Route prefix to scope the router under a sub-path
- Custom or default 404 handling
- Google Analytics pageview tracking (optional)
- Works with plain `<a href>` elements **or** the `<Navigate>` component
- TypeScript type declarations included

---

## Install

```bash
# npm
npm i svelte-router-spa

# yarn
yarn add svelte-router-spa
```

Make sure your dev server serves `index.html` for all paths (SPA mode). For the default Svelte setup, add `-s` to sirv:

```json
"start": "sirv public -s"
```

---

## Quick start

```svelte
<!-- App.svelte -->
<script>
  import { Router } from 'svelte-router-spa';
  import { routes } from './routes.js';
</script>

<Router {routes} />
```

```js
// routes.js
import Home from './views/Home.svelte';
import About from './views/About.svelte';

export const routes = [
  { name: '/', component: Home },
  { name: 'about', component: About },
];
```

That's it — both `/` and `/about` work, and you can use plain `<a href="/about">` to navigate.

---

## Realistic project example

The following example combines **prefix**, **localisation**, **nested routes**, **guards**, **redirects**, **named params**, and a **custom 404** — the combination that usually causes confusion when features stack.

### Route definitions

```js
// routes.js
import PublicLayout from './layouts/PublicLayout.svelte';
import AdminLayout from './layouts/AdminLayout.svelte';
import Home from './views/Home.svelte';
import Login from './views/Login.svelte';
import AdminDashboard from './views/admin/Dashboard.svelte';
import EmployeesIndex from './views/admin/EmployeesIndex.svelte';
import EmployeesLayout from './views/admin/EmployeesLayout.svelte';
import EmployeeShow from './views/admin/EmployeeShow.svelte';
import EmployeeCalendar from './views/admin/EmployeeCalendar.svelte';
import NotFound from './views/NotFound.svelte';

function userIsAdmin() {
  // Check your auth store / session — must return true or false synchronously
  return !!localStorage.getItem('admin_token');
}

export const routes = [
  // ── Public routes ──────────────────────────────────────────
  { name: '/', component: Home, layout: PublicLayout },
  {
    name: 'login',
    component: Login,
    layout: PublicLayout,
    lang: { es: 'iniciar-sesion', de: 'anmelden' },
  },

  // ── Static redirect ────────────────────────────────────────
  { name: 'home', redirectTo: '/' },

  // ── Protected admin area ───────────────────────────────────
  {
    name: 'admin',
    component: AdminLayout,
    lang: { es: 'administrador', de: 'verwaltung' },
    onlyIf: { guard: userIsAdmin, redirect: '/login' },
    nestedRoutes: [
      { name: 'index', component: AdminDashboard },
      {
        name: 'employees',
        component: EmployeesLayout,
        lang: { es: 'empleados', de: 'angestellte' },
        nestedRoutes: [
          { name: 'index', component: EmployeesIndex },
          {
            name: 'show/:id',
            component: EmployeeShow,
            lang: { es: 'mostrar/:id' },
          },
          {
            name: 'calendar/:month',
            component: EmployeeCalendar,
            lang: { es: 'calendario/:month', de: 'kalender/:month' },
          },
        ],
      },
    ],
  },

  // ── Custom 404 (must be at top level) ──────────────────────
  { name: '404', path: '404', component: NotFound },
];
```

### App entry

```svelte
<!-- App.svelte -->
<script>
  import { Router } from 'svelte-router-spa';
  import { routes } from './routes.js';

  const options = {
    prefix: 'company',       // All routes live under /company/*
    lang: 'es',              // Match only Spanish translations
    defaultLanguage: 'en',   // Fallback when a segment has no translation
    gaPageviews: false,
  };
</script>

<Router {routes} {options} />
```

With `prefix: 'company'` the routes above are reachable at:

```
/company/                              → Home
/company/iniciar-sesion                → Login (Spanish)
/company/administrador                 → AdminDashboard (auto-index)
/company/administrador/empleados       → EmployeesIndex (auto-index)
/company/administrador/empleados/mostrar/42   → EmployeeShow  { id: '42' }
/company/administrador/empleados/calendario/enero → EmployeeCalendar { month: 'enero' }
```

### Layout components

Every layout **must** include a `<Route>` component where child content should render. The `currentRoute` prop is passed down automatically.

```svelte
<!-- layouts/AdminLayout.svelte -->
<script>
  import { Route, Navigate, routeIsActive } from 'svelte-router-spa';

  export let currentRoute;
  const params = {};
</script>

<div class="admin-shell">
  <nav>
    <!-- <Navigate> adds class="active" automatically -->
    <Navigate to="admin" styles="nav-link">Dashboard</Navigate>
    <Navigate to="admin/employees" styles="nav-link">Employees</Navigate>

    <!-- Plain <a> also works — the global click handler intercepts it -->
    <a href="/company/login" class="nav-link">Logout</a>
  </nav>

  <main>
    <!-- Renders the matched child component or recurses into the next layout -->
    <Route {currentRoute} {params} />
  </main>

  <footer>
    Language: {currentRoute.language}
  </footer>
</div>
```

### Page components

Every page component receives `currentRoute` as a prop:

```svelte
<!-- views/admin/EmployeeShow.svelte -->
<script>
  export let currentRoute;

  // Named params from the URL
  const employeeId = currentRoute.namedParams.id;

  // Query string values
  const tab = currentRoute.queryParams.tab || 'profile';

  // Hash (e.g. #cv)
  const section = currentRoute.hash;
</script>

<h1>Employee #{employeeId}</h1>
<p>Active tab: {tab}</p>
{#if section}
  <p>Jumped to: {section}</p>
{/if}
```

---

## How route resolution works (the pipeline)

When any navigation event occurs (link click, `navigateTo()`, or browser back/forward), the router runs the following pipeline **in order**:

```
1. URL preprocessing
   ├─ Strip trailing slash
   ├─ Strip prefix (if options.prefix is set)
   └─ Parse into segments via URL API

2. Route matching (recursive, depth-first)
   ├─ For each route at the current level:
   │   ├─ Resolve i18n path (consider current language)
   │   ├─ Compare static path first (exact match wins)
   │   └─ Fall back to named-param match (:param) only if no static match
   ├─ On match → evaluate redirect (redirectTo, then guard)
   ├─ On match with nestedRoutes → recurse into children
   └─ On match with nestedRoutes but no remaining segments → try auto-index

3. Post-matching
   ├─ No match or empty nested child? → 404 (custom or /404.html)
   ├─ Strip query params from .path property
   └─ Prepend prefix back to .path

4. Side effects (setActiveRoute)
   ├─ If redirectTo is set → call navigateTo(redirectTo) (new cycle)
   ├─ Otherwise → pushState to browser history
   └─ Update the activeRoute Svelte store
```

**Key takeaway:** The prefix is stripped _before_ matching and re-added _after_. Guards and redirects are evaluated _during_ matching, not after. 404 is only the final fallback.

---

## Navigation: `<a>` vs `<Navigate>` vs `navigateTo`

### Plain `<a>` links (global click handler)

The router registers a single `window.addEventListener('click', …)` that intercepts clicks on **any** `<a>` element when **all** of the following are true:

1. The click target is an `<a>` element.
2. No modifier key is held (meta / ctrl / shift — so "open in new tab" still works).
3. The link is same-origin (same `host`).
4. If a `prefix` is configured, the link's pathname starts with `/{prefix}`.

When intercepted, `event.preventDefault()` is called and `navigateTo()` is invoked with the link's pathname + search + hash.

Special case: if the `<a>` has `target="_blank"`, the URL is opened in a new window via `window.open()` instead.

```html
<!-- These are ALL intercepted automatically -->
<a href="/admin/employees">Employees</a>
<a href="/admin/employees/show/42?tab=projects#cv">Show employee</a>

<!-- These are NOT intercepted -->
<a href="https://external.com/page">External</a>
<a href="/admin" target="_blank">Open in new tab</a>
<!-- Ctrl/Cmd+click also passes through -->
```

> **Gotcha:** Because the handler is attached to `window`, it fires for every `<a>` on the page — even those inside third-party widgets or dynamically inserted content. If a link's host matches but you do NOT want SPA navigation, add `target="_blank"` or use a different element.

### `<Navigate>` component

`<Navigate>` is a thin wrapper around `<a>` that adds two features on top of the global handler:

1. **Automatic `active` class** — adds `class="active"` when the target route matches the current route (uses `routeIsActive`).
2. **i18n conversion** — if you pass `lang="es"`, the `to` prop is converted to the localised path on mount.

```svelte
<script>
  import { Navigate } from 'svelte-router-spa';
</script>

<!-- Renders <a href="admin/employees" class="nav-link active"> when active -->
<Navigate to="admin/employees" styles="nav-link">
  Employees
</Navigate>

<!-- Renders <a href="administrador/empleados"> (converted to Spanish) -->
<Navigate to="admin/employees" lang="es" styles="nav-link">
  Empleados
</Navigate>
```

Under the hood, `<Navigate>` calls `navigateTo(to)` on click (after `preventDefault` + `stopPropagation`), so the route goes through the full resolution pipeline including guards and redirects.

### `navigateTo()` function

For programmatic navigation (form submissions, redirects in event handlers, etc.):

```js
import { navigateTo } from 'svelte-router-spa';

// Basic navigation
navigateTo('admin/employees');

// With language conversion → navigates to /configuracion
navigateTo('setup', 'es');

// Without updating browser history (rare — used internally for popstate)
navigateTo('/page', null, false);
```

`navigateTo` creates a fresh `SpaRouter` instance, runs the full pipeline (including guard and redirect evaluation), pushes a history entry, and updates the Svelte store.

### Comparison table

| Feature | `<a href>` | `<Navigate>` | `navigateTo()` |
|---|---|---|---|
| Prevents full page reload | Yes (if same-origin + prefix match) | Yes | Yes |
| Runs through guards/redirects | Yes | Yes | Yes |
| Adds `active` class automatically | No | Yes | No |
| i18n path conversion | No | Yes (`lang` prop) | Yes (2nd arg) |
| Works without Svelte component | Yes (plain HTML) | No | Yes |
| Preserves query string in URL | Yes (from href) | No (use plain `<a>` for query strings) | Yes (include in pathName) |
| Preserves hash in URL | Yes (from href) | No (use plain `<a>` for hash) | Yes (include in pathName) |
| Ctrl/Cmd+click opens new tab | Yes (native behavior) | Yes (native behavior) | N/A |

---

## Named params and query strings

### Named params (`:param`)

Define them in the route `name` with a leading colon. Each `:param` consumes exactly one URL segment:

```js
{ name: 'show/:id', component: EmployeeShow }
{ name: 'show/:id/:full-name', component: EmployeeDetail }
{ name: 'calendar/:month/:day', component: CalendarDay }
```

They are available in the component as `currentRoute.namedParams`:

```svelte
<script>
  export let currentRoute;
</script>

<p>ID: {currentRoute.namedParams.id}</p>
<p>Name: {currentRoute.namedParams['full-name']}</p>
```

### Query strings (`?key=value`)

Query strings are parsed by the native `URL` API and available as `currentRoute.queryParams`:

```
URL: /admin/employees/show/42?tab=projects&sort=name
```

```js
currentRoute.queryParams // { tab: 'projects', sort: 'name' }
```

Query strings are preserved in the browser URL but stripped from `currentRoute.path` (which only contains the pathname).

### Hash / anchors (`#section`)

The hash fragment is available as `currentRoute.hash`:

```
URL: /admin/employees/show/42#cv
```

```js
currentRoute.hash // '#cv'
```

### How params accumulate in nested routes

Named params are **accumulated** as the router recurses into nested routes. A deeply nested component receives params from all ancestor routes plus its own:

```js
// Route config
{
  name: 'admin',
  nestedRoutes: [{
    name: 'employees',
    nestedRoutes: [
      { name: 'show/:id', nestedRoutes: [
        { name: 'calendar/:month', component: CalendarPage }
      ]}
    ]
  }]
}
```

```
URL: /admin/employees/show/42/calendar/june
```

Inside `CalendarPage`:

```js
currentRoute.namedParams // { id: '42', month: 'june' }
```

Both `id` (from the parent route) and `month` (from the current route) are available.

---

## Route guards

### Defining a guard

A guard is a **synchronous function** that returns `true` (allow access) or `false` (redirect):

```js
function userIsAdmin() {
  return !!localStorage.getItem('admin_token');
}

{
  name: 'admin',
  component: AdminLayout,
  onlyIf: { guard: userIsAdmin, redirect: '/login' }
}
```

- `guard` — the function to call. Must return `true` or `false`.
- `redirect` — the path to navigate to when `guard()` returns `false`. Defaults to `'/'` if omitted.

### Guard evaluation order

Guards are evaluated **during route matching**, at the same level where the route is matched. The order is:

1. The URL segment matches the route name (static or named-param).
2. `RouterRedirect` is invoked for the matched route:
   a. First, `route.redirectTo` is checked (static redirect).
   b. Then, the guard is checked. **If the guard is valid and returns `false`, it overrides any static redirect.**
3. If the guard passes, matching continues into `nestedRoutes`.

**Important:** If a **parent** route's guard fails, the entire nested subtree is skipped — the router redirects immediately without evaluating child routes.

### Guard + redirect interaction

When a route has **both** `redirectTo` and `onlyIf`:

```js
{
  name: 'legacy-admin',
  redirectTo: '/admin',
  onlyIf: { guard: userIsAdmin, redirect: '/login' }
}
```

The resolution order inside `RouterRedirect` is:

1. Start with no redirect.
2. If `redirectTo` is set → use it (`/admin`).
3. If guard is valid AND `guard()` returns `false` → **override** with `guard.redirectPath()` (`/login`).

So: if the guard **passes** → the static redirect fires (`/admin`). If the guard **fails** → the guard redirect fires (`/login`).

### Back-button behavior after a guard redirect

When a guard redirects, the router calls `navigateTo(redirectPath)`, which:

1. Creates a new `SpaRouter` instance for the redirect target URL.
2. Runs the full resolution pipeline again (the redirect target may itself have guards).
3. Calls `history.pushState()` for the redirect target.

This means the **guarded URL is pushed to history first** (by the original navigation), and then the **redirect URL is pushed on top**. Pressing the browser's back button after a guard redirect will:

1. Go back to the guarded URL.
2. The guard will fire again and redirect forward.

This creates a "back-button trap" if the guard keeps failing. The standard solution is to use `replaceState` in your guard's redirect target or to design the redirect target (e.g., `/login`) so that it replaces history. The router itself always uses `pushState`.

> **Tip:** To avoid the trap, handle the post-login redirect programmatically: after the user logs in on the `/login` page, call `navigateTo('admin')` instead of relying on the browser back button.

---

## Redirects

### Static redirect (`redirectTo`)

Always redirects, unconditionally:

```js
{ name: 'old-page', redirectTo: '/new-page' }
{ name: 'home', redirectTo: '/' }
{ name: 'external', redirectTo: 'https://example.com' }
```

The route does not need a `component` or `layout` — it will never be rendered.

### Guard-based redirect (`onlyIf`)

Redirects only when the guard function returns `false`:

```js
{
  name: 'admin',
  component: AdminLayout,
  onlyIf: { guard: userIsAdmin, redirect: '/login' }
}
```

### Redirect priority

When both are present on the same route:

```
guard redirect (highest priority — overrides static redirect when guard fails)
  ↓
static redirectTo (used when guard passes or no guard exists)
  ↓
no redirect → render the route normally
```

### Chained redirects

If a redirect target itself has a redirect or a failing guard, the router handles it correctly. Each `navigateTo()` call creates a fresh `SpaRouter` instance that resolves the new URL independently:

```js
// Route A redirects to Route B
// Route B has a guard that redirects to Route C
// The router follows: A → B → C
```

There is no built-in loop detection. Make sure your redirect chains are acyclic.

---

## Not Found (404)

### Default 404 behavior

When no route matches and no custom 404 route is defined, the router returns:

```js
{ name: '404', component: '', path: '404', redirectTo: '/404.html' }
```

The `redirectTo: '/404.html'` causes `setActiveRoute` to set the browser's active route to `/404.html` (without calling `navigateTo`, to avoid an infinite redirect loop). Your hosting provider is expected to serve a `404.html` file for this path.

### Custom 404 component

Define a route with `name: '404'` at the **top level** of your routes array:

```js
{ name: '404', path: '404', component: MyCustomNotFoundComponent }
```

When no route matches, the router finds this route by name and renders its component like any other route. The `currentRoute` prop will contain `{ name: '404', path: '404', language: '...' }`.

### When does 404 trigger?

A route is considered "not found" when:

1. `searchActiveRoutes` returns an empty object (no route matched at any level), **or**
2. `anyEmptyNestedRoutes` returns `true` (recursively checks if any `childRoute` in the chain is empty — this happens when a parent matches but no child matches the remaining path segments).

**Example:** With the route config from the realistic example above:

```
/company/nonexistent          → 404 (no top-level match)
/company/admin/employees/junk → 404 (parent matches but no child matches "junk")
/company/admin                → AdminDashboard (auto-index, NOT 404)
```

---

## Route prefix

```svelte
<Router {routes} options={{ prefix: 'blog' }} />
```

With a prefix, all your routes live under `/{prefix}/*` without needing to include the prefix in your route definitions.

### How prefix changes URL parsing

1. **Before matching:** The prefix is stripped from the URL. So `/blog/about-us` becomes `/about-us` internally.
2. **During matching:** Routes are matched as if the prefix doesn't exist.
3. **After matching:** The prefix is prepended back to `currentRoute.path`.

Your route definitions do **not** include the prefix:

```js
// With prefix: 'blog', these routes match /blog/ and /blog/about
const routes = [
  { name: '/', component: BlogIndex },
  { name: 'about', component: BlogAbout },
];
```

### Prefix and link interception

The global click handler only intercepts `<a>` links whose pathname starts with `/{prefix}`. Links to paths outside the prefix scope are left alone, allowing normal (non-SPA) navigation:

```html
<!-- With prefix: 'blog' -->
<a href="/blog/about">Intercepted → SPA navigation</a>
<a href="/dashboard">NOT intercepted → normal navigation to /dashboard</a>
```

This is useful when the SPA is embedded in a larger site — only links within the SPA's scope are handled by the router.

---

## Localisation (i18n)

Routes can define translated path segments via the `lang` property:

```js
{
  name: 'employees',
  component: EmployeesIndex,
  lang: { es: 'empleados', de: 'angestellte' }
}
```

How the router uses translations depends on the `lang` option passed to `<Router>`:

### Mode A — no `lang` option (match any language)

When no language is set in options, the router accepts **all** language variants plus the default name:

```
/admin/employees              → matches (default name)
/administrador/empleados      → matches (all Spanish)
/admin/empleados              → matches (mixed: default + Spanish)
```

`currentRoute.language` reflects whichever language the **last matched segment** used. If the default name matched, `language` is `undefined` (or `defaultLanguage` if that option is set).

### Mode B — `lang` option set (match only that language)

When `options.lang = 'es'`, only the Spanish translations are accepted. Default names become **invalid** for segments that have a Spanish translation:

```
/login                        → 404 (has lang.es = 'iniciar-sesion')
/iniciar-sesion               → matches
/admin/employees              → 404 (both have Spanish translations)
/administrador/empleados      → matches
```

For segments that have **no** translation in the active language, the default name is still used as a fallback.

### Mode C — language conversion via `navigateTo` / `<Navigate>`

When you pass a language to `navigateTo` or the `<Navigate>` component, the router matches the default-name route but **converts** the output path to the localised version:

```js
navigateTo('admin/employees', 'es');
// → navigates to /administrador/empleados
```

```svelte
<Navigate to="admin/employees" lang="es">Empleados</Navigate>
<!-- Renders <a href="administrador/empleados"> -->
```

### Partial localisation

If only some segments have translations, the untranslated ones use the default name:

```js
// calendar has de: 'kalender', but admin has no German translation
// URL: /admin/employees/show/123/kalender/april → valid
// currentRoute.language === 'de' (from the last translated segment)
```

### `defaultLanguage` option

When set, `currentRoute.language` returns this value instead of `undefined` when no specific language matched:

```svelte
<Router {routes} options={{ defaultLanguage: 'en' }} />
```

```js
// Navigating to /admin/employees (default names)
currentRoute.language // 'en' (instead of undefined)
```

---

## Nested routes and layouts

Nested routes are matched recursively. When the URL has more segments than the current level, the router recurses into `nestedRoutes`:

```
URL: /admin/employees/show/42

Level 1: 'admin' matches → has nestedRoutes, segments remain → recurse
Level 2: 'employees' matches → has nestedRoutes, segments remain → recurse
Level 3: 'show/:id' matches → id = '42', no more segments → done
```

### Rendering priority inside `<Route>`

The `<Route>` component renders in this priority order:

```
1. layout  → render layout component, pass currentRoute (with layout stripped)
2. component → render component, pass currentRoute (with component stripped)
3. childRoute → recurse with <svelte:self> into the child route
```

When a **layout** is rendered, it receives the `currentRoute` object which still contains `childRoute`. The layout is expected to embed a `<Route {currentRoute} />` to continue the rendering chain.

When a **component** is rendered, it receives `currentRoute` with the `component` property stripped (to prevent infinite recursion if the component accidentally includes `<Route>`).

### Auto-index behavior

If a route has `nestedRoutes` but the URL has no more segments, the router automatically tries to match a child route with name `'index'`:

```js
{
  name: 'admin',
  component: AdminLayout,
  nestedRoutes: [
    { name: 'index', component: AdminDashboard },
    { name: 'settings', component: AdminSettings },
  ]
}
```

```
/admin → matches 'admin', no more segments → auto-tries ['index'] → renders AdminDashboard
```

---

## TypeScript reference

Type declarations are included in the `types/` directory and referenced via the `types` field in `package.json`.

### `Route` (route definition)

```ts
import type { Route } from 'svelte-router-spa/types/components/router';

type Route = {
  name: string;                                    // URL path segment (required)
  component?: typeof SvelteComponent;              // Page component to render
  layout?: typeof SvelteComponent;                 // Layout wrapper component
  nestedRoutes?: Route[];                          // Child routes
  redirectTo?: string;                             // Static redirect target
  onlyIf?: {                                       // Guard configuration
    guard: (...args: any) => boolean | Promise<boolean>;
    redirect: string;                              // Path to redirect to on guard failure
  };
  lang?: Record<string, string> | string;          // i18n translations { es: '...', de: '...' }
};
```

### `RouterOptions`

```ts
import type { RouterOptions } from 'svelte-router-spa/types/components/router';

type RouterOptions = Partial<{
  prefix: string;           // Scope all routes under this path
  gaPageviews: boolean;     // Track route changes as GA pageviews
  lang: Language;           // Match only this language's translations
  defaultLanguage: string;  // Fallback value for currentRoute.language
}>;
```

### `CurrentRoute` (runtime route object)

This is the object passed as `currentRoute` prop to every layout and component:

```ts
import type { CurrentRoute } from 'svelte-router-spa/types/components/route';

type CurrentRoute = {
  name: string;                              // Resolved path name
  path: string;                              // Full path (without query params)
  hash: string;                              // URL hash fragment (e.g. '#cv')
  component?: typeof SvelteComponent;        // Matched component (stripped after render)
  layout?: typeof SvelteComponent;           // Matched layout (stripped after render)
  queryParams: Record<string, string>;       // Parsed query string
  namedParams: Record<string, string>;       // Accumulated named params from all ancestors
  childRoute: CurrentRoute;                  // Next level's route info (recursive)
  language?: string;                         // Language of the last matched segment
};
```

### `NavigateProps`

```ts
import type { NavigateProps } from 'svelte-router-spa/types/components/navigate';

interface NavigateProps {
  to: string;       // Target route path (required)
  title?: string;   // HTML title attribute for the <a> element
  styles?: string;  // CSS class names to apply to the <a> element
  lang?: string;    // Language to convert the path to (i18n)
}
```

### Function signatures

```ts
// Navigate programmatically
function navigateTo(
  pathName: string,
  language?: string,
  updateBrowserHistory?: boolean  // default: true
): void;

// Check if a path is the current active route
function routeIsActive(
  queryPath: string,
  includePath?: boolean  // default: false (exact match); true = substring match
): boolean;

// Get the localised version of a route without navigating
function localisedRoute(
  pathName: string,
  language: string
): { redirectTo: string };  // Returns the route object with the converted path

// Low-level router engine (used internally; rarely needed directly)
function SpaRouter(
  routes: Route[],
  currentUrl: string | undefined,
  options?: {}
): Readonly<{
  setActiveRoute: (updateBrowserHistory?: boolean) => any;
  findActiveRoute: () => { redirectTo: string };
}>;
```

### `activeRoute` Svelte store

The router exposes a Svelte writable store that updates on every navigation:

```ts
import { activeRoute } from 'svelte-router-spa/src/store';

// In a component:
$: console.log($activeRoute);  // Reactively logs the current route object

// Methods:
activeRoute.subscribe(callback)  // Svelte store subscribe
activeRoute.set(route)           // Set the active route (used internally)
activeRoute.remove()             // Reset to empty object
```

---

## API quick reference

| Export | Type | Import | Purpose |
|---|---|---|---|
| `Router` | Component | `import { Router } from 'svelte-router-spa'` | Top-level router; takes `routes` and `options` props |
| `Route` | Component | `import { Route } from 'svelte-router-spa'` | Renders matched component/layout inside layouts |
| `Navigate` | Component | `import { Navigate } from 'svelte-router-spa'` | Link with auto `active` class and i18n conversion |
| `navigateTo` | Function | `import { navigateTo } from 'svelte-router-spa'` | Programmatic navigation |
| `routeIsActive` | Function | `import { routeIsActive } from 'svelte-router-spa'` | Check if a path is the current route |
| `localisedRoute` | Function | `import { localisedRoute } from 'svelte-router-spa'` | Get i18n version of a route path |
| `SpaRouter` | Function | `import { SpaRouter } from 'svelte-router-spa'` | Low-level router engine (rarely needed) |

---

## Google Analytics

Track route changes as pageviews:

```svelte
<Router {routes} options={{ gaPageviews: true }} />
```

This calls `ga('set', 'page', path)` and `ga('send', 'pageview')` on every navigation (including redirects).

---

## Credits

Svelte Router has been developed by [Jorge Alvarez](https://www.alvareznavarro.es)

### Contributors

[Mark Kopenga](https://github.com/mjarkk)

[Fidel Ramos](https://github.com/haplo)

[Steve Phillips](https://github.com/elimisteve)

[David McCrea](https://github.com/davemccrea)

[Pascal Clanget](https://github.com/Gh05d)

[A J](https://github.com/aj-nk)

[David Kiss](https://github.com/xdavidkissx)

[Common Creator](https://github.com/CommonCreator)

[SianLoong](https://github.com/si3nloong)

[Frippertronics](https://github.com/frippertronics)

[CHamalainen](https://github.com/CHamalainen)
