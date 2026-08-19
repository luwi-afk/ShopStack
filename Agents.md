# ShopStack — Master Agent Instructions

## 1. Project Mission

ShopStack is a full-stack e-commerce application with two user roles:

- `admin`
- `customer`

There is no seller role.

The application must support product browsing, carts, authentication, checkout, order tracking, Stripe payments, administration, analytics, order inquiries, and audit logging.

The project is being developed as a coordinated multi-agent system using Orca and OpenCode.

## 2. Authoritative Technology Architecture

### Application

- Next.js
- TypeScript
- Responsive web application
- Server-side rendering and server components where appropriate
- Server Actions and Route Handlers for server-side operations

### Database

- PostgreSQL
- PostgreSQL runs in Docker for local development
- Prisma is the ORM
- Prisma owns schema, migrations, generated client, and typed database access

### Authentication

- Auth.js / NextAuth
- Credentials provider
- Passwords hashed using bcrypt
- Passwords must never be stored, logged, or transmitted in plaintext
- User role is carried in the authenticated session
- Default registration role is `customer`
- Admin accounts are provisioned through trusted seed/database procedures

### Payments

- Stripe Checkout
- Stripe sandbox/development environment
- Payment confirmation must happen through verified Stripe webhooks
- Never trust a client-side redirect as proof of payment

### Storage

- Local disk during development
- Production storage provider will be decided later
- Do not hard-code a production storage vendor unless explicitly requested

### Visualization

- Chart.js for analytics

## 3. Repository Architecture

The project must use the following architectural organization:

```text
ShopStack/
├── apps/
│   └── web/
│       ├── src/
│       │   ├── app/
│       │   ├── components/
│       │   ├── services/
│       │   ├── utils/
│       │   ├── lib/
│       │   └── types/
│       └── public/
│           └── branding/
│
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
│
├── docker-compose.yml
├── .env
├── .env.example
├── .gitignore
└── BRANDING.md

```

### `app/`

Contains Next.js route files, layouts, pages, server actions, and route handlers.

Route files should remain thin and delegate business logic to services.

### `components/`

Presentational UI only.

Components must not:

- import Prisma
- directly access the database
- contain domain business logic
- contain secrets
- perform authorization as a security boundary

### `services/`

Domain business logic and data access.

Services are the only application layer permitted to access Prisma.

Expected domains include:

```text
product.service.ts
order.service.ts
auth.service.ts
audit.service.ts
message.service.ts
cart.service.ts

```

### `utils/`

Pure functions only.

Utilities must not:

- access Prisma
- perform I/O
- access secrets
- mutate persistent state
- depend on request/session state

Expected utilities include:

```text
validators.ts
order-state-machine.ts
formatters.ts

```

### `lib/`

Infrastructure clients and singletons.

Examples:

```text
prisma.ts
auth.ts
stripe.ts

```

## 4. Database Models

The Prisma schema must represent:

### User

- id
- email
- password_hash
- full_name
- role
- address
- created_at

Roles:

```text
admin
customer

```

### Product

- id
- name
- description
- price
- stock
- category
- sku
- image_url
- created_at

### Order

- id
- user_id
- status
- payment_status
- total_amount
- created_at
- updated_at

### OrderItem

- id
- order_id
- product_id
- quantity
- price_at_purchase

### AuditLog

- id
- admin_id
- action
- target_table
- target_id
- metadata
- created_at

### OrderMessage

- id
- order_id
- sender_id
- message
- created_at

All relationships and referential behavior must be explicitly defined in Prisma.

## 5. Authorization and Security

Security is enforced on the server.

Never trust the client.

Every protected server action and route handler must:

1. Validate the authenticated session.
2. Validate the user's role where required.
3. Validate and sanitize incoming data.
4. Perform the requested operation only after authorization succeeds.

Hiding a button in the frontend is not authorization.

### Admin security

Every admin route and admin mutation must perform a server-side admin check.

### Customer security

Customers may access only their own protected resources.

