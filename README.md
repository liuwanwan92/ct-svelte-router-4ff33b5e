# Svelte Router SPA

![version](https://img.shields.io/npm/v/svelte-router-spa.svg)
![license](https://img.shields.io/github/license/jorgegorka/svelte-router.svg)
![Code climate](https://img.shields.io/codeclimate/maintainability/jorgegorka/svelte-router.svg)

## What is Svelte Router SPA

Svelte Router adds routing to your Svelte apps. It keeps your routes organized in a single place.

With Svelte Router SPA you have all the features you need to create modern web applications with minimal configuration.

It's designed for Single Page Applications (SPA). If you need Server Side Rendering then consider using [Svelte Kit](https://kit.svelte.dev/).

## -- Svelte Router SPA is feature complete. No new features will be added, only bugfixes will be solved --.

## Index

* [Features](#features)
* [Install](#install)
* [Usage](#usage)
  * [Layouts and route info](#layouts-and-route-info)
  * [Anatomy of a route](#anatomy-of-a-route)
  * [Using named params as first part of path name (not recommended)](#using-named-params-as-first-part-of-path-name-not-recommended)
* [Route prefix](#route-prefix)
* [Localisation](#localisation)
  * [Rendering a page in different languages](#rendering-a-page-in-different-languages)
* [Google Analytics](#google-analytics)
* [Not Found - 404](#not-found---404)
* [API](#api)
  * [Router](#router)
  * [Route](#route)
  * [currentRoute](#currentroute)
  * [Navigate](#navigate)
  * [navigateTo](#navigateto)
  * [routeIsActive](#routeisactive)
  * [localisedRoute](#localisedroute)
* [Example of use](#example-of-use)
* [实战指南 叠加特性的生效顺序与跳转（中文）](#实战指南-叠加特性的生效顺序与跳转)
  * [接近真实项目的配置示例](#接近真实项目的配置示例)
  * [解析与生效顺序](#解析与生效顺序)
  * [普通链接与组件跳转的差别](#普通链接与组件跳转的差别)
  * [参数与查询串如何到达 currentRoute](#参数与查询串如何到达-currentroute)
  * [守卫拦截与返回上一页](#守卫拦截与返回上一页)
  * [未匹配与重定向的优先级](#未匹配与重定向的优先级)
  * [类型说明](#类型说明)
  * [注意事项与已知陷阱](#注意事项与已知陷阱)
* [Credits](#credits)

## Features

- Define your routes in a single interface
- Layouts global, per page or nested.
- Nested routes.
- Named params.
- Localisation.
- Guards to protect urls. Public and private urls.
- Route prefix.
- Track pageviews in Google Analytics (optional).
- Use standard `<a href="/about-us">About</a>` elements to navigate between pages (or use [`<Navigate />`](#navigate) for bonus features).

Svelte Router is smart enought to inject the corresponding params to each Route component. Every Route component has information about their named params, query params and child route.

You can use all that information (availabe in the currentRoute property) to help you implement your business logic and secure the app.

## Install

To install Svelte Router on your svelte app:

with npm

```bash
npm i svelte-router-spa
```

with Yarn

```bash
yarn add svelte-router-spa
```

## Usage

Ensure your local server is configured in SPA mode. In a default Svelte installation you need to edit your package.json and add _-s_ to `sirv public`.

```javascript
"start": "sirv public -s"
```

Instead of having your routes spread inside your code Svelte Router SPA lets you define them inside a file where you can easily identify all available routes.

Add a routes.js file with your routes info. Example:

```javascript
import Login from './views/public/login.svelte'
import PublicIndex from './views/public/index.svelte'
import PublicLayout from './views/public/layout.svelte'
import AdminLayout from './views/admin/layout.svelte'
import AdminIndex from './views/admin/index.svelte'
import EmployeesIndex from './views/admin/employees/index.svelte'

function userIsAdmin() {
  //check if user is admin and returns true or false
}

const routes = [
  {
    name: '/',
    component: PublicLayout,
  },
  { name: 'login', component: Login, layout: PublicLayout },
  {
    name: 'admin',
    component: AdminLayout,
    onlyIf: { guard: userIsAdmin, redirect: '/login' },
    nestedRoutes: [
      { name: 'index', component: AdminIndex },
      {
        name: 'employees',
        component: '',
        nestedRoutes: [
          { name: 'index', component: EmployeesIndex },
          { name: 'show/:id', component: EmployeesShow },
        ],
      },
    ],
  },
]

export { routes }
```

Import the routes into your main component (probably App.svelte)

```javascript
<script>
  import { Router } from 'svelte-router-spa'
  import { routes } from './routes'
</script>

<Router {routes} />
```

That's all

### Layouts and route info

Every Route file will receive a currentRoute property with information about the current route, params, queries, etc.

You can add any number of layouts nested inside Router. For instance assuming that I want two layouts one for public pages and the other for private admin pages I would create these two files:

Filename: _public_layout.svelte_

```javascript
<script>
  import { Route } from 'svelte-router-spa'
  import TopHeader from './top_header.svelte'
  export let currentRoute
  const params = {}
</script>

<div class="app">
  <TopHeader />
  <section class="section">
    <Route {currentRoute}  {params} />
  </section>
</div>
```

Filename: _admin_layout.svelte_

```javascript
<script>
  import { Route } from "svelte-router-spa";

  export let currentRoute;
  const params = {}
</script>

<div>
  <h1>Admin Layout</h1>
  <Route {currentRoute} {params} />
</div>
```

The route page will take care of rendering the appropriate component inside the layout. It will also pass a property called _currentRoute_ to the component with information about the route, nested and query params.

**Tip:** You can have any number of layouts and you can nest them into each other as much as you want. Just remember to add a _Route_ component where the content should be rendered inside the layout.

**Tip:** The _Route_ component will pass a property to the rendered component named _currentRoute_ with information about the current route, params, queries, etc.

### Anatomy of a route

Each route is an object that may have the following properties:

```javascript

function userIsAdmin() {
  // return true or false
}


{
  name: 'about-us',
  component: About,
  layout: PublicLayout,
  redirectTo: 'company',
  onlyIf: { guard: userIsAdmin, redirect: '/login' },
  lang: { es: 'acerca-de' },
  nestedRoutes: [
    { name: 'our-values', component: CompanyValues, lang: { es: 'nuestros-valores' } }
  ]
}

```

**name (required)**: The name that will be used in the url. This is the default name for the route if no localisation is defined or no language is set.

**component (required if no layout is present)**: A component that will be rendered when this route is active. If the route has nestedRoutes the component should be a Layout.

**layout (required if no component is present)**: A component that acts as a layout (a container for other child components).

_Either a component or a layout should be specified. Both can not be empty._

**nestedRoutes**: An array of routes.

**redirectTo**: A url or pathname (https://yourwebsite.com) or (/my-product).

**onlyIf**: An object to conditionally render a route. If guard returns true then route is rendered. If guard is false it redirects to _redirect_.

**lang**: An object with route names localised. Check [Localisation](#localisation)

Routes can contain as many nested routes as needed.

They can also contain as many layouts as needed. Layouts can be nested into other layouts.

In the following example both the home root ('/' and 'login' will use the same layout). Admin, employees and employeesShow will use the admin layout and employees will also use the employees layout, rendered inside the admin layout.

Example of routes:

```javascript
const routes = [
  {
    name: '/',
    component: PublicIndex,
    layout: PublicLayout,
  },
  { name: 'login', component: Login, layout: PublicLayout },
  {
    name: 'admin',
    component: AdminIndex,
    layout: AdminLayout,
    nestedRoutes: [
      {
        name: 'employees',
        component: EmployeesIndex,
        nestedRoutes: [
          {
            name: 'show/:id',
            component: EmployeesShowLayout,
            nestedRoutes: [
              { name: 'index', component: EmployeesShow },
              { name: 'list', component: EmployeesShowList },
            ],
          },
        ],
      },
    ],
  },
]
```

The routes that this file will parse successfully are:

```
/
/login
/admin
/admin/employees
/admin/employees/show
/admin/employees/show/{id}
/admin/employees/show/{id}/list
```

### Using named params as first part of path name (not recommended)

Svelte Router is usually smart enough to find the right route for you. It means that you don't need to care about the order in which you write your routes. There is an exception to this rule: If you define a named param as the very first part of the path like: /:user-name/edit

In this specific case order matters and you should add that route **after** all other routes.

This is not recommended and you should always start your routes with a static path name. You can add as many named params as you need after the first static name.

```javascript

function userIsAdmin() {
  // return true or false
}


{
  name: 'about-us',
  component: About,
  lang: { es: 'acerca-de' },
  nestedRoutes: [
    {
      name: 'our-values', component: CompanyValues, lang: { es: 'nuestros-valores' }
    }
  ]
},
{
  name: '/',
  component: HomeComponent
},
{
  name: '/project/:name',
  component: ProjectComponent
},
{
  name: '/:user-name/edit',
  component: EditUserComponent
}

```

## API

### Router

`import { Router } from 'svelte-router-spa'`

This is the main component that needs to be included before any other content as it holds information about your routes and which route should be rendered.

The simplest approach (although not required) is to have an App.svelte file like this:

```javascript
<script>
  import { Router } from 'svelte-router-spa'
  import { routes } from './routes'

  let options = { gaPageviews: true}
</script>

<Router {routes} {options} />
```

The layout and/or the component that matches the active route will be rendered inside _Router_.

Options is an object that supports three properties:

_gaPageviews_ that will record route changes as pageviews in Google Analytics. It's disabled by default.

_lang_ a string that sets the language that the router will use to match the active route. Check [Localisation](#localisation)

_defaultLanguage_ If no language is set the active route will return this value as the active language.

### Route

`import { Route } from 'svelte-router-spa'`

This component is only needed if you create a layout. It will take care of rendering the content for the child components or child layouts recursively. You can have as many nested layouts as you need.

The info about the current route will be received as a property so you need to define _currentRoute_ and export it.

It will also accept an object called named _params_ where you can send any aditional params to the rendered component. This is usefull if you add any logic in your template, to check user's permission for instance, and want to send extra info to the rendered component.

currentRoute has all the information about the current route and the child routes.

Route is smart enough to expose the named params in the route component where they will be rendered.

Example:

```javascript
<script>
  import { Route } from 'svelte-router-spa'
  import TopHeader from './top_header.svelte'
  import FooterContent from './footer_content.svelte'
  export let currentRoute

  const params = { validCheck: true }
</script>

<div class="app">
  <TopHeader />
  <section class="section">
    <Route {currentRoute} {params} />
    <p>Route params are: {currentRoute.namedParams} and {currentRoute.queryParams}</p>
  </section>
  <FooterContent />
</div>
```

### currentRoute

This object is propagated from _Route_ to the components it renders. It contains information about the current route and the child routes.

These are the properties available in this object:

- name
- component
- layout
- queryParams
- namedParams
- childRoute
- language

**Example:**

```javascript
const routes = [
  {
    name: '/public',
    component: PublicLayout,
    nestedRoutes: [
      {
        name: 'about-us',
        component: 'AboutUsLayout',
        nestedRoutes: [
          { name: 'company', component: CompanyPage },
          { name: 'people', component: PeoplePage },
        ],
      },
    ],
  },
]
```

That configuration will parse correctly the following routes:

```javascript
/public
/public/about-us
/public/about-us/company
/public/about-us/people/:name
```

If the user visits /public/about-us/people/jack the following components will be rendered:

```
Router -> PublicLayout(Route) -> AboutUsLayout(Route) -> PeoplePage
```

Inside PeoplePage you can get all the information about the current route like this:

```javascript
<script>
  export let currentRoute
</script>

<h1>Your name is: {currentRoute.namedParams.name}</h1>
```

This will render:

```html
<h1>Your name is: Jack</h1>
```

### Navigate

`import { Navigate } from 'svelte-router-spa'`

#### params

- **to** (Required) String A valid route to navigate to.
- **title** (Optional) String A title for the _a_ element.
- **styles** (Optional) String Class styles to be applied to the _a class_ element.
- **lang** (Optional) String A language to convert the route to.

Navigate is a wrapper around the < a href="" > element to help you generate links quick and easily.

It adds an _active_ class to the styles if the generated route is the active one.

Check **navigateTo** belown for more information about the language param.

Example:

```javascript
<script>
  import { Navigate } from 'svelte-router-spa'
</script>

<div class="app">
  <h1>My content</h1>
  <p>Now I want to generate a <Navigate to="admin/employees">link to /admin/employees</Navigate></p>
</div>
```

### navigateTo

`import { navigateTo } from 'svelte-router-spa'`

#### params

- **route name** (Required) String A valid route to navigate to.
- **language** (Optional) String A language to convert the route to.

navigateTo allows you to programatically navigate to a route from inside your app updating the browser url and history.

navigateTo receives a route name as a param and an optional language and will try to navigate to that route.

When a language is provided _navigateTo_ will try to convert the _route name_ to the localised version of the route.

```javascript
  // Example route
  {
    name: '/setup',
    component: 'SetupComponent',
    lang: { es: 'configuracion' }
  }
```

```javascript
navigateTo('setup') // Will redirect to /setup

navigateTo('setup', 'es') // Will redirect to /configuracion
```

### routeIsActive

`import { routeIsActive } from 'svelte-router-spa'`

Returns a boolean indicating if the path is the current active route.

This is useful, for instance to set an _active_ class on a menu.

#### Params

- **pathName** (required): A string with the path that you want to check.
- **includePath** (optional | default is _false_): A boolean indicating if pathName should match exactly the current route or if it should be included.

The [Navigate](https://github.com/jorgegorka/svelte-router/blob/master/README.md#navigate) component does this automatically and adds an _active_ class if the generated route is the active one. Navigate sets _includePath_ to false

Example:

```javascript
<script>
  import { routeIsActive } from 'svelte-router-spa'
</script>

<a href="/contact-us" class:active={routeIsActive('/contact-us')}>
  Say hello
</a>

// If current route is /admin/companies/show/my-company

routeIsActive('admin') // returns false
routeIsActive('show/my-company') // returns false
routeIsActive('admin/companies/show/my-company') // returns true
routeIsActive('admin', true) // returns true
routeIsActive('show/my-company', true) // returns true
routeIsActive('my-company', true) // returns true
routeIsActive('other-company', true) // returns false
```

If _includePath_ is true and the current route is `/admin/companies/show/my-company`

### localisedRoute

`import { localisedRoute } from 'svelte-router-spa'`

#### params

- **route name** (Required) String A valid route to navigate to.
- **language** (Required) String A language to convert the route to.

localisedRoute returns a string with the route localised to the specified language.

```javascript
  // Example route
  {
    name: '/setup',
    component: 'SetupComponent',
    lang: { es: 'configuracion', it: 'configurazione' }
  }
```

```javascript
localisedRoute('setup', 'es') // Will return the string "/configuracion"

localisedRoute('setup', 'it') // Will return the string "/configurazione"
```

### Not Found - 404

#### Default behaviour

Svelte Router redirects to a 404.html page if a route is not found. Most hosting providers support this configuration and will serve a 404.html page automatically for not found pages. Just add a 404.html page in the same directory where your index.html file is.

#### Custom behaviour

If you define a 404 route it will be rendered instead of the default behaviour.

```javascript
  // Example of a custom 404 route
  {
    name: '404',
    path: '404',
    component: MyCustomNotFoundcomponent
  }
```

## Route prefix

You can easily constrain all your routes to a specific path like _/blog_

```javascript
<Router { routes } options={ {prefix: 'blog'} } />
```

With this option you don't have to define all your routes starting with _blog_ they will be prefixed automatically.

Using prefix has two advantages:

- You don't need to create a top level route in your routes file and then add every route as a nested route.
- Routes that don't start with the prefix will not be returned as 404 since they are out of the scope of the prefixed routes so you can navigate to them.

## Google Analytics

If you want to track route changes as pageviews in Google Analytics just add

```javascript
<Router { routes } options={ {gaPageviews: true} } />
```

## Localisation

How localisation works depends on the _lang_ param being passed to the _Router_ component. If a language is specified the router will try to match a route in that language only. If no language is specified then the router will try to find a route in any language available.

```javascript
  const options = { lang: 'de' }

  <Router {routes} {options} />
```

Let's see some examples using the following routes.

```javascript
const routes = [
  {
    name: '/',
    component: PublicIndex,
  },
  { name: 'login', component: Login, lang: { es: 'iniciar-sesion' } },
  { name: 'signup', component: SignUp, lang: { es: 'registrarse' } },
  {
    name: 'admin',
    component: AdminIndex,
    lang: { es: 'administrador' },
    nestedRoutes: [
      {
        name: 'employees',
        component: EmployeesIndex,
        lang: { es: 'empleados' },
        nestedRoutes: [
          {
            name: 'show/:id',
            component: ShowEmployeeLayout,
            lang: { es: 'mostrar/:id' },
            nestedRoutes: [
              {
                name: 'index',
                component: ShowEmployee,
              },
              {
                name: 'calendar/:month',
                component: CalendarEmployee,
                lang: { es: 'calendario/:month', de: 'kalender/:month' },
              },
            ],
          },
        ],
      },
    ],
  },
]
```

If we don't specify a language the following routes are valid:

`/login`

`/setup`

`/admin/employees`

`/admin/employees/show/123`

`/admin/employees/show/123/calendar/june`

`/iniciar-sesion`

`/registrarse`

`/administrador/empleados`

`/administrador/empleados/mostrar/123`

`/administrador/empleados/mostrar/123/calendario/junio`

If we specify a language the router will try to find routes **only** in that language so if in our current example we set the _lang_ variable to _'es'_ these routes will be **invalid** and the router will return a 404 page:

`/login`

`/setup`

`/admin/employees`

`/admin/employees/show/123`

`/admin/employees/show/123/calendar/june`

However these other routes will be **valid**:

`/iniciar-sesion`

`/registrarse`

`/administrador/empleados`

`/administrador/empleados/mostrar/123`

`/administrador/empleados/mostrar/123/calendario/junio`

_currentRoute_ will return the language of the matched route.

Another example: In the routes above there is only one german localised route for _calendar_ so this url will be valid:

`/admin/employees/show/123/kalender/april`

The router will match the default route for all paths that are not localised and will match the german one for the one that specfies a localisation.

That route will set **'de'** as the language in _currentRoute_

### Rendering a page in different languages

If you use _Navigate_ and _navigateTo_ to generate links and navigate to different parts of your application an automatic language conversion will be done for you.

Both _Navigate_ and _navigateTo_ support an aditional parameter with a language. If a language is provided they will try to convert the default route into the corresponding one for that language.

Example:

```javascript
  // Example route
  {
    name: '/setup',
    component: 'SetupComponent',
    lang: { es: 'configuracion' }
  }
```

```javascript
navigateTo('setup') // Will redirect to /setup

navigateTo('setup', 'es') // Will redirect to /configuracion
```

There is also available a function called _localisedRoute_ that will return a string with the translated route, in case you want the translation but not navigating to the route.

The router options accept a property called _defaultLanguage_ This value will be returned by the activeRoute object if there is no language selected.

### Example of use

[Demanda](https://github.com/jorgegorka/demanda) is an open source e-commerce application made with Ruby on Rails for the backend and Svelte for the frontend.  It is a [very good example](https://github.com/jorgegorka/demanda/tree/master/frontend/src/lib/routes) of how to use Svelte Router SPA.

## 实战指南 叠加特性的生效顺序与跳转

> 本节为中文补充，面向把本库接入现有前端的场景。基础跳转之外，最容易出问题的是 **路由前缀、本地化、嵌套路由、守卫、重定向、404 叠在一起时谁先生效**。下面给一套接近真实项目的配置，并把每个环节按源码里的真实行为讲清楚，避免现有用法被误解。

本节内容：

* [接近真实项目的配置示例](#接近真实项目的配置示例)
* [解析与生效顺序](#解析与生效顺序)
* [普通链接与组件跳转的差别](#普通链接与组件跳转的差别)
* [参数与查询串如何到达 currentRoute](#参数与查询串如何到达-currentroute)
* [守卫拦截与返回上一页](#守卫拦截与返回上一页)
* [未匹配与重定向的优先级](#未匹配与重定向的优先级)
* [类型说明](#类型说明)
* [注意事项与已知陷阱](#注意事项与已知陷阱)

### 接近真实项目的配置示例

这套配置同时用到了：路由前缀（`prefix`）、本地化（`lang`）、嵌套路由（`nestedRoutes`）、自动 `index` 子路由、命名参数（`:id`）、守卫（`onlyIf`）、永久重定向（`redirectTo`）以及自定义 404。

`routes.js`

```js
import PublicLayout from './views/public/layout.svelte'
import Home from './views/public/home.svelte'
import Login from './views/public/login.svelte'
import AdminLayout from './views/admin/layout.svelte'
import Dashboard from './views/admin/dashboard.svelte'
import EmployeesIndex from './views/admin/employees/index.svelte'
import EmployeeShow from './views/admin/employees/show.svelte'
import NotFound from './views/not_found.svelte'

// 守卫必须“同步返回布尔值”，不要返回 Promise，原因见“注意事项与已知陷阱”
function userIsLoggedIn() {
  return Boolean(localStorage.getItem('token'))
}

const routes = [
  // 首页：component + layout，渲染顺序为 PublicLayout -> Home
  { name: '/', component: Home, layout: PublicLayout },

  // 普通页 + 本地化名（西语 acceso）
  { name: 'login', component: Login, layout: PublicLayout, lang: { es: 'acceso' } },

  {
    name: 'admin',
    layout: AdminLayout,
    // 守卫挂在父级上，会保护整棵子树；失败时整条链重定向到 /login
    onlyIf: { guard: userIsLoggedIn, redirect: '/login' },
    lang: { es: 'administrador' },
    nestedRoutes: [
      // 访问 /admin（无更多路径段）时会自动渲染名为 index 的子路由
      { name: 'index', component: Dashboard },
      {
        name: 'employees',
        component: EmployeesIndex,
        lang: { es: 'empleados' },
        nestedRoutes: [
          // 命名参数 :id；本地化后为 ver/:id
          { name: 'show/:id', component: EmployeeShow, lang: { es: 'ver/:id' } },
        ],
      },
    ],
  },

  // 纯重定向路由：命中即重定向，component 可留空（重定向会先短路，不会去渲染它）
  { name: 'old-dashboard', component: '', redirectTo: '/admin' },

  // 自定义 404：名字必须是 '404'，会“原地渲染”而不是跳转到 /404.html
  { name: '404', component: NotFound },
]

export { routes }
```

`App.svelte`

```svelte
<script>
  import { Router } from 'svelte-router-spa'
  import { routes } from './routes'

  // 注意这里没有设置 lang：表示“默认名和任意 lang 变体都能匹配”
  const options = { prefix: 'app', defaultLanguage: 'en' }
</script>

<Router {routes} {options} />
```

加上 `prefix: 'app'` 后，下面这些 URL 都能正确解析（前缀会在匹配前被剥离，命中后再拼回到 `currentRoute.path`）：

```
/app
/app/login              /app/acceso
/app/admin              （自动渲染 index -> Dashboard）
/app/admin/employees    /app/administrador/empleados
/app/admin/employees/show/42   /app/administrador/empleados/ver/42
/app/old-dashboard      （重定向到 /admin）
```

布局里要放一个 `<Route>` 作为子内容的渲染位：

`views/admin/layout.svelte`

```svelte
<script>
  import { Route } from 'svelte-router-spa'
  export let currentRoute
  const params = {} // 可借此把额外数据（如权限）透传给被渲染的组件
</script>

<div class="admin">
  <nav><!-- 侧边栏 --></nav>
  <Route {currentRoute} {params} />
</div>
```

页面组件通过 `currentRoute` 拿到参数与查询串：

`views/admin/employees/show.svelte`

```svelte
<script>
  export let currentRoute
  // 访问 /app/admin/employees/show/42?tab=profile#notes 时：
  $: id = currentRoute.namedParams.id   // "42"
  $: tab = currentRoute.queryParams.tab // "profile"
  $: anchor = currentRoute.hash         // "#notes"
</script>
```

### 解析与生效顺序

给定一个 URL，路由器（`SpaRouter` -> `RouterFinder`）按下面的顺序求值。理解这个顺序，基本就能预测任何叠加场景的结果：

1. **去掉尾部斜杠**。`/app/admin/` 与 `/app/admin` 等价。
2. **剥离前缀（在匹配之前）**。设了 `prefix` 时，URL 里的前缀会先被去掉再去匹配路由树；匹配成功后，前缀会被重新拼回到 `currentRoute.path`。**没有以前缀开头的链接，路由器根本不接管**（见下一节），因此它们不会被判成 404。
3. **解析 URL**：拆成路径段数组、`queryParams`（对象）、`hash`。
4. **沿路由树逐段下钻匹配**。每一层取当前路径段去和该层的路由比对：
   - **静态段优先于命名参数段**。同一层里只要有一个静态名（或其本地化名）匹配上，`:命名参数` 路由在这一层就不再参与匹配。
   - 因此 **把命名参数作为路径“第一段”的路由（如 `/:user/edit`）必须写在所有静态路由之后**（详见上文 “Using named params as first part of path name”）。
5. **本地化在“逐段匹配”里一起发生**：
   - 如果在 `options` 里设了 `lang`，则**只**接受该语言的名字，其它语言/默认名会落到 404。
   - 如果**没有**设 `lang`，则默认名或**任意** `lang` 变体都能匹配，命中哪个就把对应语言写进 `currentRoute.language`（无则用 `defaultLanguage`）。同一路径里不同段可以命中不同语言。
6. **每命中一层就收集一次“重定向”**，优先级为：继承自上层的值 → 若本层有 `redirectTo` 则覆盖 → 若本层 `onlyIf` 守卫**判定失败**则再覆盖为守卫的 `redirect`。也就是说 **`onlyIf` 守卫的重定向优先级高于 `redirectTo`**，且**父层的重定向会沿子链一直生效**（父级守卫即可保护整棵子树）。
7. **自动 `index`**：某层路由有 `nestedRoutes` 但路径段已经用完时，会自动尝试名为 `index` 的子路由。
8. **404 是最后才判定的**。只有在“**完全没匹配上**”或“**匹配了父级但必需的子段没匹配上（子链不完整）**”时才落到 404：
   - 若你定义了 `name: '404'` 的路由 → **原地渲染**它（路径标记为 `404`）。
   - 否则 → 默认行为是重定向到静态页 `/404.html`。
9. **最后执行重定向**：`setActiveRoute` 发现解析结果带有重定向目标时，通过导航跳转过去（而不是渲染当前结果）。

> 走查示例：访问 `/app/administrador/empleados/ver/42?tab=profile`（`options = { prefix: 'app' }`，未设 `lang`）
>
> 1. 去尾斜杠 →（无变化）
> 2. 剥离前缀 `app` → `/administrador/empleados/ver/42?tab=profile`
> 3. 解析 → 段 `[administrador, empleados, ver, 42]`，`queryParams = { tab: 'profile' }`
> 4–5. 顶层 `administrador` 命中 `admin` 的 `es` 变体 → `language = 'es'`；`admin` 上有守卫 → 先收集（若失败则整链重定向 `/login`）
> 6. 下钻 `empleados` 命中 `employees`（es），再下钻 `ver/42` 命中 `show/:id`（es），`:id = 42`
> 7. 段用尽，无需自动 index
> 8. 未触发 404；把前缀拼回 → `currentRoute.path = /app/administrador/empleados/ver/42`
> 9. 结果：`language='es'`、`namedParams={ id: '42' }`、`queryParams={ tab: 'profile' }`
>
> 若此时守卫 `userIsLoggedIn()` 返回 `false`：第 4 步就把重定向收集为 `/login`，第 8 步不再渲染，第 9 步直接跳转到 `/login`——**整棵 admin 子树都被守住**。

### 普通链接与组件跳转的差别

三种跳转方式行为并不等价，尤其在**带查询串**时差异很大：

| 维度 | 普通 `<a href>` | `<Navigate to>` | `navigateTo()` |
| --- | --- | --- | --- |
| 触发方式 | 全局 `window` 的 click 监听拦截 | 组件自身 `on:click` 内直接调用 | 直接函数调用（程序内跳转） |
| 是否拦截整页刷新 | 仅**同源**链接；设了 `prefix` 时**仅** `/prefix` 下的路径会被拦截，其它路径走浏览器整页跳转 | 是（`preventDefault` + `stopPropagation`） | 不适用 |
| 查询串处理 | ⚠️ 见“注意事项”：URL 被拼成 `pathname + search + search + hash`，**查询串会被重复**、解析错乱 | 原样传入，正确 | 原样传入，正确 |
| `active` 高亮类 | 需自己用 `routeIsActive` 配合 `class:active` | **自动**添加（内部用 `routeIsActive(to)`，精确匹配，`includePath=false`） | 不适用 |
| 多语言转换 | 需自己写好目标路径 | 传 `lang` 属性，挂载时自动转成该语言路径 | 第 2 个参数传 `language`，自动转 |
| 修饰键 / 新标签 | 浏览器默认；`cmd/ctrl/shift` 点击不拦截；`target="_blank"` 走 `window.open` | `cmd/ctrl/shift` 点击不拦截，交回浏览器 | 不适用 |
| 是否写入浏览器历史 | 写入（`pushState`） | 写入 | 默认写入；第 3 个参数传 `false` 可**不**写历史 |

选型建议：

- **纯静态、无查询串**的简单链接：用普通 `<a href>` 即可。
- **带查询串 / 需要 active 高亮 / 需要多语言**：用 `<Navigate>`。
- **程序内跳转、或需要控制是否写历史**：用 `navigateTo(path, language?, updateBrowserHistory?)`。

```svelte
<script>
  import { Navigate, navigateTo } from 'svelte-router-spa'
</script>

<!-- 组件跳转：查询串安全、自动 active、可本地化 -->
<Navigate to="/app/admin/employees/show/42?tab=profile">查看员工 42</Navigate>
<Navigate to="login" lang="es">Acceso</Navigate>

<!-- 程序跳转 -->
<button on:click={() => navigateTo('/app/admin/employees?page=2')}>下一页</button>
<!-- 替换式跳转：不往历史里压新记录 -->
<button on:click={() => navigateTo('/app/admin', null, false)}>回到后台</button>
```

### 参数与查询串如何到达 currentRoute

`<Route>` 会把一个 `currentRoute` 对象传给它渲染的每个布局/组件，参数都在里面：

- **`namedParams`**：路径里 `:xxx` 占位对应的值，按占位出现的位置从路径段取值。**父子会合并**——父层路由的命名参数在更深的子组件里依然可见。
- **`queryParams`**：`?a=1&b=2` 解析成的对象（重复键时**后者覆盖前者**）。
- **`hash`**：URL 的 `#片段`（含 `#`）。
- 此外还有 `name`、`path`、`component`、`layout`、`language`、`childRoute`。

在组件里直接 `export let currentRoute` 读取即可：

```svelte
<script>
  export let currentRoute
  // /app/admin/employees/show/42?tab=profile&sort=name#top
  $: id = currentRoute.namedParams.id        // "42"
  $: tab = currentRoute.queryParams.tab      // "profile"
  $: sort = currentRoute.queryParams.sort    // "name"
  $: anchor = currentRoute.hash              // "#top"
  $: lang = currentRoute.language            // "en" / "es" ...
</script>
```

> 提示：`namedParams` / `queryParams` 的值都是**字符串**。需要数字、布尔时请自行转换（如 `Number(id)`）。

### 守卫拦截与返回上一页

**守卫语义（`onlyIf`）**：

- `onlyIf = { guard, redirect }`。只有当 `guard` 是一个函数时守卫才“有效”；**`guard()` 返回假值（falsy）就触发重定向**到 `redirect`（未填 `redirect` 时默认回到 `/`）。
- 守卫的重定向**优先级高于** `redirectTo`；挂在父级上的守卫**保护整棵子树**（见“解析与生效顺序”第 6 步）。

**正常的“后退/前进”**：浏览器的 `popstate`（点返回键）会以 `updateBrowserHistory = false` 重新解析并渲染，**不会再往历史里压记录**——这是期望行为。

**但要小心“守卫重定向 + 返回键”的组合**：当某次解析结果带有重定向目标时，对**非 404** 的目标，路由器是用**默认 `updateBrowserHistory = true`** 去执行 `navigateTo` 的，会**压入一条新的历史记录**。于是会出现：

> 你被守卫从 `/app/admin` 弹到 `/login`，登录后到了别的页；此时按“返回键”想回到上一页，却又落到受守卫的 `/app/admin`，守卫再次失败、再次 `navigateTo('/login')` 并**又压入一条历史**——形成“返回键回不去 / 历史里多了记录”的现象。
>
>（默认的 `/404.html` 重定向是个例外，它会沿用 `updateBrowserHistory = false`，**不**压历史。）

规避建议：尽量在**发起跳转之前**就判断权限（例如点击时先检查再 `navigateTo`），把守卫当作“兜底”而非唯一拦截点；或在设计返回逻辑时预期到这一行为。

### 未匹配与重定向的优先级

- **匹配成功且带重定向** → 永远走重定向，**不会**变成 404。
- **父级匹配上、但必需的子段没匹配上**（子链不完整）→ 判为 **404**，而不是“退而渲染父级”。例如配置里 `employees` 必须再有合法子段时，`/app/admin/employees/不存在` 会 404。
- **完全没匹配上** → 404。
- **前缀范围外的链接** → 路由器**不接管**（交给浏览器整页跳转），因此不算 404。

404 的两种表现见“解析与生效顺序”第 8 步：有 `name: '404'` 路由则**原地渲染**，否则**重定向到 `/404.html`**。

### 类型说明

> 按你的选择，本节**只在文档里说明类型**，不改动随包发布的 `.d.ts`。下面的形状直接对应库内自带的声明文件，可作为你手写类型注解的参考。

**路由配置类型（对应 `types/components/router.d.ts`）**

```ts
type Language = Record<string, string> | string

type Route = {
  name: string
  component?: typeof SvelteComponent
  layout?: typeof SvelteComponent
  nestedRoutes?: Route[]
  redirectTo?: string
  onlyIf?: {
    // ⚠️ 类型虽允许 Promise<boolean>，但运行时不会 await，异步守卫实际不生效，
    //    见“注意事项与已知陷阱”。请只返回同步布尔值。
    guard: (...args: any) => boolean | Promise<boolean>
    redirect: string
  }
  lang?: Language
}

type RouterOptions = Partial<{
  prefix: string
  gaPageviews: boolean
  lang: Language
  defaultLanguage: string
}>
```

**`currentRoute` 的类型（对应 `types/components/route.d.ts`）**

```ts
type CurrentRoute = {
  name: string
  path: string
  hash: string
  component?: typeof SvelteComponent
  layout?: typeof SvelteComponent
  queryParams: Record<string, string>
  namedParams: Record<string, string>
  childRoute: CurrentRoute
  language?: string
}
```

**`<Navigate>` 的属性**：`to: string`（必填）、`title?: string`、`styles?: string`、`lang?: string`。

**导出与导入**：从包入口 `svelte-router-spa` 导出的是组件（`Router` / `Route` / `Navigate`）和函数（`SpaRouter` / `navigateTo` / `localisedRoute` / `routeIsActive`）。上面的 `Route`（路由配置）与 `RouterOptions` 类型**目前并未从包入口再导出**——它们存在于随包发布的声明文件里，但需要深层路径才能引用，能否直接 `import type` 取决于你的打包器/TS 解析配置。最稳妥的做法是按上面的形状在你自己的项目里写一份类型别名来约束 `routes`。

**容易被误解的声明**（请以实际运行行为为准，不要据声明推断）：

- `spa_router.d.ts` 里 `SpaRouter` 的 `options` 被声明成空对象 `{}`，实际应按上面的 `RouterOptions` 传入。
- `localisedRoute(name, language)` 声明的返回类型是 `{ redirectTo: string }`，**但运行时返回的是整个路由对象**——本地化后的字符串路径在它的 **`.path`** 字段上（`<Navigate>` 内部正是取 `route.path`）。
- `SpaRouter(...).findActiveRoute()` 返回的同样是完整的路由对象，而非只有 `redirectTo`。

### 注意事项与已知陷阱

以下都是**当前版本的真实行为**，本节只做记录与规避，不改动库源码：

1. **带查询串的普通 `<a>` 会重复查询串。** 全局点击拦截在拼接目标 URL 时把查询串拼了两次（`pathname + search + search + hash`），于是 `/x?a=1` 会变成 `/x?a=1?a=1`，解析出的 `queryParams` 会错乱。**规避**：带查询串的链接用 `<Navigate>` 或 `navigateTo()`。
2. **守卫/重定向在“返回键”时会污染历史。** 正常后退不写历史；但解析结果带重定向、且目标不是默认 404 时，会以默认 `updateBrowserHistory = true` 执行跳转、**压入新历史**，导致“返回键回不去”。详见 [守卫拦截与返回上一页](#守卫拦截与返回上一页)。
3. **异步守卫实际不生效。** 运行时是 `!guard()` 同步取反，若 `guard` 返回 `Promise`，`!Promise` 恒为 `false`，等于守卫**永远通过、从不拦截**。**守卫函数必须同步返回布尔值。**
4. **`localisedRoute` 返回的是路由对象、不是字符串。** 需要本地化后的字符串路径请取 `.path`。
5. **首段为命名参数的路由要放最后。** 形如 `/:user/edit` 的路由必须排在所有静态路由之后，否则可能抢先匹配（见上文 “Using named params as first part of path name”）。

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
