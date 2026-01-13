# 📝 White Label SaaS Implementation Guide

> Step-by-step guide to building a production-ready white label SaaS platform from scratch or using open-source boilerplates.

## Table of Contents

- [Getting Started](#getting-started)
- [Option 1: Using Open Source Boilerplate](#option-1-using-open-source-boilerplate)
- [Option 2: Building from Scratch](#option-2-building-from-scratch)
- [Phase-by-Phase Implementation](#phase-by-phase-implementation)
- [Testing Strategy](#testing-strategy)
- [Deployment](#deployment)

---

## Getting Started

### Prerequisites

**Required:**
- Node.js 18+ installed
- PostgreSQL 15+ installed (or use Supabase/Neon)
- Git installed
- Code editor (VS Code recommended)
- Basic knowledge of React, Next.js, and TypeScript

**Accounts Needed:**
- GitHub account
- Vercel account (for deployment)
- Stripe account (for billing)
- AWS account or Vercel Blob (for file storage)
- Upstash account (for Redis)
- Resend or SendGrid account (for emails)

### Development Environment Setup

```bash
# Install Node.js (using nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18

# Install PostgreSQL (macOS)
brew install postgresql@15
brew services start postgresql@15

# Install PostgreSQL (Ubuntu)
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql

# Verify installations
node --version  # Should be 18+
npm --version
psql --version
```

---

## Option 1: Using Open Source Boilerplate

### Recommended: ixartz/SaaS-Boilerplate

**Why this one?**
- ⭐ 6,690+ stars
- ✅ Next.js 14 with App Router
- ✅ Multi-tenancy built-in
- ✅ Shadcn UI components
- ✅ TypeScript strict mode
- ✅ Production-ready
- ✅ Active maintenance

### Step 1: Clone and Setup

```bash
# Clone the repository
git clone https://github.com/ixartz/SaaS-Boilerplate.git my-white-label-saas
cd my-white-label-saas

# Install dependencies
npm install

# Copy environment variables
cp .env.example .env.local
```

### Step 2: Configure Environment Variables

Edit `.env.local`:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/myapp"

# NextAuth
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="generate-with-openssl-rand-base64-32"

# OAuth Providers
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# Stripe
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_PUBLISHABLE_KEY="pk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Email
RESEND_API_KEY="re_..."

# Storage
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"
AWS_REGION="us-east-1"
AWS_S3_BUCKET="your-bucket-name"

# Redis
REDIS_URL="redis://localhost:6379"
```

### Step 3: Database Setup

```bash
# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev

# Seed database (optional)
npx prisma db seed
```

### Step 4: Run Development Server

```bash
npm run dev
```

Visit `http://localhost:3000` to see your app!

### Step 5: Customize for White Label

**1. Update Branding:**
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

**2. Add Multi-Tenant Features:**
```typescript
// lib/tenant.ts
export async function getTenantFromRequest(req: Request) {
  const hostname = req.headers.get('host');
  
  // Check for custom domain
  const tenant = await prisma.tenant.findFirst({
    where: {
      OR: [
        { customDomain: hostname },
        { slug: hostname?.split('.')[0] },
      ],
    },
  });
  
  return tenant;
}
```

**3. Add White Label Settings:**
```typescript
// app/(dashboard)/settings/branding/page.tsx
export default function BrandingSettings() {
  return (
    <div>
      <h1>Branding Settings</h1>
      <LogoUpload />
      <ColorPicker />
      <CustomDomainConfig />
      <CssEditor />
    </div>
  );
}
```

---

## Option 2: Building from Scratch

### Phase 1: Project Setup (Week 1)

#### Day 1-2: Initialize Project

```bash
# Create Next.js app
npx create-next-app@latest my-white-label-saas --typescript --tailwind --app --src-dir

cd my-white-label-saas

# Install core dependencies
npm install @prisma/client @next-auth/prisma-adapter next-auth
npm install stripe @stripe/stripe-js
npm install zod react-hook-form @hookform/resolvers
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu
npm install class-variance-authority clsx tailwind-merge
npm install lucide-react

# Install dev dependencies
npm install -D prisma
npm install -D @types/node
```

#### Day 3-4: Setup Prisma

```bash
# Initialize Prisma
npx prisma init

# Create database
createdb myapp_dev
```

Create `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Tenant {
  id                    String   @id @default(cuid())
  name                  String
  slug                  String   @unique
  domain                String?  @unique
  customDomain          String?  @unique
  
  // Branding
  logo                  String?
  primaryColor          String?  @default("#3b82f6")
  secondaryColor        String?  @default("#8b5cf6")
  customCss             String?  @db.Text
  
  // Billing
  plan                  String   @default("free")
  stripeCustomerId      String?  @unique
  stripeSubscriptionId  String?  @unique
  
  // Limits
  maxUsers              Int      @default(5)
  apiLimit              Int      @default(1000)
  
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  
  users                 User[]
  apiKeys               ApiKey[]
  
  @@map("tenants")
}

model User {
  id            String    @id @default(cuid())
  tenantId      String
  email         String
  name          String?
  image         String?
  role          String    @default("member")
  emailVerified DateTime?
  password      String?
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  tenant        Tenant    @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  accounts      Account[]
  sessions      Session[]
  
  @@unique([tenantId, email])
  @@index([tenantId])
  @@map("users")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

model ApiKey {
  id        String    @id @default(cuid())
  tenantId  String
  name      String
  key       String    @unique
  lastUsed  DateTime?
  createdAt DateTime  @default(now())
  expiresAt DateTime?
  
  tenant    Tenant    @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  
  @@index([tenantId])
  @@map("api_keys")
}
```

```bash
# Create migration
npx prisma migrate dev --name init

# Generate Prisma client
npx prisma generate
```

#### Day 5-7: Setup Authentication

Create `lib/auth.ts`:

```typescript
import { NextAuthOptions } from "next-auth";
import { PrismaAdapter } from "@next-auth/prisma-adapter";
import GoogleProvider from "next-auth/providers/google";
import { prisma } from "./prisma";

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, user }) {
      const userWithTenant = await prisma.user.findUnique({
        where: { id: user.id },
        include: { tenant: true },
      });
      
      session.user.id = user.id;
      session.user.tenantId = userWithTenant?.tenantId;
      session.user.role = userWithTenant?.role;
      session.tenant = userWithTenant?.tenant;
      
      return session;
    },
    async signIn({ user, account, profile }) {
      const email = user.email!;
      const domain = email.split('@')[1];
      
      // Find or create tenant
      let tenant = await prisma.tenant.findFirst({
        where: { domain },
      });
      
      if (!tenant) {
        tenant = await prisma.tenant.create({
          data: {
            name: domain,
            slug: domain.replace(/\./g, '-'),
            domain,
            plan: 'free',
          },
        });
      }
      
      // Update user with tenant
      await prisma.user.update({
        where: { email },
        data: { tenantId: tenant.id },
      });
      
      return true;
    },
  },
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error',
  },
};
```

Create `app/api/auth/[...nextauth]/route.ts`:

```typescript
import NextAuth from "next-auth";
import { authOptions } from "@/lib/auth";

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };
```

### Phase 2: Core Features (Weeks 2-4)

#### Week 2: Tenant Management

**Create Tenant Context:**

```typescript
// lib/tenant-context.tsx
'use client';

import { createContext, useContext, ReactNode } from 'react';
import { Tenant } from '@prisma/client';

const TenantContext = createContext<Tenant | null>(null);

export function TenantProvider({ 
  tenant, 
  children 
}: { 
  tenant: Tenant; 
  children: ReactNode;
}) {
  return (
    <TenantContext.Provider value={tenant}>
      {children}
    </TenantContext.Provider>
  );
}

export function useTenant() {
  const context = useContext(TenantContext);
  if (!context) {
    throw new Error('useTenant must be used within TenantProvider');
  }
  return context;
}
```

**Create Tenant Middleware:**

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';

export async function middleware(request: NextRequest) {
  const token = await getToken({ req: request });
  
  // Protect dashboard routes
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/auth/signin', request.url));
    }
  }
  
  // Get tenant from hostname or subdomain
  const hostname = request.headers.get('host') || '';
  const tenant = await getTenantFromHostname(hostname);
  
  if (tenant) {
    // Add tenant to request headers
    const requestHeaders = new Headers(request.headers);
    requestHeaders.set('x-tenant-id', tenant.id);
    
    return NextResponse.next({
      request: {
        headers: requestHeaders,
      },
    });
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

#### Week 3: Billing Integration

**Setup Stripe:**

```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export const PLANS = {
  free: {
    name: 'Free',
    price: 0,
    priceId: null,
    features: ['5 users', '1,000 API calls/month', '1GB storage'],
  },
  pro: {
    name: 'Pro',
    price: 29,
    priceId: 'price_pro_monthly',
    features: ['25 users', '10,000 API calls/month', '10GB storage'],
  },
  enterprise: {
    name: 'Enterprise',
    price: 99,
    priceId: 'price_enterprise_monthly',
    features: ['Unlimited users', 'Unlimited API calls', '100GB storage'],
  },
};
```

**Create Checkout API:**

```typescript
// app/api/billing/checkout/route.ts
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import { stripe, PLANS } from '@/lib/stripe';
import { prisma } from '@/lib/prisma';

export async function POST(req: Request) {
  try {
    const session = await getServerSession(authOptions);
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { plan } = await req.json();
    const priceId = PLANS[plan as keyof typeof PLANS]?.priceId;

    if (!priceId) {
      return NextResponse.json({ error: 'Invalid plan' }, { status: 400 });
    }

    const tenant = await prisma.tenant.findUnique({
      where: { id: session.user.tenantId },
    });

    // Create or retrieve Stripe customer
    let customerId = tenant?.stripeCustomerId;
    
    if (!customerId) {
      const customer = await stripe.customers.create({
        email: session.user.email!,
        metadata: {
          tenantId: session.user.tenantId!,
        },
      });
      customerId = customer.id;
      
      await prisma.tenant.update({
        where: { id: session.user.tenantId },
        data: { stripeCustomerId: customerId },
      });
    }

    // Create checkout session
    const checkoutSession = await stripe.checkout.sessions.create({
      customer: customerId,
      mode: 'subscription',
      payment_method_types: ['card'],
      line_items: [
        {
          price: priceId,
          quantity: 1,
        },
      ],
      success_url: `${process.env.NEXTAUTH_URL}/dashboard/billing?success=true`,
      cancel_url: `${process.env.NEXTAUTH_URL}/dashboard/billing?canceled=true`,
      metadata: {
        tenantId: session.user.tenantId!,
      },
    });

    return NextResponse.json({ url: checkoutSession.url });
  } catch (error) {
    console.error('Checkout error:', error);
    return NextResponse.json(
      { error: 'Failed to create checkout session' },
      { status: 500 }
    );
  }
}
```

**Create Webhook Handler:**

```typescript
// app/api/billing/webhook/route.ts
import { NextResponse } from 'next/server';
import { headers } from 'next/headers';
import { stripe } from '@/lib/stripe';
import { prisma } from '@/lib/prisma';
import Stripe from 'stripe';

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (error) {
    return NextResponse.json(
      { error: 'Invalid signature' },
      { status: 400 }
    );
  }

  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated':
      const subscription = event.data.object as Stripe.Subscription;
      await prisma.tenant.update({
        where: { stripeCustomerId: subscription.customer as string },
        data: {
          stripeSubscriptionId: subscription.id,
          plan: getPlanFromPriceId(subscription.items.data[0].price.id),
          subscriptionStatus: subscription.status,
        },
      });
      break;

    case 'customer.subscription.deleted':
      const deletedSubscription = event.data.object as Stripe.Subscription;
      await prisma.tenant.update({
        where: { stripeCustomerId: deletedSubscription.customer as string },
        data: {
          plan: 'free',
          subscriptionStatus: 'canceled',
        },
      });
      break;

    case 'invoice.payment_succeeded':
      // Handle successful payment
      break;

    case 'invoice.payment_failed':
      // Handle failed payment
      break;
  }

  return NextResponse.json({ received: true });
}

function getPlanFromPriceId(priceId: string): string {
  // Map price IDs to plan names
  const priceMap: Record<string, string> = {
    'price_pro_monthly': 'pro',
    'price_enterprise_monthly': 'enterprise',
  };
  return priceMap[priceId] || 'free';
}
```

#### Week 4: Dashboard & UI

**Install Shadcn UI:**

```bash
npx shadcn-ui@latest init
npx shadcn-ui@latest add button card dialog dropdown-menu form input label select table toast
```

**Create Dashboard Layout:**

```typescript
// app/(dashboard)/layout.tsx
import { Sidebar } from '@/components/dashboard/sidebar';
import { Header } from '@/components/dashboard/header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header />
        <main className="flex-1 overflow-y-auto p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

### Phase 3: White Label Features (Weeks 5-8)

#### Week 5-6: Branding Customization

**Logo Upload Component:**

```typescript
// components/settings/logo-upload.tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Upload } from 'lucide-react';

export function LogoUpload() {
  const [uploading, setUploading] = useState(false);

  async function handleUpload(e: React.ChangeEvent<HTMLInputElement>) {
    const file = e.target.files?.[0];
    if (!file) return;

    setUploading(true);

    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await fetch('/api/upload/logo', {
        method: 'POST',
        body: formData,
      });

      const data = await response.json();
      
      // Update tenant logo
      await fetch('/api/tenant/branding', {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ logo: data.url }),
      });

      // Refresh page or update state
    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setUploading(false);
    }
  }

  return (
    <div>
      <label htmlFor="logo-upload">
        <Button asChild disabled={uploading}>
          <span>
            <Upload className="mr-2 h-4 w-4" />
            {uploading ? 'Uploading...' : 'Upload Logo'}
          </span>
        </Button>
      </label>
      <input
        id="logo-upload"
        type="file"
        accept="image/*"
        className="hidden"
        onChange={handleUpload}
      />
    </div>
  );
}
```

**Color Picker Component:**

```typescript
// components/settings/color-picker.tsx
'use client';

import { useState } from 'react';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Button } from '@/components/ui/button';

export function ColorPicker() {
  const [colors, setColors] = useState({
    primary: '#3b82f6',
    secondary: '#8b5cf6',
    accent: '#10b981',
  });

  async function handleSave() {
    await fetch('/api/tenant/branding', {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        primaryColor: colors.primary,
        secondaryColor: colors.secondary,
        accentColor: colors.accent,
      }),
    });
  }

  return (
    <div className="space-y-4">
      <div>
        <Label htmlFor="primary-color">Primary Color</Label>
        <div className="flex gap-2">
          <Input
            id="primary-color"
            type="color"
            value={colors.primary}
            onChange={(e) => setColors({ ...colors, primary: e.target.value })}
          />
          <Input
            type="text"
            value={colors.primary}
            onChange={(e) => setColors({ ...colors, primary: e.target.value })}
          />
        </div>
      </div>
      
      <div>
        <Label htmlFor="secondary-color">Secondary Color</Label>
        <div className="flex gap-2">
          <Input
            id="secondary-color"
            type="color"
            value={colors.secondary}
            onChange={(e) => setColors({ ...colors, secondary: e.target.value })}
          />
          <Input
            type="text"
            value={colors.secondary}
            onChange={(e) => setColors({ ...colors, secondary: e.target.value })}
          />
        </div>
      </div>

      <Button onClick={handleSave}>Save Colors</Button>
    </div>
  );
}
```

#### Week 7-8: Custom Domains

**Domain Configuration:**

```typescript
// app/api/tenant/domain/route.ts
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import { prisma } from '@/lib/prisma';

