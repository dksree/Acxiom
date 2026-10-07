# AcxiomCRM

AcxiomCRM is a role-aware ASP.NET Core MVC CRM application built for the supplied academic assignment. It uses ASP.NET Core Identity for authentication and role management, Entity Framework Core with SQLite, server-side business validation, audit logging, REST APIs, dashboards, reports, and sample CRM data.

## Technology stack

- .NET 8 / ASP.NET Core MVC
- ASP.NET Core Identity + Identity roles
- Entity Framework Core 8
- SQLite
- Bootstrap 5
- Chart.js
- Razor Views
- REST API controllers with DTOs

## Main features

- Identity authentication, password policy and lockout
- Admin / Manager / SalesExecutive roles
- Role-aware CRM data visibility
- Customers, Leads, Opportunities, Follow-Ups and Activities
- Lead conversion to Customer and optional Opportunity
- Opportunity weighted pipeline calculations
- Client + server + business validation
- Search, filters, sorting and pagination on CRM lists
- Dashboard KPI cards, charts and date ranges
- Audit logging for authentication, security and CRM changes
- Admin-only user/role/audit administration
- REST APIs for Customers, Leads and Opportunities
- Reports with filters and pagination
- Customer CSV export
- Responsive Bootstrap UI

## Architecture

Controllers -> Services -> EF Core DbContext -> SQLite

Views are separated from business logic. DTOs are used by the REST API. ASP.NET Core Identity owns authentication and security-related user data; there is no custom authentication user table.

## Project structure

```text
AcxiomCRM/
├── AcxiomCRM.sln
├── README.md
└── AcxiomCRM/
    ├── Controllers/
    │   ├── AccountController.cs
    │   ├── DashboardController.cs
    │   ├── CustomersController.cs
    │   ├── LeadsController.cs
    │   ├── OpportunitiesController.cs
    │   ├── FollowUpsController.cs
    │   ├── ActivitiesController.cs
    │   ├── UsersController.cs
    │   ├── RolesController.cs
    │   ├── AuditLogsController.cs
    │   ├── ReportsController.cs
    │   └── Api/
    ├── Data/
    ├── DTOs/
    ├── Models/
    ├── Services/
    ├── ViewModels/
    ├── Views/
    ├── Areas/Identity/
    ├── Migrations/
    ├── wwwroot/
    ├── Program.cs
    └── AcxiomCRM.csproj
```

## Database setup

The project uses SQLite and stores the development database as `acxiomcrm.db`.

Restore packages:

```bash
dotnet restore
```

Apply the included initial EF Core migration:

```bash
dotnet ef database update
```

The application also attempts `Database.MigrateAsync()` during development and falls back to `EnsureCreatedAsync()` if no migration can be applied. For the assignment, prefer the normal migration command above so the database is explicitly managed through EF Core migrations.

If `dotnet ef` is not installed:

```bash
dotnet tool install --global dotnet-ef --version 8.*
```

## Run

```bash
dotnet restore
dotnet ef database update
dotnet run
```

Then open the URL printed by ASP.NET Core, normally:

- http://localhost:5100
- https://localhost:7100

## Seeded test accounts

The development seed creates these accounts:

| Role | Email | Password |
|---|---|---|
| Admin | admin@acxiom.local | Admin@123 |
| Manager | manager@acxiom.local | Manager@123 |
| SalesExecutive | sales1@acxiom.local | Sales@123 |
| SalesExecutive | sales2@acxiom.local | Sales@123 |

These are development/demo credentials only. Do not use them in production.

## Role permissions

### Admin

- Full CRM visibility
- User administration
- Role administration
- Audit log access
- Reports

### Manager

- Team-level CRM visibility
- Customer, lead, opportunity, follow-up and activity workflows
- Team reports/dashboard
- No unrestricted security administration

### SalesExecutive

- Assigned/authorized CRM records
- Customer, lead, opportunity, follow-up and activity workflows
- Own/team-authorized dashboard/report data
- No user/role/audit administration

Hiding navigation links is not the security boundary; controller authorization and service-level visibility checks are also applied.

## API endpoints

All API endpoints require an authenticated Identity user and use DTOs.

