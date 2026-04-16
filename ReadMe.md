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

And inside each feature:

// features/customers/routes.ts
export const CUSTOMERS_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/customer-list-page.component').then(
        m => m.CustomerListPageComponent
      ),
  },
  {
    path: ':id',
    loadComponent: () =>
      import('./pages/customer-detail-page.component').then(
        m => m.CustomerDetailPageComponent
      ),
  },
];

This keeps features isolated and makes code splitting natural.

Separate “page”, “smart”, and “presentational” responsibilities

A useful convention:

pages/

Route-level containers.
They:

read route params
call facades/stores
compose feature UI
handle navigation
components/

Feature-specific UI pieces.
They:

receive inputs
emit outputs
avoid direct API calls when possible
shared/ui/

Reusable, app-agnostic UI components.
They:

should not know feature business rules
should be reusable across many features

This dramatically reduces bloated components.

Put data access behind a boundary

Inside each feature, create a small data-access layer:

features/
  customers/
    data-access/
      customers-api.service.ts

That service should own:

HTTP calls
DTO mapping
API-specific concerns
retry / caching policies if needed

Your components should not directly know endpoint details.

Keep state close to the feature

Angular 21 strongly supports reactive patterns with signals, which are now a core part of Angular’s direction.

For many apps, a simple feature store is enough:

@Injectable()
export class CustomersStore {
  private readonly api = inject(CustomersApiService);

  readonly customers = signal<Customer[]>([]);
  readonly loading = signal(false);
  readonly error = signal<string | null>(null);

  async load(): Promise<void> {
    this.loading.set(true);
    this.error.set(null);

    try {
      const data = await this.api.getCustomers();
      this.customers.set(data);
    } catch {
      this.error.set('Failed to load customers');
    } finally {
      this.loading.set(false);
    }
  }
}

Good rule:

local UI state → component signals
feature state → feature store/facade
app-wide cross-cutting state → only if truly global

Do not make everything global.

Keep core small and intentional

core/ is for app-wide singleton or infrastructure concerns only.

Good examples:

auth
interceptors
app config
logging
shell layout
global error handling

Bad examples:

feature-specific business logic
random reusable helpers with no clear ownership
everything you do not know where to place

If core grows too large, it becomes a dumping ground.

Keep shared even smaller

shared/ should contain only things that are:

generic
reusable
business-agnostic

Good:

button
dialog wrapper
date pipe
loading spinner
input mask directive

Bad:

TradeOrderSummaryCardComponent
CustomerRiskBadgeComponent

Those belong in a feature unless truly reused across multiple business domains.

Naming conventions that scale

Angular’s style guide emphasizes consistency in naming and colocating related files.

A practical convention:

customer-list-page.component.ts
customer-filter.component.ts
customers-api.service.ts
customers.store.ts
customer.models.ts

Use:

singular for a single entity component: customer-card.component.ts
plural for collection/state/services when appropriate: customers.store.ts
suffix by role: .component.ts, .service.ts, .store.ts, .guard.ts

Avoid vague names like:

common.service.ts
helper.ts
manager.ts
base.component.ts
Recommended dependency direction

Keep dependencies flowing inward:

app -> feature page -> feature state/facade -> data-access -> API
                     -> feature components
shared UI can be used by any feature
core is used by app/features, but core should not depend on features

Important rule:
features should not casually import from each other.

If many features need the same thing:

move it to shared/ if UI-generic
move it to core/ if infrastructure
move it to a dedicated library/domain package in larger workspaces
Use libraries when the app gets bigger

If your codebase becomes large, use an Angular workspace with application + libraries. Angular’s workspace structure officially supports multiple projects in one workspace.

A bigger setup might become:

projects/
  app-shell/
  libs/
    shared-ui/
    shared-util/
    auth/
    customers/
    orders/

This is especially useful when:

team is growing
domains are large
you need stricter boundaries
multiple apps share code

For a normal single app, do not start with too many libraries too early.

Recommended file split inside a feature

A good medium-sized feature looks like this:

features/
  customers/
    routes.ts
    pages/
      customer-list-page.component.ts
      customer-detail-page.component.ts
    components/
      customer-table.component.ts
      customer-filter.component.ts
      customer-form.component.ts
    data-access/
      customers-api.service.ts
      customer.mapper.ts
    state/
      customers.store.ts
      customers.facade.ts
    models/
      customer.models.ts
      customer.filters.ts

Use:

store.ts for reactive state
facade.ts only if it adds value as a public API over store/services
mapper.ts when DTOs differ from UI models
models.ts for types/interfaces
Avoid the most common Angular architecture mistakes
1. Fat components

Component does routing, HTTP, validation, transformation, and state updates.

Better:

component handles view interaction
store/facade handles state
service handles API
2. Everything in shared

If everything is reusable, nothing is understandable.

Better:

feature first
promote to shared only after real reuse appears
3. Too much global state

Not every filter, modal flag, or form state belongs in app-wide state.

Better:

keep state local unless multiple distant parts truly depend on it
4. Cross-feature imports

One feature starts importing internals from another feature.

Better:

define public boundaries
extract common domain pieces if needed
5. Giant app.component

Putting layout, auth checks, menus, and startup logic there.

Better:

keep app.component thin
move shell/layout into dedicated components
Recommended Angular 21 conventions

These are good defaults for modern Angular:

standalone components
signals for local/feature state
built-in control flow syntax
route-based lazy loading
inject() instead of constructor injection where it improves readability
deferrable views for expensive UI when useful
strict TypeScript mode

Angular 21 highlights continued investment in signals, performance, and the updated style guidance.

A practical starter blueprint

If I were starting a fresh Angular 21 business app today, I would use this:

src/app/
  app.config.ts
  app.routes.ts

  core/
    auth/
    http/
    layout/
    config/

  shared/
    ui/
    pipes/
    directives/
    utils/

  features/
    dashboard/
    customers/
    orders/
    admin/

Inside each feature:

feature/
  routes.ts
  pages/
  components/
  data-access/
  state/
  models/

That is simple, scalable, and easy for teams to follow.

Rule of thumb for deciding where code goes

Ask:

Is it specific to one business area?
Put it in that feature.
Is it reusable UI with no business meaning?
Put it in shared.
Is it app-wide infrastructure?
Put it in core.
Is it route-entry UI?
Put it in pages.
Is it backend communication?
Put it in data-access.
Is it reactive feature state?
Put it in state.
My recommendation in one sentence

For Angular 21, build standalone, feature-first, route-lazy-loaded applications, with thin components, feature-scoped state, a small core, and a very selective shared layer.
