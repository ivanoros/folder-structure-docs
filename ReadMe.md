For Angular 21, the cleanest default is:

organize by feature, use standalone APIs, lazy-load by route, keep shared code small, and keep business logic out of components. Angular’s current docs emphasize standalone architecture, an updated style guide, and route-level lazy loading as first-class patterns.

The mental model

A maintainable Angular app usually has 5 layers:

App shell
Bootstrap, global providers, top-level routing, layout.
Features
Real business areas like orders, customers, portfolio, admin.
Shared UI
Reusable presentational pieces like buttons, grids, cards, dialogs, pipes, directives.
Core / infrastructure
Auth, HTTP setup, interceptors, API clients, config, logging, guards.
Domain logic / state
Types, business rules, facades, stores, data-access services.

That structure keeps you from ending up with a giant components/, services/, models/ folder soup.

Best-practice folder structure

For a single Angular application, this is a strong default:
src/
  app/
    app.config.ts
    app.routes.ts
    app.component.ts
    app.component.html
    app.component.css

    core/
      config/
        app-env.ts
      http/
        api-client.ts
        auth.interceptor.ts
        error.interceptor.ts
      auth/
        auth.service.ts
        auth.guard.ts
      layout/
        shell.component.ts
        header.component.ts
        sidebar.component.ts

    shared/
      ui/
        button/
          app-button.component.ts
        loading-spinner/
          loading-spinner.component.ts
      directives/
        autofocus.directive.ts
      pipes/
        currency-format.pipe.ts
      utils/
        date.utils.ts

    features/
      dashboard/
        routes.ts
        pages/
          dashboard-page.component.ts
        components/
          dashboard-summary.component.ts
          dashboard-chart.component.ts
        data-access/
          dashboard-api.service.ts
        state/
          dashboard.store.ts
        models/
          dashboard.models.ts

      customers/
        routes.ts
        pages/
          customer-list-page.component.ts
          customer-detail-page.component.ts
        components/
          customer-table.component.ts
          customer-filter.component.ts
        data-access/
          customers-api.service.ts
        state/
          customers.store.ts
        models/
          customer.models.ts

      orders/
        routes.ts
        pages/
        components/
        data-access/
        state/
        models/

This matches Angular’s style guide direction to keep code consistent and its workspace/project structure guidance, while adapting it to a modern feature-first layout.

Prefer feature-first over type-first

Avoid this as your main structure:
components/
services/
models/
guards/
pipes/

It looks neat at first, but as the app grows, related code gets scattered everywhere.

Prefer this instead:
features/
  customers/
    pages/
    components/
    data-access/
    state/
    models/

Why this works better:

easier to find related code
safer refactoring
better boundaries between business areas
simpler lazy loading
Use standalone by default

Angular’s current direction is to use standalone components, directives, and pipes rather than building everything around NgModules. Angular provides migration support for this and positions standalone as the simplified authoring model.

Use:

bootstrapApplication(...)
app.config.ts
provideRouter(...)
standalone components
route-level provider scoping when helpful

That gives you fewer files, clearer dependencies, and better lazy-loading ergonomics.

Route-level lazy loading should be the default

Angular’s docs recommend eager loading for primary landing pages and lazy loading for most other routes. Angular also ships a migration specifically for lazy-loaded routes.

A strong pattern is:

// app.routes.ts
export const appRoutes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./core/layout/shell.component').then(m => m.ShellComponent),
    children: [
      {
        path: '',
        pathMatch: 'full',
        redirectTo: 'dashboard',
      },
      {
        path: 'dashboard',
        loadChildren: () =>
          import('./features/dashboard/routes').then(m => m.DASHBOARD_ROUTES),
      },
      {
        path: 'customers',
        loadChildren: () =>
          import('./features/customers/routes').then(m => m.CUSTOMERS_ROUTES),
      },
    ],
  },
];