```text
GET    /api/customers
GET    /api/customers/{id}
POST   /api/customers
PUT    /api/customers/{id}
DELETE /api/customers/{id}

GET    /api/leads
POST   /api/leads

GET    /api/opportunities
POST   /api/opportunities
```

Typical response codes include 200, 201, 400, 401, 403, 404 and 409.

## Validation/business rules

- Customer email and phone are required, valid and unique.
- Indian phone validation uses a 10-digit mobile pattern beginning with 6-9.
- Opportunity amount must be greater than zero.
- Probability must be 0-100.
- Active opportunity close date cannot be in the past.
- Weighted pipeline = Amount × Probability / 100.
- Planned follow-up date cannot be earlier than today.
- Lead statuses are New, Contacted, Qualified, Unqualified, Converted and Lost.
- Invalid lead status transitions are rejected.
- Only Qualified leads can be converted.

## Security

- ASP.NET Core Identity password hashing
- Password policy
- Account lockout
- Role-based authorization
- Server-side validation
- Anti-forgery protection on MVC POST actions
- Secure HttpOnly authentication cookie
- EF Core LINQ/parameterized database access
- No password hashes, tokens or secrets in audit logs
- Friendly error handling without database/stack-trace details
- No production database credentials hard-coded

## Audit logging

Audit entries cover login success/failure, lockout, logout, user/role security changes, customer changes, lead changes/conversion, opportunity changes, follow-up changes and activity changes.

Audit logs are append-oriented. There is no normal edit/delete UI for audit records.

## Dashboard

The dashboard contains:

- Total Customers
- Total Leads
- Open Leads
- Total Opportunities
- Open Opportunities
- Won Opportunities
- Lost Opportunities
- Total Pipeline Value
- Lead Status chart
- Opportunity Pipeline chart
- Monthly sales/outcome chart
- Today / This Week / This Month / Custom date filtering

## Reports

Available report areas include Customer, Lead/Conversion, Follow-Up, Opportunity/Pipeline, User Activity and Audit. Reports support practical filtering and pagination; the customer report also supports CSV export.

## Testing checklist

After setup, verify:

1. Anonymous access to protected CRM pages redirects to Login.
2. Valid login reaches Dashboard.
3. Invalid login is rejected and audited.
4. Repeated failed logins trigger Identity lockout.
5. Invalid email/phone is rejected by browser validation and server validation.
6. Duplicate customer email/phone is rejected.
7. Opportunity amount 0 is rejected.
8. Opportunity probability above 100 is rejected.
9. Past active opportunity close date is rejected.
10. Planned follow-up before today is rejected with the required message.
11. SalesExecutive sees only authorized records.
12. Manager sees team records.
13. Admin sees all records and security administration.
14. Customer/opportunity/lead/follow-up/activity changes create audit records.
15. `/api/customers` returns authorized JSON.
16. Dashboard KPI cards and Chart.js charts render.
17. Reports filter and paginate.
18. Passwords/hashes/tokens are not exposed in pages or audit data.

## EF Core migration note

The supplied build environment did not contain the .NET SDK, so the initial migration was authored directly as an EF Core `Migration` class rather than generated with `dotnet ef migrations add`. On a normal .NET 8 development machine, `dotnet ef database update` applies it. If you change the model afterward, create a new normal EF migration and commit the generated migration plus model snapshot.

## Assignment mapping

| Assignment area | Implementation |
|---|---|
| Identity authentication | `ApplicationUser`, `AccountController`, Identity configuration |
| Customer management | `CustomersController`, `CustomerService`, views/API |
| Lead management | `LeadsController`, `LeadService`, conversion workflow |
| Opportunity management | `OpportunitiesController`, `OpportunityService`, weighted pipeline |
| Follow-ups | `FollowUpsController`, `FollowUpService` |
| Activities | `ActivitiesController`, `ActivityService` |
| Dashboard | `DashboardController`, `DashboardService`, Chart.js |
| Search/filter/pagination | CRM list controllers + report controllers |
| User/role management | `UsersController`, `RolesController` |
| Audit | `AuditService`, `AuditLogsController` |
| Validation/security | DataAnnotations + service rules + Identity + anti-forgery |
| REST API | `Controllers/Api/*`, DTOs |
| Reports | `ReportsController`, report views/services |
| Database | EF Core + SQLite + migration |
| Seed data | `DbInitializer` |