### Password security

Use bcrypt.

Never:

- store plaintext passwords
- log passwords
- expose password hashes to clients
- include passwords in error messages

### Database security

Use Prisma parameterized queries.

Never construct unsafe raw SQL from untrusted user input.

### Stripe security

Verify Stripe webhook signatures.

Never mark an order as paid based solely on a browser redirect.

## 6. Order State Machine

Valid transitions are:

```text
pending → paid
pending → cancelled
paid → shipped
paid → cancelled
shipped → delivered

```

Rules:

- `pending → paid` can only occur from the verified Stripe webhook flow.
- `pending → cancelled` is an admin action.
- `paid → shipped` is an admin action.
- `paid → cancelled` is an admin action and must integrate with the appropriate refund workflow.
- `shipped → delivered` is an admin action.

Every transition except the Stripe-driven `pending → paid` transition must create an `AuditLog`.

Invalid transitions must be rejected.

The state machine must be implemented as a pure utility and reused by the service layer.

## 7. Stripe Flow

Checkout is initiated server-side.

The flow is:

```text
Customer
→ create pending order
→ create Stripe Checkout session
→ redirect to Stripe
→ Stripe processes payment
→ Stripe webhook
→ verify webhook signature
→ transition pending → paid
→ decrement stock in a transaction
→ record resulting state

```

Client-side success redirects must never be treated as authoritative payment confirmation.

## 8. Customer Features

The customer experience must include:

- public product catalog
- product search
- filtering
- categories
- product detail pages
- cart
- authentication
- registration
- shipping address management
- Stripe checkout
- order history
- order status tracking
- order-specific support messaging

Public catalog browsing does not require authentication.

Protected customer operations require an authenticated session.

## 9. Admin Features

The admin dashboard must support:

- product creation
- product editing
- product deletion
- image management
- stock management
- order monitoring
- payment status monitoring
- analytics
- revenue trends
- top products
- order inquiries
- admin profile
- audit logging

Admin operations must always be authorized server-side.

## 10. Branding and Existing Assets

Existing ShopStack logos, icons, illustrations, and brand assets supplied by the project owner are authoritative.

Agents must inspect existing assets before creating branded UI.

### Required behavior

- Reuse existing official logos.
- Do not redesign the official logo.
- Do not generate a replacement logo when an official asset exists.
- Do not substitute generic placeholder branding when an official asset exists.
- Preserve the original appearance and proportions.
- Use the correct provided variant for light and dark backgrounds.
- Reuse existing icons and brand imagery where appropriate.
- If a required brand asset is missing, report the missing asset rather than inventing an official replacement.

Brand assets should normally live under:

```text
apps/web/public/branding/

```

The Frontend Agent is responsible for integrating the provided assets into the UI.

The Reviewer must verify that official branding has not been replaced or altered unnecessarily.

## 11. Agent Separation of Responsibilities

### Planner — Gemini 3.5 Flash Lite

The Planner is responsible for:

- understanding requirements
- architecture
- dependency analysis
- task decomposition
- identifying risks
- identifying database/backend/frontend boundaries
- creating implementation plans
- coordinating implementation waves

The Planner should not implement application features unless explicitly instructed.

### Frontend — Cohere North Mini Code (free)

The Frontend Agent is responsible for:

- Next.js pages
- layouts
- UI components
- forms
- responsive behavior
- client-facing services
- frontend utilities
- accessibility
- integrating official ShopStack branding

The Frontend Agent must not bypass backend authorization or access Prisma directly from presentation components.

### Backend — Cohere North Mini Code (free)

The Backend Agent is responsible for:

- server-side services
- Prisma
- database access
- migrations
- authentication
- authorization
- Stripe integration
- webhooks
- business rules
- state transitions
- audit logging
- validation

### Reviewer — Gemini 3.5 Flash Lite

The Reviewer must inspect completed work for:

