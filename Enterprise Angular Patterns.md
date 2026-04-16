Yes — these two patterns are worth understanding well, because they solve two different problems:

Feature-based architecture solves how you organize the codebase
Smart vs presentational components solves how you organize responsibilities inside the UI

They work very well together in Angular 21, especially with standalone components, route-based lazy loading, DI, and signals. Angular’s current docs support the underlying pieces: consistent structure, standalone components by default, routing as a core organizing mechanism, lazy-loaded routes, signals for reactive state, and DI/services for shared logic.

1. Feature-based architecture
What it is

Feature-based architecture means you organize the app around business capabilities, not technical file types.

Instead of this:

components/
services/
models/
guards/
pipes/

you do this:

features/
  customers/
  orders/
  portfolio/
  admin/

and each feature contains most of what it needs.

Angular’s style guide does not prescribe one rigid folder tree, but it does emphasize consistency and keeping related code together, and Angular’s routing model makes it natural to organize code around routeable slices of the app. Angular also provides official support for converting eagerly loaded routes into lazy-loaded routes, which reinforces route/feature boundaries.

Why people use it

This pattern becomes valuable once the app stops being tiny.

With type-based organization, one feature gets split across many folders:

UI in components
API calls in services
types in models
routing somewhere else
guards elsewhere again

That makes a single business change harder, because the code is scattered.

With feature-based organization, most of the change lives in one place. That reduces search time, lowers accidental coupling, and makes lazy loading easier because the feature already has a clean boundary. Angular’s router and lazy-loading support line up very naturally with this style.

What it looks like in Angular 21

A practical Angular 21 version usually looks like this:

src/app/
  core/
  shared/
  features/
    customers/
      routes.ts
      pages/
      components/
      data-access/
      state/
      models/
    orders/
      routes.ts
      pages/
      components/
      data-access/
      state/
      models/

Why this works well in modern Angular:

standalone components reduce NgModule overhead
route files define feature entry points cleanly
lazy loading can happen at the feature boundary
signals let you keep state local to the feature instead of reaching for global state too early
The main idea

A feature is a vertical slice:

UI
state
routing
data access
models

all grouped by one business purpose.

That is why people also call it vertical slice architecture.

Good boundaries

Inside a feature:

pages/ for route-entry components
components/ for feature-specific UI pieces
data-access/ for HTTP/API integration
state/ for stores/facades/signals
models/ for types

Outside a feature:

shared/ only for generic reusable UI or utilities
core/ only for app-wide infrastructure like auth, interceptors, config, shell layout

That separation matches Angular’s DI model and hierarchical providers: shared infrastructure can be global, while feature-specific services/providers can stay scoped more locally.

When feature-based architecture helps most

It is especially useful when:

the app has several business areas
multiple developers work on it
you want route-based lazy loading
features evolve independently
you want clearer ownership boundaries
When not to overdo it

For a very small app, a heavy feature structure can feel like ceremony. In that case, you can still think feature-first, but keep it light.

For example, one small feature might just be:

features/
  dashboard/
    dashboard-page.component.ts
    dashboard-api.service.ts
    dashboard.store.ts
    routes.ts

You can add more subfolders later.

2. Smart vs presentational components
What it is

This pattern is about splitting UI components by responsibility.

A smart component:

knows where data comes from
talks to services/store/facade
handles routing
coordinates business actions

A presentational component:

receives data via inputs
emits user actions via outputs
focuses on rendering and interaction
does not know where data came from

Angular’s component communication model strongly supports this: parents pass data to children with inputs, and children emit events upward with outputs. Angular’s newer signal input/output APIs make that pattern even more natural.

Where the idea comes from

Angular doesn’t have an official page called “smart vs presentational,” but the pattern is a frontend expression of older separation-of-concerns ideas. Martin Fowler’s Separated Presentation describes keeping presentation code separate from domain code, and Presentation Model describes extracting view behavior into a model that is independent of the concrete UI framework.

So even though the exact labels “smart” and “presentational” are community terms, the architectural principle behind them is well established.

Example
Smart/page component
@Component({
  standalone: true,
  selector: 'app-customer-list-page',
  template: `
    <app-customer-filter
      [filter]="store.filter()"
      (filterChange)="store.setFilter($event)"
    />

    <app-customer-table
      [customers]="store.filteredCustomers()"
      [loading]="store.loading()"
      (refresh)="store.load()"
      (selectCustomer)="openCustomer($event)"
    />
  `,
  imports: [CustomerFilterComponent, CustomerTableComponent]
})
export class CustomerListPageComponent {
  readonly store = inject(CustomersStore);
  private readonly router = inject(Router);

