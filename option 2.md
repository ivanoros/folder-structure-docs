for a more enterprise-shaped Angular 21 app, I’d keep the same feature-first + smart/presentational approach, but add a few stronger boundaries:

core/ for app-wide infrastructure like HTTP setup and interceptors
shared/ui/ for reusable presentational building blocks
feature data-access/ for API calls only
feature state/ for signal-based page state
reactive forms for edit screens
route lazy loading for feature boundaries
optionally resolvers for detail routes when preloading data improves UX

Those choices line up with Angular’s current guidance around standalone components, lazy-loaded routes, signals, HttpClient, functional interceptors, and reactive forms. Angular also marks Signal Forms as experimental, so for a production-style scaffold I would still use standard reactive forms today.

Enterprise scaffold
src/
  main.ts
  styles.css
  app/
    app.component.ts
    app.config.ts
    app.routes.ts

    core/
      config/
        api.config.ts
      http/
        api-base-url.interceptor.ts
        http-error.interceptor.ts
      layout/
        app-shell.component.ts

    shared/
      ui/
        page-header/
          page-header.component.ts
        loading-spinner/
          loading-spinner.component.ts
        error-alert/
          error-alert.component.ts
        empty-state/
          empty-state.component.ts

    features/
      customers/
        routes.ts
        models/
          customer.models.ts
        data-access/
          customers-api.service.ts
          customer.mapper.ts
        state/
          customers-list.store.ts
        resolvers/
          customer-detail.resolver.ts
        pages/
          customer-list-page.component.ts
          customer-detail-page.component.ts
          customer-edit-page.component.ts
        components/
          customer-filter.component.ts
          customer-table.component.ts
          customer-form.component.ts
Why this is more “enterprise”

This version gives you:

a shared UI layer that stays generic
a feature data-access layer that hides HTTP details
a detail page + edit page
loading/error abstraction
reactive form editing
functional interceptors for cross-cutting HTTP concerns, which Angular recommends over DI-based interceptors
an optional resolver for the detail page, which Angular documents for fetching route data before activation

How the patterns show up here
Feature-based architecture

Everything for customers lives under features/customers. That makes the feature easy to own, test, and lazy-load. Angular’s routing and lazy-loading model fit this especially well.

Smart vs presentational

The smart components are the pages:

customer-list-page.component.ts
customer-detail-page.component.ts
customer-edit-page.component.ts

The presentational components are:

customer-filter.component.ts
customer-table.component.ts
customer-form.component.ts
shared UI components like page-header, error-alert, loading-spinner

That keeps routing, orchestration, and backend interaction out of the leaf UI pieces.

What I would keep from this in a real app

Use this version when you want:

one or more teams working in parallel
route-level feature boundaries
reusable page chrome and feedback UI
testable data-access and form code
a path to scale without jumping straight into Nx or microfrontends
Practical next steps

The two most useful additions after this would be:

swap the mocked of(...).pipe(delay(...)) calls for a real backend
add unit tests around customers-list.store, customer.mapper, and customer-form.component

If you want, I can turn this into a full copy-paste runnable Angular 21 scaffold with every file included, including main.ts, styles.css, and the exact imports so you can paste it into a fresh project without filling gaps yourself.