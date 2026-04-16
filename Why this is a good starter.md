Why this is a good starter

This scaffold gives you a clean baseline:

Feature-based architecture

Everything for customers is inside:

features/customers/
Smart component

customer-list-page.component.ts

coordinates the feature
talks to the store
handles actions/navigation
Presentational components

customer-filter.component.ts
customer-table.component.ts

They:

receive inputs
emit outputs
do not know about API/store/routing
Data-access layer

customers-api.service.ts

State layer

customers.store.ts

That separation is the core idea you wanted to learn.