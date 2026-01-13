# 🌟 Open Source White Label SaaS Platforms

> Comprehensive analysis of open-source SaaS boilerplates and starter kits available on GitHub.

## Table of Contents

- [Top Recommendations](#top-recommendations)
- [Detailed Analysis](#detailed-analysis)
- [Comparison Matrix](#comparison-matrix)
- [How to Choose](#how-to-choose)
- [Fork and Customize Guide](#fork-and-customize-guide)

---

## Top Recommendations

### 🥇 #1: ixartz/SaaS-Boilerplate

**GitHub:** https://github.com/ixartz/SaaS-Boilerplate  
**Stars:** ⭐ 6,690+  
**License:** MIT

**Why Choose This:**
- ✅ Most comprehensive Next.js 14 boilerplate
- ✅ Built-in multi-tenancy with teams
- ✅ Shadcn UI components (modern, accessible)
- ✅ TypeScript strict mode
- ✅ Role-based permissions
- ✅ Internationalization (i18n)
- ✅ SEO optimized
- ✅ Active maintenance

**Tech Stack:**
- Frontend: Next.js 14 (App Router), React, TypeScript
- Styling: Tailwind CSS, Shadcn UI
- Auth: Clerk (can be swapped with NextAuth)
- Database: PostgreSQL with Prisma
- Testing: Jest, React Testing Library
- Monitoring: Sentry

**Features:**
```
✅ Multi-tenant architecture
✅ Team management
✅ Role-based access control (Owner, Admin, Member)
✅ Billing integration ready
✅ Email templates
✅ Landing page
✅ Dashboard
✅ Settings pages
✅ API routes
✅ Logging system
✅ Error tracking
✅ i18n support
```

**Quick Start:**
```bash
git clone https://github.com/ixartz/SaaS-Boilerplate.git my-saas
cd my-saas
npm install
cp .env.example .env.local
npm run dev
```

**Customization Effort:** 🟢 Low (2-3 weeks to production)

---

### 🥈 #2: wasp-lang/open-saas

**GitHub:** https://github.com/wasp-lang/open-saas  
**Stars:** ⭐ 13,200+  
**License:** MIT

**Why Choose This:**
- ✅ Highest star count
- ✅ Wasp framework (full-stack with superpowers)
- ✅ Built-in auth, payments, email
- ✅ Type-safe APIs
- ✅ Community-driven
- ✅ Excellent documentation

**Tech Stack:**
- Framework: Wasp (React + Node.js)
- Frontend: React, TypeScript
- Backend: Node.js, Express
- Database: PostgreSQL with Prisma
- Auth: Built-in (email, Google, GitHub)
- Payments: Stripe
- Email: SendGrid

**Features:**
```
✅ Full-stack type safety
✅ Authentication (email, OAuth)
✅ Stripe integration
✅ Email sending
✅ Cron jobs
✅ File uploads
✅ Admin dashboard
✅ Landing page
✅ Blog
✅ Analytics
```

**Quick Start:**
```bash
git clone https://github.com/wasp-lang/open-saas.git my-saas
cd my-saas
npm install
wasp db migrate-dev
wasp start
```

**Customization Effort:** 🟡 Medium (3-4 weeks to production)

**Note:** Requires learning Wasp framework, but provides significant productivity boost.

---

### 🥉 #3: apptension/saas-boilerplate

**GitHub:** https://github.com/apptension/saas-boilerplate  
**Stars:** ⭐ 2,800+  
**License:** MIT

**Why Choose This:**
- ✅ Enterprise-grade architecture
- ✅ React + Django (Python backend)
- ✅ AWS deployment ready
- ✅ Comprehensive CI/CD
- ✅ Production-tested
- ✅ Microservices-ready

**Tech Stack:**
- Frontend: React, TypeScript, Vite
- Backend: Django, Python
- Database: PostgreSQL
- Infrastructure: AWS (ECS, RDS, S3)
- Auth: Django Auth + JWT
- Payments: Stripe
- Email: AWS SES

**Features:**
```
✅ Multi-tenant support
✅ User authentication
✅ Subscription billing
✅ Email notifications
✅ File uploads (S3)
✅ Admin panel
✅ API documentation
✅ Docker setup
✅ AWS CDK deployment
✅ CI/CD pipelines
```

**Quick Start:**
```bash
git clone https://github.com/apptension/saas-boilerplate.git my-saas
cd my-saas
make setup
make run
```

**Customization Effort:** 🔴 High (4-6 weeks to production)

**Best For:** Teams with Python/Django experience, enterprise deployments.

---

## Detailed Analysis

### 4. boxyhq/saas-starter-kit

**GitHub:** https://github.com/boxyhq/saas-starter-kit  
**Stars:** ⭐ 3,100+  
**License:** Apache 2.0

**Highlights:**
- Enterprise SSO (SAML, OAuth)
- Directory sync (SCIM)
- Audit logs
- Webhook management
- Next.js + Prisma

**Tech Stack:**
```
Frontend: Next.js, React, Tailwind
Backend: Next.js API Routes
Database: PostgreSQL, Prisma
Auth: NextAuth.js + BoxyHQ
```

**Best For:** B2B SaaS requiring enterprise features (SSO, SCIM).

---

### 5. Saas-Starter-Kit (Next.js + Supabase)

**GitHub:** https://github.com/Blazity/next-saas-starter  
**Stars:** ⭐ 1,200+  
**License:** MIT

**Highlights:**
- Supabase backend
- Stripe subscriptions
- Tailwind + Shadcn UI
- Email templates
- Landing page

**Tech Stack:**
```
Frontend: Next.js 14, React
Backend: Supabase (PostgreSQL, Auth, Storage)
Payments: Stripe
Email: Resend
```

**Best For:** Rapid prototyping, Supabase fans.

---

### 6. Shipped.club

**GitHub:** https://github.com/shipped-club/shipped  
**Stars:** ⭐ 800+  
**License:** MIT

**Highlights:**
- Minimalist approach
- Next.js 14
- Stripe + Lemon Squeezy
- Tailwind CSS
- Quick setup

**Tech Stack:**
```
Frontend: Next.js 14
Backend: Next.js API Routes
Database: PostgreSQL, Prisma
Payments: Stripe, Lemon Squeezy
```

**Best For:** Solo developers, MVPs.

---

### 7. Nextacular

**GitHub:** https://github.com/nextacular/nextacular  
**Stars:** ⭐ 1,500+  
**License:** MIT

**Highlights:**
- Multi-tenancy focused
- Workspace management
- Team collaboration
- Invitation system
- Next.js + Prisma

**Tech Stack:**
```
Frontend: Next.js, React, Tailwind
Backend: Next.js API Routes
Database: PostgreSQL, Prisma
Auth: NextAuth.js
```

**Best For:** Team collaboration apps, workspace-based SaaS.

---

## Comparison Matrix

| Feature | ixartz | Wasp | Apptension | BoxyHQ | Blazity | Shipped | Nextacular |
|---------|--------|------|------------|--------|---------|---------|------------|
| **Stars** | 6.7K | 13.2K | 2.8K | 3.1K | 1.2K | 800 | 1.5K |
| **Framework** | Next.js | Wasp | React+Django | Next.js | Next.js | Next.js | Next.js |
| **Multi-Tenant** | ✅ | ⚠️ | ✅ | ✅ | ⚠️ | ❌ | ✅ |
| **Auth** | Clerk | Built-in | Django | NextAuth | Supabase | NextAuth | NextAuth |
| **Billing** | Ready | Stripe | Stripe | Ready | Stripe | Stripe | Ready |
| **Database** | Prisma | Prisma | Django ORM | Prisma | Supabase | Prisma | Prisma |
| **UI Library** | Shadcn | Custom | Material | Tailwind | Shadcn | Tailwind | Tailwind |
| **TypeScript** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Testing** | ✅ | ✅ | ✅ | ⚠️ | ⚠️ | ⚠️ | ⚠️ |
| **i18n** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **SEO** | ✅ | ✅ | ⚠️ | ⚠️ | ✅ | ✅ | ⚠️ |
| **Docs** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **Maintenance** | Active | Active | Active | Active | Active | Moderate | Moderate |
| **Learning Curve** | Low | Medium | High | Low | Low | Low | Low |
| **Setup Time** | 1 hour | 2 hours | 4 hours | 1 hour | 1 hour | 30 min | 1 hour |
| **Production Ready** | ✅ | ✅ | ✅ | ✅ | ⚠️ | ⚠️ | ⚠️ |

**Legend:**
- ✅ Full support
- ⚠️ Partial support / Needs work
- ❌ Not included

---

## How to Choose

### Decision Tree

```
START
  │
  ├─ Need enterprise features (SSO, SCIM)?
  │   └─ YES → BoxyHQ SaaS Starter Kit
  │   └─ NO → Continue
  │
  ├─ Prefer Python/Django backend?
  │   └─ YES → Apptension SaaS Boilerplate
  │   └─ NO → Continue
  │
  ├─ Want fastest setup (< 1 hour)?
  │   └─ YES → Shipped.club or Blazity
  │   └─ NO → Continue
  │
  ├─ Need multi-workspace/team features?
  │   └─ YES → Nextacular
  │   └─ NO → Continue
  │
  ├─ Want most features out-of-box?
  │   └─ YES → ixartz/SaaS-Boilerplate ⭐ RECOMMENDED
  │   └─ NO → Continue
  │
  └─ Want full-stack framework with superpowers?
      └─ YES → wasp-lang/open-saas
      └─ NO → ixartz/SaaS-Boilerplate (default choice)
```

### By Use Case

**B2C SaaS (Consumer Apps):**
- 🥇 ixartz/SaaS-Boilerplate
- 🥈 wasp-lang/open-saas
- 🥉 Blazity Next SaaS Starter

**B2B SaaS (Business Apps):**
- 🥇 BoxyHQ SaaS Starter Kit
- 🥈 ixartz/SaaS-Boilerplate
- 🥉 Apptension SaaS Boilerplate

**Team Collaboration Apps:**
- 🥇 Nextacular
- 🥈 ixartz/SaaS-Boilerplate
- 🥉 wasp-lang/open-saas

**MVP / Quick Launch:**
- 🥇 Shipped.club
- 🥈 Blazity Next SaaS Starter
- 🥉 wasp-lang/open-saas

**Enterprise / Large Scale:**
- 🥇 Apptension SaaS Boilerplate
- 🥈 BoxyHQ SaaS Starter Kit
- 🥉 ixartz/SaaS-Boilerplate

---

## Fork and Customize Guide

### Step 1: Fork the Repository

```bash
# Fork on GitHub (click Fork button)
# Then clone your fork
git clone https://github.com/YOUR_USERNAME/SaaS-Boilerplate.git my-white-label-saas
cd my-white-label-saas

# Add upstream remote to pull updates
git remote add upstream https://github.com/ixartz/SaaS-Boilerplate.git
```

### Step 2: Customize Branding

**Update package.json:**
```json
{
  "name": "my-white-label-saas",
  "version": "1.0.0",
  "description": "My White Label SaaS Platform",
  "author": "Your Name"
}
```

**Update site config:**
```typescript
// config/site.ts
export const siteConfig = {
  name: "Your SaaS Name",
  description: "Your SaaS Description",
  url: "https://yoursaas.com",
  ogImage: "https://yoursaas.com/og.jpg",
  links: {
    twitter: "https://twitter.com/yoursaas",
    github: "https://github.com/yourusername/yoursaas",
  },
}
```

### Step 3: Add White Label Features

**Create tenant branding table:**
```prisma
// prisma/schema.prisma
model TenantBranding {
  id             String  @id @default(cuid())
  tenantId       String  @unique
  logo           String?
  primaryColor   String  @default("#3b82f6")
  secondaryColor String  @default("#8b5cf6")
  customCss      String? @db.Text
  customDomain   String? @unique
  
  tenant         Tenant  @relation(fields: [tenantId], references: [id])
  
  @@map("tenant_branding")
}
```

**Add branding API:**
```typescript
// app/api/tenant/branding/route.ts
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

export async function PATCH(req: Request) {
  const session = await getServerSession();
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const data = await req.json();
  
  const branding = await prisma.tenantBranding.upsert({
    where: { tenantId: session.user.tenantId },
    update: data,
    create: {
      tenantId: session.user.tenantId,
      ...data,
    },
  });

  return NextResponse.json(branding);
}
```

### Step 4: Keep Updated

```bash
# Fetch upstream changes
git fetch upstream

# Merge upstream changes
git merge upstream/main

# Resolve conflicts if any
# Then push to your fork
git push origin main
```

### Step 5: Deploy

```bash
# Deploy to Vercel
vercel

# Or deploy to your preferred platform
# See DEPLOYMENT.md for details
```

---

## Additional Resources

### Curated Lists

**Awesome SaaS Boilerplates:**
- https://github.com/tyaga001/awesome-saas-boilerplates-and-starter-kits
- https://github.com/smirnov-am/awesome-saas-boilerplates

**Open Source SaaS Directory:**
- https://github.com/open-saas-directory/awesome-saas-directory
- https://github.com/toolworks-dev/open-source-saas

### GitHub Topics

- [#saas](https://github.com/topics/saas)
- [#saas-boilerplate](https://github.com/topics/saas-boilerplate)
- [#saas-starter](https://github.com/topics/saas-starter)
- [#nextjs-saas](https://github.com/topics/nextjs-saas)
- [#multi-tenant](https://github.com/topics/multi-tenant)

### Community

**Discord Servers:**
- Wasp Discord: https://discord.gg/rzdnErX
- Next.js Discord: https://discord.gg/nextjs
- Indie Hackers: https://www.indiehackers.com/

**Reddit:**
- r/SaaS
- r/nextjs
- r/webdev
- r/entrepreneur

---

## Contribution Guide

Want to contribute to these projects?

1. **Star the repository** - Show your support
2. **Report issues** - Help improve the project
3. **Submit PRs** - Add features or fix bugs
4. **Write documentation** - Help others get started
5. **Share your experience** - Write blog posts, tutorials

---

## License Considerations

### MIT License (Most Common)

**Allows:**
- ✅ Commercial use
- ✅ Modification
- ✅ Distribution
- ✅ Private use

**Requires:**
- ⚠️ License and copyright notice

**Forbids:**
- ❌ Liability
- ❌ Warranty

### Apache 2.0 License

**Allows:**
- ✅ Commercial use
- ✅ Modification
- ✅ Distribution
- ✅ Patent use
- ✅ Private use

**Requires:**
- ⚠️ License and copyright notice
- ⚠️ State changes

**Forbids:**
- ❌ Trademark use
- ❌ Liability
- ❌ Warranty

---

## Conclusion

### Our Top Pick: ixartz/SaaS-Boilerplate

**Reasons:**
1. ✅ Most comprehensive feature set
2. ✅ Modern tech stack (Next.js 14, Shadcn UI)
3. ✅ Built-in multi-tenancy
4. ✅ Active maintenance
5. ✅ Great documentation
6. ✅ Production-ready
7. ✅ Easy to customize
8. ✅ Strong community

### Runner-Up: wasp-lang/open-saas

**Reasons:**
1. ✅ Highest star count (community trust)
2. ✅ Unique full-stack framework
3. ✅ Excellent developer experience
4. ✅ Built-in features (auth, payments, email)
5. ⚠️ Requires learning Wasp framework

### For Enterprise: apptension/saas-boilerplate

**Reasons:**
1. ✅ Battle-tested in production
2. ✅ Microservices-ready
3. ✅ AWS deployment
4. ✅ Python/Django backend
5. ⚠️ Higher complexity

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Author:** Babu B  
**Contact:** babukiran.b@gmail.com