  openCustomer(id: string) {
    this.router.navigate(['/customers', id]);
  }
}
Presentational component
@Component({
  standalone: true,
  selector: 'app-customer-table',
  template: `
    @if (loading()) {
      <p>Loading...</p>
    } @else {
      <table>
        @for (customer of customers(); track customer.id) {
          <tr (click)="selectCustomer.emit(customer.id)">
            <td>{{ customer.name }}</td>
            <td>{{ customer.status }}</td>
          </tr>
        }
      </table>
      <button (click)="refresh.emit()">Refresh</button>
    }
  `
})
export class CustomerTableComponent {
  customers = input.required<Customer[]>();
  loading = input(false);

  refresh = output<void>();
  selectCustomer = output<string>();
}

This lines up closely with Angular’s documented component communication model using input() and output().

Why this pattern helps

It makes presentational components:

easier to reuse
easier to test
easier to preview in isolation
easier to reason about

And it keeps data fetching and coordination logic from leaking into every leaf component.

Angular’s DI and service model also support this split well: shared logic can live in injectable services or stores, rather than being duplicated across components.

A simple rule of thumb

A component is probably presentational if:

it mostly renders what it is given
it has many input()s and output()s
it has little or no routing/API/store knowledge

A component is probably smart if:

it injects services or a store
it reads route params
it performs orchestration
it translates user intent into application actions
In Angular 21 terms

A good modern split is:

page components = smart/container components
UI components = presentational components
store/services = data/business orchestration

That fits very well with:

standalone components
route-level lazy loading
signals for local and feature state
DI for service boundaries
How the two patterns work together

These patterns are complementary.

Feature-based architecture says:

“Put everything related to customers together.”

Smart vs presentational says:

“Inside customers, separate coordination logic from display logic.”

So you might end up with:

features/
  customers/
    routes.ts
    pages/
      customer-list-page.component.ts       <-- smart
      customer-detail-page.component.ts     <-- smart
    components/
      customer-table.component.ts           <-- presentational
      customer-filter.component.ts          <-- presentational
      customer-summary-card.component.ts    <-- presentational
    data-access/
      customers-api.service.ts
    state/
      customers.store.ts
    models/
      customer.models.ts

That is one of the cleanest setups for a medium-to-large Angular app.

Common mistakes with these patterns
Mistake 1: making every component presentational

That creates awkward plumbing and too many wrapper components.

Use the split where it adds clarity, not as a religion.

Mistake 2: smart components that do everything

If a page component does routing, fetching, transformation, filtering, validation, and rendering, it becomes hard to maintain.

Move repeated logic into a store/service.

Mistake 3: putting feature-specific UI into shared

If CustomerRiskBadgeComponent is only for customers, it belongs in the customers feature.

Mistake 4: confusing “presentational” with “stateless”

Presentational components can still have local UI state, such as:

expanded/collapsed
active tab
hover state
local sort order

What they should avoid is owning broad business workflow state unless that is truly their job.

Mistake 5: cross-feature leakage

One feature starts importing another feature’s internal store/components directly. That weakens the feature boundary.

Best practical version for Angular 21

If you want a balanced approach, I would recommend:

Use feature-based architecture for:
top-level folder structure
routing boundaries
lazy loading boundaries
ownership boundaries
Use smart vs presentational for:
page-level components
reusable feature UI
testability and maintainability
Keep this balance:
fetch/orchestrate in pages/ + state/
render/interact in components/
call backend in data-access/

That gives you a structure that feels natural in Angular and scales well.

Useful references

Here are the references I’d actually recommend reading.

Angular official docs

Angular Style Guide
Best for consistency, naming, structure, and colocating related code.


Angular Routing Overview
Best for understanding routing as a core architectural boundary.


Lazy-loaded routes migration / lazy routing direction
Useful because it shows Angular is actively pushing route-level lazy loading as a first-class modern pattern.


Components guide
Confirms standalone-by-default modern component model.


Signals guide
Best for modern state thinking in Angular.


Dependency Injection overview
Best for understanding how services/stores fit into clean boundaries.


Creating and using services
Useful for the “move logic into services/store” part.


Input / Output docs
Best for the mechanics behind presentational components.


Architecture references

Martin Fowler — Separated Presentation
Very useful to understand the older, deeper principle behind separating UI from domain/application logic.


Martin Fowler — Presentation Model
Helpful for understanding why view logic is often extracted away from raw UI widgets/components.


Martin Fowler — GUI Architectures
Good broader context if you want to understand where these frontend patterns come from conceptually.