Below is a small Angular 21 sample feature that demonstrates both patterns together:

Feature-based architecture
Smart vs presentational components

I’ll use a simple Customers feature.

The example includes:

feature folder structure
route entry
smart page component
presentational components
store
API/data-access service
models

This is meant to be small but realistic.

Folder structure
src/app/
  features/
    customers/
      routes.ts
      models/
        customer.models.ts
      data-access/
        customers-api.service.ts
      state/
        customers.store.ts
      pages/
        customer-list-page.component.ts
      components/
        customer-filter.component.ts
        customer-table.component.ts
What each file is doing
Feature-based architecture

Everything related to customers is grouped together:

routes
models
data access
state
pages
components
Smart vs presentational
customer-list-page.component.ts → smart
customer-filter.component.ts → presentational
customer-table.component.ts → presentational

The page coordinates everything.
The presentational components only render inputs and emit outputs.

1) customer.models.ts
export interface Customer {
  id: number;
  name: string;
  city: string;
  status: 'Active' | 'Inactive';
}

export interface CustomerFilter {
  search: string;
  status: 'All' | 'Active' | 'Inactive';
}
2) customers-api.service.ts

This is the data-access layer.

In a real app, this would call HTTP endpoints.
For the sample, I’ll mock the response.

import { Injectable } from '@angular/core';
import { Customer } from '../models/customer.models';

@Injectable({ providedIn: 'root' })
export class CustomersApiService {
  async getCustomers(): Promise<Customer[]> {
    await new Promise(resolve => setTimeout(resolve, 500));

    return [
      { id: 1, name: 'Acme Capital', city: 'New York', status: 'Active' },
      { id: 2, name: 'Blue River Partners', city: 'Chicago', status: 'Inactive' },
      { id: 3, name: 'Northwind Trading', city: 'Boston', status: 'Active' },
      { id: 4, name: 'Summit Financial', city: 'Dallas', status: 'Active' }
    ];
  }
}
3) customers.store.ts

This is the feature state layer using signals.

It loads customers, stores filter state, and exposes filtered results.

import { computed, Injectable, inject, signal } from '@angular/core';
import { CustomersApiService } from '../data-access/customers-api.service';
import { Customer, CustomerFilter } from '../models/customer.models';

@Injectable()
export class CustomersStore {
  private readonly api = inject(CustomersApiService);

  readonly customers = signal<Customer[]>([]);
  readonly loading = signal(false);
  readonly error = signal<string | null>(null);

  readonly filter = signal<CustomerFilter>({
    search: '',
    status: 'All'
  });

  readonly filteredCustomers = computed(() => {
    const customers = this.customers();
    const filter = this.filter();

    return customers.filter(customer => {
      const matchesSearch =
        filter.search.trim() === '' ||
        customer.name.toLowerCase().includes(filter.search.toLowerCase()) ||
        customer.city.toLowerCase().includes(filter.search.toLowerCase());

      const matchesStatus =
        filter.status === 'All' || customer.status === filter.status;

      return matchesSearch && matchesStatus;
    });
  });

  async load(): Promise<void> {
    this.loading.set(true);
    this.error.set(null);

    try {
      const data = await this.api.getCustomers();
      this.customers.set(data);
    } catch (error) {
      console.error(error);
      this.error.set('Failed to load customers.');
    } finally {
      this.loading.set(false);
    }
  }

  setSearch(search: string): void {
    this.filter.update(current => ({
      ...current,
      search
    }));
  }

  setStatus(status: CustomerFilter['status']): void {
    this.filter.update(current => ({
      ...current,
      status
    }));
  }
}
4) customer-filter.component.ts

This is a presentational component.

It does not know anything about services, stores, or routing.
It only takes inputs and emits outputs.

import { Component, input, output } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CustomerFilter } from '../models/customer.models';

@Component({
  selector: 'app-customer-filter',
  standalone: true,
  imports: [FormsModule],
  template: `
    <div class="filter-panel">
      <label>
        Search:
        <input
          type="text"
          [ngModel]="filter().search"
          (ngModelChange)="searchChange.emit($event)"
          placeholder="Search by name or city"
        />
      </label>

      <label>
        Status:
        <select
          [ngModel]="filter().status"
          (ngModelChange)="statusChange.emit($event)"
        >
          <option value="All">All</option>
          <option value="Active">Active</option>
          <option value="Inactive">Inactive</option>
        </select>
      </label>
    </div>
  `,
  styles: [`
    .filter-panel {
      display: flex;
      gap: 16px;
      align-items: end;
      margin-bottom: 16px;
      padding: 12px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }

    label {
      display: flex;
      flex-direction: column;
      gap: 6px;
      font-weight: 600;
    }

    input, select {
      min-width: 220px;
      padding: 8px;
    }
  `]
})
export class CustomerFilterComponent {
  filter = input.required<CustomerFilter>();

  searchChange = output<string>();
  statusChange = output<CustomerFilter['status']>();
}
5) customer-table.component.ts

Another presentational component.

It gets customers and loading state as inputs, and emits actions upward.

import { Component, input, output } from '@angular/core';
import { Customer } from '../models/customer.models';