- correctness
- architecture compliance
- security
- authorization
- data integrity
- state-machine correctness
- Stripe webhook correctness
- test coverage
- branding compliance
- accidental scope expansion
- violations of component/service/utils boundaries

The Reviewer must not silently rewrite unrelated work.

## 12. Git and Worktree Rules

`master` is protected.

Agents work in isolated Orca worktrees:

```text
planner
frontend
backend
reviewer

```

Agents must not directly modify the `master` worktree.

Agents must not push to `master`.

Implementation should proceed through:

```text
agent worktree
→ validation
→ reviewer
→ approved changes
→ controlled merge

```

Each agent should keep its branch focused on its assigned task.

Avoid unrelated modifications.

## 13. Orchestration Workflow

The intended development workflow is:

```text
Requirements
    ↓
Planner
    ↓
Task graph
    ↓
Backend / Frontend
    ↓
Implementation
    ↓
Reviewer
    ↓
Pass / Fix
    ↓
Merge

```

The Planner should assign work using Orca orchestration tasks.

Frontend and Backend tasks should be independent whenever possible.

Dependencies must be explicit.

Do not dispatch work that depends on unfinished foundation work.

## 14. Implementation Phases

### Phase 1 — Foundation

- Next.js + TypeScript foundation
- apps/web structure
- Docker PostgreSQL
- Prisma
- Prisma schema
- Prisma seed infrastructure
- environment configuration
- shared types

### Phase 2 — Infrastructure and Utilities

- Prisma singleton
- Stripe client
- validators
- order state machine
- formatting utilities

### Phase 3 — Authentication and Security

- Auth.js Credentials provider
- bcrypt
- session handling
- role handling
- server-side guards
- authentication pages

### Phase 4 — Catalog and Cart

- product service
- catalog
- product detail
- search/filter/category behavior
- cart

### Phase 5 — Orders and Stripe

- order service
- order creation
- checkout
- Stripe webhook
- stock decrement transaction
- payment state transitions

### Phase 6 — Admin

- admin shell
- product management
- product CRUD
- stock management
- audit integration

### Phase 7 — Fulfillment and Analytics

- admin order management
- order state transitions
- audit logs
- analytics
- Chart.js

### Phase 8 — Inquiries and Profiles

- order messages
- customer orders/profile
- admin profile
- order-specific support threads

### Phase 9 — Verification

- security review
- authorization review
- database integrity checks
- payment-flow verification
- state-machine tests
- frontend validation
- backend tests
- end-to-end manual flows

## 15. Agent Working Rules

Every agent must:

- read the relevant repository files before making assumptions
- stay within its assigned worktree
- follow the architecture boundaries
- avoid unnecessary dependencies
- avoid unrelated refactors
- preserve existing branding
- protect secrets
- validate its work before reporting completion
- explain failures honestly
- report blockers instead of inventing workarounds that violate requirements

No agent should claim success without actually validating the result.

## 16. Current Development Objective

The immediate objective is to establish the ShopStack foundation.

The current order is:

```text
Docker PostgreSQL
      ↓
Prisma + schema
      ↓
Next.js foundation
      ↓
Authentication
      ↓
Catalog + cart
      ↓
Orders + Stripe
      ↓
Admin
      ↓
Inquiries + profiles
      ↓
Testing + security review

```

The Planner controls task decomposition and dependency ordering.

The Backend Agent establishes the shared server/database foundation.

The Frontend Agent builds the UI and presentation architecture on top of stable backend contracts.

The Reviewer validates every meaningful implementation wave before it is merged.

## 17. Non-Negotiable Rules

1. Never expose secrets.
2. Never trust the client for authorization.
3. Never treat a Stripe redirect as payment confirmation.
4. Never allow UI components to access Prisma directly.
5. Never put business logic into pure utilities.
6. Never modify `master` directly.
7. Never replace official ShopStack branding when supplied assets exist.
8. Never invent requirements that are not in the specification without clearly identifying them as proposals.
9. Never silently weaken security to make a feature easier to implement.
10. Never claim an implementation is complete without validation.

