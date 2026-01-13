# 🤖 Master Prompts for AI Coding Assistants

> Production-ready prompts for Claude, Cursor, Lovable, and other AI coding assistants to build white label SaaS platforms.

## Table of Contents

- [How to Use These Prompts](#how-to-use-these-prompts)
- [Full Stack Application Prompt](#full-stack-application-prompt)
- [Component-Specific Prompts](#component-specific-prompts)
- [Feature-Specific Prompts](#feature-specific-prompts)
- [Best Practices](#best-practices)

---

## How to Use These Prompts

### For Claude/Cursor/Windsurf
1. Copy the entire prompt
2. Paste into your AI assistant
3. Follow up with specific requirements
4. Iterate on the generated code

### For Lovable/v0/Bolt
1. Use the "Quick Start" prompts
2. Provide additional context as needed
3. Review and customize generated code

### For Antigravity
1. Use structured prompts with clear sections
2. Specify tech stack preferences
3. Request specific file structures

---

## Full Stack Application Prompt

### 🚀 Complete White Label SaaS Platform

```
I need you to build a production-ready white label SaaS platform with the following specifications:

## TECH STACK
- Frontend: Next.js 14+ (App Router), TypeScript, Tailwind CSS, Shadcn UI
- Backend: Next.js API Routes, tRPC (optional)
- Database: PostgreSQL with Prisma ORM
- Authentication: NextAuth.js with Google OAuth
- Billing: Stripe (subscriptions + usage-based)
- File Storage: AWS S3 or Vercel Blob
- Caching: Redis (Upstash)
- Email: Resend or SendGrid
- Deployment: Vercel

## CORE FEATURES

### 1. Multi-Tenant Architecture
- Shared database with tenant_id column on all tables
- Automatic tenant isolation via Prisma middleware
- Row-level security
- Tenant context in all API calls

### 2. Authentication & Authorization
- Email/password and OAuth (Google, GitHub)
- Role-based access control (Owner, Admin, Member)
- Team invitations via email
- Session management
- API key generation for programmatic access

### 3. Tenant Management
- Tenant creation and onboarding
- Team member management
- Role assignment
- Tenant settings and preferences

### 4. White Label Features
- Custom branding (logo, colors, fonts)
- Custom domain support (CNAME configuration)
- White-labeled emails
- Customizable UI themes
- CSS injection for advanced customization

### 5. Billing & Subscriptions
- Multiple pricing tiers (Free, Pro, Enterprise)
- Stripe Checkout integration
- Subscription management (upgrade/downgrade)
- Usage-based billing for API calls
- Invoice generation
- Payment method management
- Billing portal

### 6. Admin Dashboard
- Tenant overview and analytics
- User management
- Billing and subscription status
- Usage metrics and charts
- Settings and configuration

### 7. API
- RESTful API with rate limiting
- API key authentication
- Tenant-scoped endpoints
- Comprehensive error handling
- API documentation (Swagger/OpenAPI)

### 8. Security
- CSRF protection
- XSS prevention
- SQL injection prevention (via Prisma)
- Rate limiting per tenant
- Data encryption at rest
- Secure password hashing (bcrypt)
- Environment variable management

## DATABASE SCHEMA

Create Prisma schema with these models:
- Tenant (id, name, slug, domain, customDomain, logo, primaryColor, plan, stripeCustomerId, stripeSubscriptionId, maxUsers, apiLimit, createdAt, updatedAt)
- User (id, tenantId, email, name, image, role, emailVerified, password, createdAt, updatedAt)
- Account (NextAuth accounts table)
- Session (NextAuth sessions table)
- ApiKey (id, tenantId, name, key, lastUsed, createdAt, expiresAt)
- UsageRecord (id, tenantId, metric, quantity, timestamp)
- Invitation (id, tenantId, email, role, token, expiresAt, acceptedAt)

## FILE STRUCTURE

```
/app
  /(auth)
    /signin
    /signup
    /verify-email
  /(dashboard)
    /dashboard
    /settings
    /team
    /billing
    /api-keys
  /api
    /auth/[...nextauth]
    /tenants
    /users
    /billing
    /usage
/components
  /ui (shadcn components)
  /dashboard
  /auth
  /billing
/lib
  /prisma.ts
  /auth.ts
  /billing.ts
  /storage.ts
  /email.ts
  /utils.ts
/prisma
  /schema.prisma
  /migrations
```

## SPECIFIC REQUIREMENTS

1. **Tenant Isolation**: Every database query must be scoped to the current tenant. Implement Prisma middleware to automatically inject tenantId.

2. **Custom Domains**: Support CNAME configuration for custom domains. Detect tenant from hostname in middleware.

3. **Billing**: Implement Stripe webhooks to handle subscription events (created, updated, canceled, payment_succeeded, payment_failed).

4. **Rate Limiting**: Implement per-tenant rate limiting based on their plan (Free: 100 req/hour, Pro: 1000 req/hour, Enterprise: unlimited).

5. **Email Templates**: Create branded email templates for:
   - Welcome email
   - Team invitation
   - Password reset
   - Billing notifications
   - Usage alerts

6. **Error Handling**: Comprehensive error handling with user-friendly messages and proper HTTP status codes.

7. **Loading States**: Implement loading skeletons and optimistic updates for better UX.

8. **Responsive Design**: Mobile-first responsive design using Tailwind CSS.

9. **Dark Mode**: Support light/dark mode with tenant-customizable themes.

10. **Analytics**: Track key metrics (MRR, active users, API usage, churn rate).

## DELIVERABLES

Please provide:
1. Complete file structure with all code
2. Prisma schema with migrations
3. Environment variables template (.env.example)
4. README with setup instructions
5. Deployment guide for Vercel
6. API documentation
7. Testing setup (Jest + React Testing Library)

## CODE QUALITY

- Use TypeScript strict mode
- Follow Next.js best practices
- Implement proper error boundaries
- Add loading and error states
- Use server components where possible
- Implement proper SEO (metadata)
- Add comments for complex logic
- Use Zod for validation
- Implement proper logging

Start with the core authentication and tenant management, then we'll iterate on additional features.
```

---

## Component-Specific Prompts

### 1. Authentication System

```
Create a complete authentication system for a multi-tenant SaaS application:

## Requirements:
- NextAuth.js with Prisma adapter
- Email/password authentication with email verification
- Google OAuth provider
- Automatic tenant creation on signup
- Tenant detection from email domain
- Role-based access control (Owner, Admin, Member)
- Session management with JWT
- Protected routes and API endpoints
- Password reset flow
- Email verification flow

## Files to create:
1. app/api/auth/[...nextauth]/route.ts - NextAuth configuration
2. lib/auth.ts - Auth utilities and session helpers
3. middleware.ts - Route protection middleware
4. app/(auth)/signin/page.tsx - Sign in page
5. app/(auth)/signup/page.tsx - Sign up page
6. app/(auth)/verify-email/page.tsx - Email verification
7. app/(auth)/reset-password/page.tsx - Password reset
8. components/auth/SignInForm.tsx - Sign in form component
9. components/auth/SignUpForm.tsx - Sign up form component

## Prisma Schema:
Include User, Account, Session, VerificationToken models compatible with NextAuth.

## Security:
- Hash passwords with bcrypt (12 rounds)
- Generate secure verification tokens
- Implement rate limiting on auth endpoints
- CSRF protection
- Secure session cookies (httpOnly, secure, sameSite)

Provide complete, production-ready code with proper error handling and TypeScript types.
```

### 2. Multi-Tenant Database Layer

```
Create a multi-tenant database layer with automatic tenant isolation:

## Requirements:
- Prisma ORM with PostgreSQL
- Shared database, shared schema pattern
- Automatic tenant_id injection on all queries
- Prisma middleware for tenant isolation
- Helper functions for tenant-scoped queries
- Migration scripts
- Seed data for development

## Implementation:
1. Prisma schema with tenant_id on all models
2. Prisma middleware to inject tenant context
3. Helper functions: getTenantPrisma(tenantId)
4. Row-level security policies (PostgreSQL)
5. Indexes for performance
6. Soft delete support

## Models needed:
- Tenant
- User
- Product (example entity)
- Order (example entity)
- ApiKey
- UsageRecord

## Code structure:
```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// Create tenant-scoped Prisma client
export function getTenantPrisma(tenantId: string) {
  // Implementation here
}

// Middleware for automatic tenant isolation
// Usage examples
```

Provide complete implementation with TypeScript types and comprehensive comments.
```

### 3. Stripe Billing Integration

```
Create a complete Stripe billing integration for a SaaS platform:

## Features:
- Multiple pricing tiers (Free, Pro, Enterprise)
- Subscription management (create, update, cancel)
- Usage-based billing for API calls
- Customer portal for self-service
- Webhook handling for events
- Invoice generation
- Payment method management
- Proration on plan changes
- Trial periods

## Files to create:
1. lib/billing.ts - Billing service class
2. app/api/billing/checkout/route.ts - Create checkout session
3. app/api/billing/portal/route.ts - Customer portal
4. app/api/billing/webhook/route.ts - Webhook handler
5. app/(dashboard)/billing/page.tsx - Billing dashboard
6. components/billing/PricingTable.tsx - Pricing display
7. components/billing/SubscriptionCard.tsx - Current subscription
8. components/billing/UsageChart.tsx - Usage visualization

## Stripe Products:
- Free: $0/month, 100 API calls/month, 1 user
- Pro: $29/month, 10,000 API calls/month, 5 users
- Enterprise: $99/month, unlimited API calls, unlimited users

## Webhook Events to handle:
- customer.subscription.created
- customer.subscription.updated
- customer.subscription.deleted
- invoice.payment_succeeded
- invoice.payment_failed
- customer.updated

## Implementation:
```typescript
// lib/billing.ts
export class BillingService {
  async createCheckoutSession(tenantId: string, priceId: string) {}
  async createCustomerPortalSession(tenantId: string) {}
  async recordUsage(tenantId: string, metric: string, quantity: number) {}
  async handleWebhook(event: Stripe.Event) {}
  async cancelSubscription(tenantId: string) {}
  async updateSubscription(tenantId: string, newPriceId: string) {}
}
```

Provide complete, production-ready code with error handling and TypeScript types.
```

### 4. Admin Dashboard

```
Create a comprehensive admin dashboard for tenant management:

## Pages:
1. Overview - Key metrics and charts
2. Team - User management and invitations
3. Settings - Tenant configuration and branding
4. Billing - Subscription and payment management
5. API Keys - Generate and manage API keys
6. Usage - API usage analytics

## Components:
- Sidebar navigation
- Header with tenant switcher
- Metric cards (MRR, active users, API calls)
- Charts (revenue, usage trends)
- Data tables with sorting and filtering
- Forms with validation
- Modal dialogs
- Toast notifications

## Features:
- Real-time data updates
- Responsive design
- Dark mode support
- Loading skeletons
- Error boundaries
- Optimistic updates

## Tech Stack:
- Next.js 14 App Router
- Shadcn UI components
- Recharts for visualizations
- React Hook Form + Zod validation
- TanStack Query for data fetching

## File Structure:
```
/app/(dashboard)
  /layout.tsx
  /dashboard/page.tsx
  /team/page.tsx
  /settings/page.tsx
  /billing/page.tsx
  /api-keys/page.tsx
  /usage/page.tsx
/components/dashboard
  /Sidebar.tsx
  /Header.tsx
  /MetricCard.tsx
  /RevenueChart.tsx
  /UsageChart.tsx
  /TeamTable.tsx
  /ApiKeyTable.tsx
```

Provide complete implementation with TypeScript, proper data fetching, and error handling.
```

### 5. White Label Customization

```
Create a white label customization system:

## Features:
1. Logo upload and management
2. Color scheme customization (primary, secondary, accent)
3. Custom domain configuration (CNAME)
4. Custom CSS injection
5. Email template customization
6. Theme preview
7. Branding presets

## Implementation:
1. Settings page with customization form
2. File upload to S3/Vercel Blob
3. Color picker component
4. CSS variable injection
5. Domain verification system
6. Email template editor
7. Live preview component

## Files:
```
/app/(dashboard)/settings/branding/page.tsx
/components/settings/LogoUpload.tsx
/components/settings/ColorPicker.tsx
/components/settings/DomainConfig.tsx
/components/settings/CssEditor.tsx
/components/settings/ThemePreview.tsx
/lib/branding.ts
/lib/storage.ts
```

## Database Schema:
```prisma
model Tenant {
  // ... existing fields
  logo              String?
  primaryColor      String?  @default("#3b82f6")
  secondaryColor    String?  @default("#8b5cf6")
  accentColor       String?  @default("#10b981")
  customCss         String?  @db.Text
  customDomain      String?  @unique
  domainVerified    Boolean  @default(false)
  emailFromName     String?
  emailFromAddress  String?
}
```

## Custom Domain Flow:
1. User enters custom domain
2. System generates CNAME record instructions
3. User adds CNAME to their DNS
4. System verifies domain
5. SSL certificate provisioned
6. Domain activated

Provide complete implementation with file upload, validation, and preview functionality.
```

---

## Feature-Specific Prompts

### API Rate Limiting

```
Implement per-tenant API rate limiting:

## Requirements:
- Redis-based rate limiting
- Different limits per plan (Free: 100/hour, Pro: 1000/hour, Enterprise: unlimited)
- Rate limit headers in responses
- Graceful error messages
- Admin override capability
- Usage tracking

## Implementation:
```typescript
// lib/rate-limit.ts
import { Redis } from '@upstash/redis';

export async function rateLimit(
  tenantId: string,
  limit: number,
  window: number
): Promise<{ success: boolean; remaining: number; reset: number }> {
  // Implementation
}

// middleware.ts
export async function middleware(request: NextRequest) {
  const tenant = await getTenantFromRequest(request);
  const result = await rateLimit(tenant.id, tenant.apiLimit, 3600);
  
  if (!result.success) {
    return new Response('Rate limit exceeded', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': tenant.apiLimit.toString(),
        'X-RateLimit-Remaining': '0',
        'X-RateLimit-Reset': result.reset.toString(),
      },
    });
  }
  
  // Continue...
}
```

Provide complete implementation with Redis integration and proper error handling.
```

### Team Invitations

```
Create a team invitation system:

## Features:
- Send email invitations
- Invitation tokens with expiration
- Accept/decline invitations
- Role assignment
- Pending invitations list
- Resend invitations
- Cancel invitations

## Flow:
1. Owner/Admin invites user by email
2. System generates secure token
3. Email sent with invitation link
4. User clicks link and accepts
5. User added to tenant with specified role

## Files:
```
/app/api/invitations/route.ts - Create invitation
/app/api/invitations/[token]/route.ts - Accept invitation
/app/(dashboard)/team/page.tsx - Team management
/components/team/InviteUserDialog.tsx
/components/team/PendingInvitations.tsx
/lib/invitations.ts
```

## Database:
```prisma
model Invitation {
  id         String   @id @default(cuid())
  tenantId   String
  email      String
  role       String
  token      String   @unique
  expiresAt  DateTime
  acceptedAt DateTime?
  createdBy  String
  createdAt  DateTime @default(now())
  
  tenant     Tenant   @relation(fields: [tenantId], references: [id])
  inviter    User     @relation(fields: [createdBy], references: [id])
}
```

Provide complete implementation with email templates and security best practices.
```

### Usage Analytics

```
Create a usage analytics system:

## Features:
- Track API calls per tenant
- Track storage usage
- Track feature usage
- Real-time dashboards
- Historical trends
- Usage alerts
- Export reports

## Metrics to track:
- API calls (total, by endpoint)
- Storage (files, size)
- Active users
- Feature usage
- Error rates
- Response times

## Implementation:
```typescript
// lib/analytics.ts
export class AnalyticsService {
  async trackApiCall(tenantId: string, endpoint: string) {}
  async trackStorageUsage(tenantId: string, bytes: number) {}
  async getUsageStats(tenantId: string, period: string) {}
  async checkUsageLimits(tenantId: string) {}
  async sendUsageAlert(tenantId: string, metric: string) {}
}

// middleware.ts - Track all API calls
export async function middleware(request: NextRequest) {
  const start = Date.now();
  const response = await next();
  const duration = Date.now() - start;
  
  await analytics.trackApiCall(tenantId, {
    endpoint: request.url,
    method: request.method,
    status: response.status,
    duration,
  });
  
  return response;
}
```

## Visualization:
- Line charts for trends
- Bar charts for comparisons
- Pie charts for distribution
- Real-time counters

Provide complete implementation with data aggregation and visualization components.
```

---

## Best Practices

### Prompt Engineering Tips

1. **Be Specific**: Include exact tech stack, file structure, and requirements
2. **Provide Context**: Explain the use case and constraints
3. **Request Complete Code**: Ask for production-ready, not just examples
4. **Include Error Handling**: Explicitly request error handling and edge cases
5. **Specify Types**: Request TypeScript with proper types
6. **Ask for Tests**: Include testing requirements
7. **Iterate**: Start with core features, then add complexity

### Follow-Up Prompts

After initial generation:

```
"Add comprehensive error handling with user-friendly messages"
"Implement loading states and optimistic updates"
"Add unit tests using Jest and React Testing Library"
"Optimize for performance (memoization, lazy loading)"
"Add accessibility features (ARIA labels, keyboard navigation)"
"Implement proper logging for debugging"
"Add input validation with Zod schemas"
"Create API documentation with examples"
```

### Code Review Prompts

```
"Review this code for security vulnerabilities"
"Optimize this code for performance"
"Refactor this code to follow best practices"
"Add TypeScript types to this code"
"Improve error handling in this code"
"Make this code more maintainable"
```

---

## Quick Start Prompts

### For Lovable.dev

```
Create a white label SaaS starter with:
- Next.js 14 + TypeScript + Tailwind
- Multi-tenant architecture
- Stripe billing
- Team management
- Custom branding
- Admin dashboard

Use Shadcn UI components and make it production-ready.
```

### For v0.dev

```
Design a SaaS dashboard with:
- Sidebar navigation
- Metric cards showing MRR, users, API calls
- Revenue chart (line chart)
- Recent activity table
- Settings panel

Use Shadcn UI, Tailwind CSS, and Recharts. Make it responsive and include dark mode.
```

### For Bolt.new

```
Build a complete authentication flow:
- Sign in/up pages
- Email verification
- Password reset
- OAuth (Google)
- Protected routes

Use Next.js 14, NextAuth.js, and Shadcn UI. Include form validation with Zod.
```

---

## Advanced Prompts

### Microservices Architecture

```
Design a microservices architecture for a white label SaaS platform:

## Services:
1. Auth Service (authentication, authorization)
2. Tenant Service (tenant management, settings)
3. Billing Service (Stripe integration, invoicing)
4. API Gateway (routing, rate limiting)
5. Analytics Service (usage tracking, reporting)
6. Notification Service (emails, webhooks)

## Tech Stack:
- Node.js + Express for services
- PostgreSQL for data
- Redis for caching
- RabbitMQ for messaging
- Docker + Kubernetes for deployment

## Communication:
- REST APIs between services
- Message queue for async operations
- gRPC for internal service communication

Provide:
1. Service architecture diagram
2. API contracts
3. Docker compose setup
4. Kubernetes manifests
5. Service implementation examples
```

### AI Integration

```
Add AI features to the white label SaaS platform:

## Features:
1. AI-powered content generation
2. Chatbot for customer support
3. Predictive analytics
4. Automated email responses
5. Smart recommendations

## Implementation:
- OpenAI API integration
- Streaming responses
- Context management
- Token usage tracking
- Cost optimization

## Files:
```
/lib/ai.ts - AI service
/app/api/ai/chat/route.ts - Chat endpoint
/app/api/ai/generate/route.ts - Content generation
/components/ai/ChatWidget.tsx
/components/ai/ContentGenerator.tsx
```

Provide complete implementation with streaming, error handling, and cost tracking.
```

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Author:** Babu B  
**Contact:** babukiran.b@gmail.com