@Component({
  selector: 'app-customer-table',
  standalone: true,
  template: `
    @if (loading()) {
      <p>Loading customers...</p>
    } @else {
      <table class="customers-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>City</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          @for (customer of customers(); track customer.id) {
            <tr (click)="customerSelected.emit(customer.id)">
              <td>{{ customer.name }}</td>
              <td>{{ customer.city }}</td>
              <td>{{ customer.status }}</td>
            </tr>
          } @empty {
            <tr>
              <td colspan="3">No customers found.</td>
            </tr>
          }
        </tbody>
      </table>

      <button type="button" (click)="refreshClicked.emit()">
        Refresh
      </button>
    }
  `,
  styles: [`
    .customers-table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 12px;
    }

    .customers-table th,
    .customers-table td {
      border: 1px solid #ddd;
      padding: 10px;
      text-align: left;
    }

    .customers-table tbody tr {
      cursor: pointer;
    }

    .customers-table tbody tr:hover {
      background: #f7f7f7;
    }

    button {
      padding: 8px 14px;
    }
  `]
})
export class CustomerTableComponent {
  customers = input.required<Customer[]>();
  loading = input(false);

  refreshClicked = output<void>();
  customerSelected = output<number>();
}
6) customer-list-page.component.ts

This is the smart component.

It:

injects the store
loads data
wires child components together
handles navigation/user actions
import { Component, OnInit, inject } from '@angular/core';
import { Router } from '@angular/router';
import { CustomerFilterComponent } from '../components/customer-filter.component';
import { CustomerTableComponent } from '../components/customer-table.component';
import { CustomersStore } from '../state/customers.store';

@Component({
  selector: 'app-customer-list-page',
  standalone: true,
  imports: [CustomerFilterComponent, CustomerTableComponent],
  providers: [CustomersStore],
  template: `
    <section class="page">
      <h1>Customers</h1>

      @if (store.error(); as error) {
        <div class="error-banner">
          {{ error }}
        </div>
      }

      <app-customer-filter
        [filter]="store.filter()"
        (searchChange)="store.setSearch($event)"
        (statusChange)="store.setStatus($event)"
      />

      <app-customer-table
        [customers]="store.filteredCustomers()"
        [loading]="store.loading()"
        (refreshClicked)="reload()"
        (customerSelected)="openCustomer($event)"
      />
    </section>
  `,
  styles: [`
    .page {
      padding: 24px;
    }

    .error-banner {
      margin-bottom: 16px;
      padding: 12px;
      border: 1px solid #d9534f;
      background: #fbeaea;
      color: #a94442;
      border-radius: 6px;
    }
  `]
})
export class CustomerListPageComponent implements OnInit {
  readonly store = inject(CustomersStore);
  private readonly router = inject(Router);

  ngOnInit(): void {
    void this.store.load();
  }

  reload(): void {
    void this.store.load();
  }

  openCustomer(customerId: number): void {
    // In a real app:
    // void this.router.navigate(['/customers', customerId]);

    console.log('Navigate to customer details:', customerId);
  }
}
7) routes.ts

This is the feature route entry.

import { Routes } from '@angular/router';

export const CUSTOMERS_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/customer-list-page.component').then(
        m => m.CustomerListPageComponent
      )
  }
];
8) Example app.routes.ts

To lazy-load the feature:

import { Routes } from '@angular/router';

export const appRoutes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'customers'
  },
  {
    path: 'customers',
    loadChildren: () =>
      import('./features/customers/routes').then(m => m.CUSTOMERS_ROUTES)
  }
];
Why this demonstrates both patterns
Feature-based architecture

Everything lives under:

features/customers/

That means the entire customer feature is easy to find, change, test, and lazy-load.

Smart vs presentational
Smart

customer-list-page.component.ts

injects CustomersStore
loads data
handles navigation
coordinates child components
Presentational

customer-filter.component.ts

receives filter via input
emits search/status changes

customer-table.component.ts

receives data and loading state
emits refresh/select actions

Neither child knows anything about:

routing
services
stores
HTTP

That is the key split.

Why this scales well

As the feature grows, you can add:

customers/
  pages/
    customer-list-page.component.ts
    customer-detail-page.component.ts
    customer-edit-page.component.ts
  components/
    customer-filter.component.ts
    customer-table.component.ts
    customer-summary-card.component.ts
    customer-form.component.ts
  data-access/
    customers-api.service.ts
    customer-command.service.ts
  state/
    customers.store.ts
  models/
    customer.models.ts

The pattern still holds.

A few practical notes
1. Provider scope

Notice this line in the page:

providers: [CustomersStore]

That means the store is scoped to this feature page subtree, which is often a very clean default.

2. Presentational components can still have small UI state

For example:

selected tab
expanded row
open/closed panel

That is fine.

3. Do not force this split everywhere

For very small features, one component may be enough.
Use the pattern where it improves clarity.

Rule of thumb
Put logic here:
API calls → data-access
feature state → state
orchestration and navigation → pages
rendering and user interaction → components

That is the structure in one sentence.

If you want the next step

I can turn this into either:

a full copy-paste Angular 21 starter scaffold with all files including main.ts, app.config.ts, and app.component.ts, or
a more enterprise version with:
detail page
edit form
HTTP client
loading/error abstraction
reusable shared UI layer