export async function POST(req: Request) {
  const session = await getServerSession(authOptions);
  if (!session?.user || session.user.role !== 'owner') {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { domain } = await req.json();

  // Validate domain format
  const domainRegex = /^[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,}$/;
  if (!domainRegex.test(domain)) {
    return NextResponse.json({ error: 'Invalid domain' }, { status: 400 });
  }

  // Check if domain is already taken
  const existing = await prisma.tenant.findFirst({
    where: { customDomain: domain },
  });

  if (existing) {
    return NextResponse.json(
      { error: 'Domain already in use' },
      { status: 400 }
    );
  }

  // Update tenant
  await prisma.tenant.update({
    where: { id: session.user.tenantId },
    data: {
      customDomain: domain,
      domainVerified: false,
    },
  });

  return NextResponse.json({
    success: true,
    instructions: {
      type: 'CNAME',
      name: domain,
      value: 'cname.yoursaas.com',
    },
  });
}

export async function GET(req: Request) {
  const session = await getServerSession(authOptions);
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const tenant = await prisma.tenant.findUnique({
    where: { id: session.user.tenantId },
    select: { customDomain: true, domainVerified: true },
  });

  return NextResponse.json(tenant);
}
```

### Phase 4: Polish & Launch (Weeks 9-12)

#### Week 9-10: Testing

**Setup Testing:**

```bash
npm install -D jest @testing-library/react @testing-library/jest-dom @testing-library/user-event
npm install -D @types/jest jest-environment-jsdom
```

**Create Test:**

```typescript
// __tests__/components/logo-upload.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LogoUpload } from '@/components/settings/logo-upload';

describe('LogoUpload', () => {
  it('renders upload button', () => {
    render(<LogoUpload />);
    expect(screen.getByText('Upload Logo')).toBeInTheDocument();
  });

  it('handles file upload', async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve({ url: 'https://example.com/logo.png' }),
      })
    ) as jest.Mock;

    render(<LogoUpload />);
    
    const file = new File(['logo'], 'logo.png', { type: 'image/png' });
    const input = screen.getByLabelText('Upload Logo') as HTMLInputElement;
    
    fireEvent.change(input, { target: { files: [file] } });

    await waitFor(() => {
      expect(global.fetch).toHaveBeenCalledWith(
        '/api/upload/logo',
        expect.objectContaining({ method: 'POST' })
      );
    });
  });
});
```

#### Week 11: Performance Optimization

**Add Caching:**

```typescript
// lib/cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL!,
  token: process.env.REDIS_TOKEN!,
});

