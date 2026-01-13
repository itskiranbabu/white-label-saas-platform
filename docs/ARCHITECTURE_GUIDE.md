# 🏗️ White Label SaaS Architecture Guide

> Complete technical architecture guide for building scalable, secure, and maintainable white label SaaS platforms.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Multi-Tenant Patterns](#multi-tenant-patterns)
- [System Components](#system-components)
- [Database Design](#database-design)
- [Security Architecture](#security-architecture)
- [Scalability Patterns](#scalability-patterns)
- [Infrastructure](#infrastructure)

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CDN / Load Balancer                      │
│                    (CloudFlare / AWS CloudFront)                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼────────┐  ┌────────▼────────┐  ┌───────▼────────┐
│   Web Client   │  │   Mobile App    │  │  Admin Panel   │
│   (Next.js)    │  │  (React Native) │  │    (React)     │
└───────┬────────┘  └────────┬────────┘  └───────┬────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   API Gateway   │
                    │  (Rate Limiting) │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼────────┐  ┌────────▼────────┐  ┌───────▼────────┐
│  Auth Service  │  │  Core API       │  │  Billing API   │
│  (NextAuth.js) │  │  (Node.js)      │  │  (Stripe)      │
└───────┬────────┘  └────────┬────────┘  └───────┬────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼────────┐  ┌────────▼────────┐  ┌───────▼────────┐
│   PostgreSQL   │  │     Redis       │  │   S3 Storage   │
│  (Primary DB)  │  │  (Cache/Queue)  │  │   (Files)      │
└────────────────┘  └─────────────────┘  └────────────────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼────────┐  ┌────────▼────────┐  ┌───────▼────────┐
│   Analytics    │  │   Email Service │  │   Monitoring   │
│  (Mixpanel)    │  │   (SendGrid)    │  │   (Sentry)     │
└────────────────┘  └─────────────────┘  └────────────────┘
```

### Architecture Principles

1. **Separation of Concerns** - Clear boundaries between components
2. **Scalability** - Horizontal scaling capability
3. **Security** - Defense in depth
4. **Maintainability** - Clean code, documentation
5. **Performance** - Caching, optimization
6. **Reliability** - Fault tolerance, redundancy

---

## Multi-Tenant Patterns

### Pattern 1: Database Per Tenant (Highest Isolation)

**Architecture:**
```
┌─────────────────────────────────────────┐
│         Application Layer               │
│  (Tenant Router / Connection Manager)   │
└──────────┬──────────────────────────────┘
           │
    ┌──────┴──────┬──────────┬──────────┐
    │             │          │          │
┌───▼───┐   ┌────▼────┐ ┌───▼────┐ ┌───▼────┐
│ DB-1  │   │  DB-2   │ │  DB-3  │ │  DB-N  │
│Tenant1│   │ Tenant2 │ │Tenant3 │ │TenantN │
└───────┘   └─────────┘ └────────┘ └────────┘
```

**Pros:**
- ✅ Maximum data isolation
- ✅ Easy backup/restore per tenant
- ✅ Regulatory compliance (HIPAA, GDPR)
- ✅ Custom schema per tenant
- ✅ Easy to migrate tenants

**Cons:**
- ❌ Higher infrastructure costs
- ❌ Complex connection management
- ❌ Difficult to implement cross-tenant features
- ❌ Schema updates across all databases

**Best For:**
- Enterprise clients
- Healthcare/Finance (compliance-heavy)
- High-value customers
- Regulated industries

**Implementation:**
```typescript
// Tenant database connection manager
class TenantDatabaseManager {
  private connections: Map<string, Connection> = new Map();

  async getConnection(tenantId: string): Promise<Connection> {
    if (!this.connections.has(tenantId)) {
      const config = await this.getTenantDbConfig(tenantId);
      const connection = await createConnection(config);
      this.connections.set(tenantId, connection);
    }
    return this.connections.get(tenantId)!;
  }

  private async getTenantDbConfig(tenantId: string) {
    return {
      host: process.env.DB_HOST,
      database: `tenant_${tenantId}`,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
    };
  }
}
```

---

### Pattern 2: Shared Database, Separate Schemas (Balanced)

**Architecture:**
```
┌─────────────────────────────────────────┐
│         Application Layer               │
│      (Schema Context Manager)           │
└──────────┬──────────────────────────────┘
           │
    ┌──────▼──────────────────────────┐
    │      PostgreSQL Database        │
    │  ┌──────────────────────────┐   │
    │  │  Schema: tenant_1        │   │
    │  │  - users                 │   │
    │  │  - products              │   │
    │  └──────────────────────────┘   │
    │  ┌──────────────────────────┐   │
    │  │  Schema: tenant_2        │   │
    │  │  - users                 │   │
    │  │  - products              │   │
    │  └──────────────────────────┘   │
    │  ┌──────────────────────────┐   │
    │  │  Schema: tenant_N        │   │
    │  └──────────────────────────┘   │
    └─────────────────────────────────┘
```

**Pros:**
- ✅ Good data isolation
- ✅ Cost-effective
- ✅ Easier management than separate DBs
- ✅ Better resource utilization
- ✅ Simpler connection pooling

**Cons:**
- ❌ Schema migration complexity
- ❌ Potential noisy neighbor issues
- ❌ Backup/restore more complex
- ❌ Limited by single DB capacity

**Best For:**
- Mid-market SaaS
- B2B platforms
- 100-1000 tenants
- Balanced security/cost needs

**Implementation:**
```typescript
// Prisma with schema-based multi-tenancy
import { PrismaClient } from '@prisma/client';

class TenantPrismaClient {
  private clients: Map<string, PrismaClient> = new Map();

  getClient(tenantId: string): PrismaClient {
    if (!this.clients.has(tenantId)) {
      const client = new PrismaClient({
        datasources: {
          db: {
            url: `${process.env.DATABASE_URL}?schema=tenant_${tenantId}`,
          },
        },
      });
      this.clients.set(tenantId, client);
    }
    return this.clients.get(tenantId)!;
  }
}

// Usage in API
app.get('/api/products', async (req, res) => {
  const tenantId = req.user.tenantId;
  const prisma = tenantPrismaClient.getClient(tenantId);
  const products = await prisma.product.findMany();
  res.json(products);
});
```

---

### Pattern 3: Shared Database, Shared Schema (Most Efficient)

**Architecture:**
```
┌─────────────────────────────────────────┐
│         Application Layer               │
│    (Tenant ID Filter Middleware)        │
└──────────┬──────────────────────────────┘
           │
    ┌──────▼──────────────────────────┐
    │      PostgreSQL Database        │
    │  ┌──────────────────────────┐   │
    │  │  Table: users            │   │
    │  │  - id                    │   │
    │  │  - tenant_id (FK)        │   │
    │  │  - email                 │   │
    │  │  - name                  │   │
    │  └──────────────────────────┘   │
    │  ┌──────────────────────────┐   │
    │  │  Table: products         │   │
    │  │  - id                    │   │
    │  │  - tenant_id (FK)        │   │
    │  │  - name                  │   │
    │  │  - price                 │   │
    │  └──────────────────────────┘   │
    └─────────────────────────────────┘
```

**Pros:**
- ✅ Lowest cost
- ✅ Easiest to scale horizontally
- ✅ Simple schema updates
- ✅ Cross-tenant analytics easy
- ✅ Efficient resource usage

**Cons:**
- ❌ Requires careful data isolation
- ❌ Security risks if not implemented correctly
- ❌ Noisy neighbor problems
- ❌ Complex queries with tenant filtering

**Best For:**
- SMB-focused SaaS
- High-volume platforms (1000+ tenants)
- Cost-sensitive applications
- Rapid scaling needs

**Implementation:**
```typescript
// Prisma schema with tenant_id
model User {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  email     String
  name      String
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  
  @@index([tenantId])
  @@unique([tenantId, email])
}

model Product {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  name      String
  price     Decimal
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  
  @@index([tenantId])
}

// Middleware to enforce tenant isolation
app.use((req, res, next) => {
  const tenantId = req.user?.tenantId;
  if (!tenantId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Inject tenant filter into all queries
  req.prisma = prisma.$extends({
    query: {
      $allModels: {
        async findMany({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async findFirst({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async create({ args, query }) {
          args.data = { ...args.data, tenantId };
          return query(args);
        },
      },
    },
  });
  
  next();
});
```

---

## System Components

### 1. API Gateway

**Responsibilities:**
- Request routing
- Rate limiting
- Authentication validation
- Request/response transformation
- API versioning

**Implementation:**
```typescript
// API Gateway with rate limiting
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Tenant-based rate limiting
const createTenantRateLimiter = () => {
  return rateLimit({
    store: new RedisStore({
      client: redis,
      prefix: 'rl:',
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: async (req) => {
      const tenant = await getTenant(req.user.tenantId);
      return tenant.plan.apiLimit; // Different limits per plan
    },
    keyGenerator: (req) => {
      return `${req.user.tenantId}:${req.ip}`;
    },
    handler: (req, res) => {
      res.status(429).json({
        error: 'Too many requests',
        retryAfter: req.rateLimit.resetTime,
      });
    },
  });
};

app.use('/api', createTenantRateLimiter());
```

### 2. Authentication Service

**Features:**
- Multi-tenant authentication
- OAuth providers
- JWT token management
- Session handling
- SSO support

**Implementation with NextAuth.js:**
```typescript
// pages/api/auth/[...nextauth].ts
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import { PrismaAdapter } from '@next-auth/prisma-adapter';
import { prisma } from '@/lib/prisma';

export default NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, user }) {
      // Add tenant info to session
      const userWithTenant = await prisma.user.findUnique({
        where: { id: user.id },
        include: { tenant: true },
      });
      
      session.user.tenantId = userWithTenant?.tenantId;
      session.user.role = userWithTenant?.role;
      session.tenant = userWithTenant?.tenant;
      
      return session;
    },
    async signIn({ user, account, profile }) {
      // Custom domain-based tenant detection
      const email = user.email;
      const domain = email.split('@')[1];
      
      // Find or create tenant
      let tenant = await prisma.tenant.findFirst({
        where: { domain },
      });
      
      if (!tenant) {
        tenant = await prisma.tenant.create({
          data: {
            name: domain,
            domain,
            plan: 'free',
          },
        });
      }
      
      // Associate user with tenant
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
});
```

### 3. Billing Service

**Features:**
- Subscription management
- Usage tracking
- Invoice generation
- Payment processing
- Plan upgrades/downgrades

**Implementation with Stripe:**
```typescript
// lib/billing.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export class BillingService {
  // Create customer and subscription
  async createSubscription(tenantId: string, priceId: string) {
    const tenant = await prisma.tenant.findUnique({
      where: { id: tenantId },
      include: { owner: true },
    });

    // Create Stripe customer
    const customer = await stripe.customers.create({
      email: tenant.owner.email,
      metadata: {
        tenantId: tenant.id,
      },
    });

    // Create subscription
    const subscription = await stripe.subscriptions.create({
      customer: customer.id,
      items: [{ price: priceId }],
      payment_behavior: 'default_incomplete',
      expand: ['latest_invoice.payment_intent'],
    });

    // Save to database
    await prisma.tenant.update({
      where: { id: tenantId },
      data: {
        stripeCustomerId: customer.id,
        stripeSubscriptionId: subscription.id,
        plan: this.getPlanFromPriceId(priceId),
      },
    });

    return subscription;
  }

  // Handle usage-based billing
  async recordUsage(tenantId: string, metric: string, quantity: number) {
    const tenant = await prisma.tenant.findUnique({
      where: { id: tenantId },
    });

    if (!tenant.stripeSubscriptionId) {
      throw new Error('No active subscription');
    }

    // Get subscription item for usage reporting
    const subscription = await stripe.subscriptions.retrieve(
      tenant.stripeSubscriptionId
    );

    const usageItem = subscription.items.data.find(
      (item) => item.price.lookup_key === metric
    );

    if (usageItem) {
      await stripe.subscriptionItems.createUsageRecord(
        usageItem.id,
        {
          quantity,
          timestamp: Math.floor(Date.now() / 1000),
        }
      );
    }

    // Track in database for analytics
    await prisma.usageRecord.create({
      data: {
        tenantId,
        metric,
        quantity,
        timestamp: new Date(),
      },
    });
  }

  // Webhook handler
  async handleWebhook(event: Stripe.Event) {
    switch (event.type) {
      case 'customer.subscription.updated':
        await this.handleSubscriptionUpdate(event.data.object);
        break;
      case 'customer.subscription.deleted':
        await this.handleSubscriptionCancellation(event.data.object);
        break;
      case 'invoice.payment_succeeded':
        await this.handlePaymentSuccess(event.data.object);
        break;
      case 'invoice.payment_failed':
        await this.handlePaymentFailure(event.data.object);
        break;
    }
  }
}
```

### 4. File Storage Service

**Implementation with AWS S3:**
```typescript
// lib/storage.ts
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3Client = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

export class StorageService {
  private bucket = process.env.S3_BUCKET!;

  // Upload file with tenant isolation
  async uploadFile(
    tenantId: string,
    file: Buffer,
    filename: string,
    contentType: string
  ) {
    const key = `tenants/${tenantId}/${Date.now()}-${filename}`;

    await s3Client.send(
      new PutObjectCommand({
        Bucket: this.bucket,
        Key: key,
        Body: file,
        ContentType: contentType,
        Metadata: {
          tenantId,
        },
      })
    );

    return {
      key,
      url: `https://${this.bucket}.s3.amazonaws.com/${key}`,
    };
  }

  // Generate presigned URL for secure access
  async getSignedUrl(key: string, expiresIn: number = 3600) {
    const command = new GetObjectCommand({
      Bucket: this.bucket,
      Key: key,
    });

    return await getSignedUrl(s3Client, command, { expiresIn });
  }

  // Verify tenant has access to file
  async verifyAccess(tenantId: string, key: string): Promise<boolean> {
    return key.startsWith(`tenants/${tenantId}/`);
  }
}
```

---

## Database Design

### Core Schema (Prisma)

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Tenant model
model Tenant {
  id                    String   @id @default(cuid())
  name                  String
  slug                  String   @unique
  domain                String?  @unique
  customDomain          String?  @unique @map("custom_domain")
  
  // Branding
  logo                  String?
  primaryColor          String?  @map("primary_color")
  secondaryColor        String?  @map("secondary_color")
  customCss             String?  @map("custom_css")
  
  // Billing
  plan                  String   @default("free")
  stripeCustomerId      String?  @unique @map("stripe_customer_id")
  stripeSubscriptionId  String?  @unique @map("stripe_subscription_id")
  subscriptionStatus    String?  @map("subscription_status")
  
  // Limits
  maxUsers              Int      @default(5) @map("max_users")
  maxStorage            BigInt   @default(1073741824) @map("max_storage") // 1GB
  apiLimit              Int      @default(1000) @map("api_limit")
  
  // Metadata
  createdAt             DateTime @default(now()) @map("created_at")
  updatedAt             DateTime @updatedAt @map("updated_at")
  
  // Relations
  users                 User[]
  apiKeys               ApiKey[]
  usageRecords          UsageRecord[]
  
  @@map("tenants")
}

// User model
model User {
  id            String    @id @default(cuid())
  tenantId      String    @map("tenant_id")
  email         String
  name          String?
  image         String?
  role          String    @default("member") // owner, admin, member
  
  // Auth
  emailVerified DateTime? @map("email_verified")
  password      String?   // For email/password auth
  
  // Metadata
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")
  lastLoginAt   DateTime? @map("last_login_at")
  
  // Relations
  tenant        Tenant    @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  accounts      Account[]
  sessions      Session[]
  
  @@unique([tenantId, email])
  @@index([tenantId])
  @@map("users")
}

// NextAuth models
model Account {
  id                String  @id @default(cuid())
  userId            String  @map("user_id")
  type              String
  provider          String
  providerAccountId String  @map("provider_account_id")
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
  sessionToken String   @unique @map("session_token")
  userId       String   @map("user_id")
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

// API Keys for programmatic access
model ApiKey {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  name      String
  key       String   @unique
  lastUsed  DateTime? @map("last_used")
  createdAt DateTime @default(now()) @map("created_at")
  expiresAt DateTime? @map("expires_at")
  
  tenant    Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  
  @@index([tenantId])
  @@map("api_keys")
}

// Usage tracking
model UsageRecord {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  metric    String   // api_calls, storage, emails_sent, etc.
  quantity  Int
  timestamp DateTime @default(now())
  
  tenant    Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  
  @@index([tenantId, metric, timestamp])
  @@map("usage_records")
}
```

### Row-Level Security (PostgreSQL)

```sql
-- Enable RLS on tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Create policy for tenant isolation
CREATE POLICY tenant_isolation_policy ON users
  USING (tenant_id = current_setting('app.current_tenant_id')::text);

CREATE POLICY tenant_isolation_policy ON products
  USING (tenant_id = current_setting('app.current_tenant_id')::text);

-- Set tenant context in application
-- Before each query:
SET LOCAL app.current_tenant_id = 'tenant_123';
```

---

## Security Architecture

### 1. Authentication & Authorization

**JWT Token Structure:**
```typescript
interface JWTPayload {
  userId: string;
  tenantId: string;
  role: 'owner' | 'admin' | 'member';
  permissions: string[];
  iat: number;
  exp: number;
}
```

**Permission System:**
```typescript
// lib/permissions.ts
const PERMISSIONS = {
  'users:read': ['owner', 'admin', 'member'],
  'users:write': ['owner', 'admin'],
  'users:delete': ['owner'],
  'billing:read': ['owner', 'admin'],
  'billing:write': ['owner'],
  'settings:read': ['owner', 'admin'],
  'settings:write': ['owner'],
};

export function hasPermission(
  role: string,
  permission: string
): boolean {
  return PERMISSIONS[permission]?.includes(role) ?? false;
}

// Middleware
export function requirePermission(permission: string) {
  return (req, res, next) => {
    if (!hasPermission(req.user.role, permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}
```

### 2. Data Encryption

```typescript
// Encryption at rest (database)
import crypto from 'crypto';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY!;
const ALGORITHM = 'aes-256-gcm';

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(ALGORITHM, ENCRYPTION_KEY, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
}

export function decrypt(encryptedText: string): string {
  const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
  
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  const decipher = crypto.createDecipheriv(ALGORITHM, ENCRYPTION_KEY, iv);
  
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

### 3. API Security

```typescript
// Security headers middleware
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));

// CORS configuration
import cors from 'cors';

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests from tenant custom domains
    const tenant = getTenantFromOrigin(origin);
    if (tenant) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
}));

// Input validation
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  role: z.enum(['owner', 'admin', 'member']),
});

app.post('/api/users', async (req, res) => {
  try {
    const data = createUserSchema.parse(req.body);
    // Process validated data
  } catch (error) {
    res.status(400).json({ error: error.errors });
  }
});
```

---

## Scalability Patterns

### 1. Caching Strategy

```typescript
// Redis caching layer
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export class CacheService {
  // Cache with tenant isolation
  async get(tenantId: string, key: string) {
    const cacheKey = `tenant:${tenantId}:${key}`;
    const cached = await redis.get(cacheKey);
    return cached ? JSON.parse(cached) : null;
  }

  async set(
    tenantId: string,
    key: string,
    value: any,
    ttl: number = 3600
  ) {
    const cacheKey = `tenant:${tenantId}:${key}`;
    await redis.setex(cacheKey, ttl, JSON.stringify(value));
  }

  async invalidate(tenantId: string, pattern: string) {
    const keys = await redis.keys(`tenant:${tenantId}:${pattern}`);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }
}

// Usage in API
app.get('/api/products', async (req, res) => {
  const tenantId = req.user.tenantId;
  const cacheKey = 'products:list';
  
  // Try cache first
  let products = await cache.get(tenantId, cacheKey);
  
  if (!products) {
    // Cache miss - fetch from database
    products = await prisma.product.findMany({
      where: { tenantId },
    });
    
    // Cache for 5 minutes
    await cache.set(tenantId, cacheKey, products, 300);
  }
  
  res.json(products);
});
```

### 2. Queue System

```typescript
// Bull queue for background jobs
import Queue from 'bull';

const emailQueue = new Queue('email', process.env.REDIS_URL);

// Producer
export async function sendEmail(
  tenantId: string,
  to: string,
  subject: string,
  body: string
) {
  await emailQueue.add({
    tenantId,
    to,
    subject,
    body,
  }, {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
  });
}

// Consumer
emailQueue.process(async (job) => {
  const { tenantId, to, subject, body } = job.data;
  
  // Get tenant branding
  const tenant = await prisma.tenant.findUnique({
    where: { id: tenantId },
  });
  
  // Send email with tenant branding
  await sendGridClient.send({
    to,
    from: tenant.customDomain || 'noreply@yourapp.com',
    subject,
    html: applyBranding(body, tenant),
  });
});
```

### 3. Database Optimization

```typescript
// Connection pooling
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum pool size
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Read replicas for scaling reads
const readPool = new Pool({
  host: process.env.DB_READ_REPLICA_HOST,
  // ... other config
});

// Query optimization with indexes
// In Prisma schema:
model Product {
  id        String   @id @default(cuid())
  tenantId  String   @map("tenant_id")
  name      String
  price     Decimal
  
  @@index([tenantId]) // Index for tenant queries
  @@index([tenantId, createdAt]) // Composite index for sorted queries
  @@index([name(ops: TextSearchOps)]) // Full-text search index
}
```

---

## Infrastructure

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Kubernetes Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: white-label-saas
spec:
  replicas: 3
  selector:
    matchLabels:
      app: white-label-saas
  template:
    metadata:
      labels:
        app: white-label-saas
    spec:
      containers:
      - name: app
        image: your-registry/white-label-saas:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: white-label-saas
spec:
  selector:
    app: white-label-saas
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Author:** Babu B