export async function getCached<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 3600
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return cached as T;
  }

  // Cache miss - fetch and cache
  const data = await fetcher();
  await redis.setex(key, ttl, JSON.stringify(data));
  
  return data;
}

export async function invalidateCache(pattern: string) {
  const keys = await redis.keys(pattern);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}
```

**Optimize Images:**

```typescript
// next.config.js
module.exports = {
  images: {
    domains: ['your-bucket.s3.amazonaws.com'],
    formats: ['image/avif', 'image/webp'],
  },
};
```

#### Week 12: Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for detailed deployment instructions.

---

## Testing Strategy

### Unit Tests

```bash
# Run unit tests
npm test

# Run with coverage
npm test -- --coverage

# Watch mode
npm test -- --watch
```

### Integration Tests

```typescript
// __tests__/api/tenant.test.ts
import { POST } from '@/app/api/tenant/route';
import { prisma } from '@/lib/prisma';

describe('/api/tenant', () => {
  beforeEach(async () => {
    await prisma.tenant.deleteMany();
  });

  it('creates a new tenant', async () => {
    const request = new Request('http://localhost:3000/api/tenant', {
      method: 'POST',
      body: JSON.stringify({
        name: 'Test Tenant',
        slug: 'test-tenant',
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(201);
    expect(data.tenant.name).toBe('Test Tenant');
  });
});
```

### E2E Tests (Playwright)

```bash
npm install -D @playwright/test
npx playwright install
```

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test('user can sign in', async ({ page }) => {
  await page.goto('http://localhost:3000/auth/signin');
  
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('http://localhost:3000/dashboard');
});
```

---

## Deployment

### Vercel Deployment

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy
vercel

# Production deployment
vercel --prod
```

### Environment Variables

Set these in Vercel dashboard:

```
DATABASE_URL
NEXTAUTH_URL
NEXTAUTH_SECRET
GOOGLE_CLIENT_ID
GOOGLE_CLIENT_SECRET
STRIPE_SECRET_KEY
STRIPE_PUBLISHABLE_KEY
STRIPE_WEBHOOK_SECRET
RESEND_API_KEY
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_S3_BUCKET
REDIS_URL
REDIS_TOKEN
```

### Database Migration

```bash
# Run migrations in production
npx prisma migrate deploy
```

---

## Next Steps

After completing the implementation:

1. **Security Audit** - Review code for vulnerabilities
2. **Performance Testing** - Load testing with k6 or Artillery
3. **Documentation** - API docs, user guides
4. **Monitoring** - Setup Sentry, LogRocket
5. **Analytics** - Implement Mixpanel or PostHog
6. **Beta Testing** - Invite early users
7. **Marketing** - Launch landing page, content
8. **Support** - Setup help desk, documentation

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Author:** Babu B
