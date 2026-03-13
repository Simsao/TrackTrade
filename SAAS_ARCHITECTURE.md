# TradeLog Pro — Architecture SaaS Complète
## Journal de Trading Interactif · Production-Ready · Fintech Sécurisé

---

## 1. ARCHITECTURE GLOBALE DU SYSTÈME

```
┌─────────────────────────────────────────────────────────────────────┐
│                          INTERNET / CDN                             │
│                       Vercel Edge Network                           │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────────┐
│                        FRONTEND (Next.js 14)                        │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │ Landing Page│  │ Auth Pages   │  │ Dashboard (Journal App)  │   │
│  │ /           │  │ /login       │  │ /dashboard               │   │
│  │             │  │ /register    │  │ /dashboard/stats         │   │
│  │             │  │ /verify      │  │ /dashboard/premium       │   │
│  └─────────────┘  └──────────────┘  └──────────────────────────┘   │
└───────────────────────────────┬─────────────────────────────────────┘
                                │ API Routes (HTTPS only)
┌───────────────────────────────▼─────────────────────────────────────┐
│                     BACKEND (Next.js API Routes)                    │
│  ┌──────────────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │   /api/auth/*    │  │ /api/user/*  │  │ /api/trading/*     │    │
│  │   register       │  │ profile      │  │ connect-account    │    │
│  │   login          │  │ settings     │  │ sync-trades        │    │
│  │   verify-email   │  │ delete       │  │ positions          │    │
│  │   logout         │  └──────────────┘  └────────────────────┘    │
│  │   refresh        │                                               │
│  └──────────────────┘  ┌──────────────────────────────────────┐    │
│                         │ /api/payment/*                        │    │
│                         │ create-checkout (Stripe)              │    │
│                         │ webhook (Stripe — vérifié côté srv)   │    │
│                         │ portal (gestion abonnement)           │    │
│                         └──────────────────────────────────────┘    │
└───────────┬───────────────────────────────────────┬─────────────────┘
            │                                       │
┌───────────▼──────────┐               ┌────────────▼──────────────┐
│    PostgreSQL         │               │    Services Externes       │
│    (Supabase)         │               │  ┌─────────────────────┐  │
│  ┌────────────────┐  │               │  │ Stripe (paiements)  │  │
│  │ users          │  │               │  └─────────────────────┘  │
│  │ sessions       │  │               │  ┌─────────────────────┐  │
│  │ trades         │  │               │  │ Resend (emails)     │  │
│  │ trading_apis   │  │               │  └─────────────────────┘  │
│  │ subscriptions  │  │               │  ┌─────────────────────┐  │
│  └────────────────┘  │               │  │ Broker APIs (MT4/5) │  │
└──────────────────────┘               │  └─────────────────────┘  │
                                       └───────────────────────────┘
```

---

## 2. STRUCTURE DES DOSSIERS

```
tradelog-pro/
├── app/                              # Next.js 14 App Router
│   ├── (public)/                     # Routes publiques (sans auth)
│   │   ├── page.tsx                  # Landing page
│   │   ├── pricing/page.tsx          # Page tarifs
│   │   └── layout.tsx
│   ├── (auth)/                       # Routes d'authentification
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   ├── verify-email/page.tsx
│   │   ├── forgot-password/page.tsx
│   │   └── reset-password/page.tsx
│   ├── (dashboard)/                  # Routes protégées (auth requise)
│   │   ├── dashboard/
│   │   │   ├── page.tsx              # Journal principal
│   │   │   ├── stats/page.tsx
│   │   │   ├── calendar/page.tsx
│   │   │   ├── gallery/page.tsx
│   │   │   ├── academy/page.tsx
│   │   │   └── settings/page.tsx
│   │   ├── premium/
│   │   │   ├── page.tsx              # Upsell premium
│   │   │   ├── connect/page.tsx      # Liaison compte trading
│   │   │   └── sync/page.tsx         # Synchronisation
│   │   └── layout.tsx                # Layout avec sidebar + auth guard
│   └── api/                          # API Routes (backend)
│       ├── auth/
│       │   ├── register/route.ts
│       │   ├── login/route.ts
│       │   ├── logout/route.ts
│       │   ├── verify-email/route.ts
│       │   ├── forgot-password/route.ts
│       │   └── reset-password/route.ts
│       ├── user/
│       │   ├── profile/route.ts
│       │   └── delete/route.ts
│       ├── trades/
│       │   ├── route.ts              # GET (list) + POST (create)
│       │   └── [id]/route.ts         # GET + PUT + DELETE
│       ├── payment/
│       │   ├── create-checkout/route.ts
│       │   ├── webhook/route.ts      # Stripe webhook (raw body)
│       │   └── portal/route.ts       # Stripe Customer Portal
│       └── trading/
│           ├── connect-account/route.ts
│           ├── sync-trades/route.ts
│           └── positions/route.ts
├── components/
│   ├── ui/                           # Composants UI réutilisables
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   ├── Toast.tsx
│   │   └── Badge.tsx
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── RegisterForm.tsx
│   │   └── AuthGuard.tsx
│   ├── dashboard/
│   │   ├── TradingJournal.tsx        # Votre composant existant (adapté)
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── PremiumGate.tsx           # Composant gate premium
│   ├── payment/
│   │   ├── PricingCard.tsx
│   │   ├── CheckoutButton.tsx
│   │   └── PremiumBadge.tsx
│   └── landing/
│       ├── Hero.tsx
│       ├── Features.tsx
│       ├── Pricing.tsx
│       └── Footer.tsx
├── lib/
│   ├── db/
│   │   ├── index.ts                  # Client PostgreSQL (Prisma/Supabase)
│   │   └── schema.prisma             # Schéma base de données
│   ├── auth/
│   │   ├── session.ts                # Gestion JWT / cookies
│   │   ├── password.ts               # Bcrypt hash/verify
│   │   └── tokens.ts                 # Tokens email verification
│   ├── email/
│   │   ├── index.ts                  # Client Resend
│   │   └── templates/
│   │       ├── verify-email.tsx      # Template React Email
│   │       └── reset-password.tsx
│   ├── stripe/
│   │   ├── index.ts                  # Client Stripe
│   │   └── webhooks.ts               # Handlers webhook
│   ├── trading/
│   │   ├── encryption.ts             # AES-256 clés API
│   │   ├── brokers/
│   │   │   ├── mt4.ts
│   │   │   ├── mt5.ts
│   │   │   └── tradelocker.ts
│   │   └── sync.ts                   # Logique de synchronisation
│   └── middleware/
│       ├── auth.ts                   # Middleware d'authentification
│       ├── premium.ts                # Middleware vérification premium
│       └── rateLimit.ts              # Rate limiting
├── middleware.ts                     # Next.js middleware global
├── prisma/
│   └── schema.prisma
├── .env.local                        # Variables d'environnement (jamais committé)
├── .env.example                      # Template des variables
├── next.config.js
├── package.json
└── tsconfig.json
```

---

## 3. SCHÉMA BASE DE DONNÉES (PostgreSQL via Prisma)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                String        @id @default(cuid())
  email             String        @unique
  emailVerified     Boolean       @default(false)
  passwordHash      String
  name              String?
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt

  // Relations
  sessions          Session[]
  trades            Trade[]
  subscription      Subscription?
  tradingAccounts   TradingAccount[]
  verificationTokens EmailVerificationToken[]

  @@map("users")
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  ipAddress String?
  userAgent String?
  createdAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([token])
  @@map("sessions")
}

model EmailVerificationToken {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  type      TokenType
  expiresAt DateTime
  usedAt    DateTime?
  createdAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([token])
  @@map("email_verification_tokens")
}

enum TokenType {
  EMAIL_VERIFICATION
  PASSWORD_RESET
}

model Subscription {
  id                   String             @id @default(cuid())
  userId               String             @unique
  stripeCustomerId     String             @unique
  stripeSubscriptionId String?            @unique
  stripePaymentIntentId String?
  plan                 SubscriptionPlan   @default(FREE)
  status               SubscriptionStatus @default(ACTIVE)
  currentPeriodEnd     DateTime?
  cancelAtPeriodEnd    Boolean            @default(false)
  createdAt            DateTime           @default(now())
  updatedAt            DateTime           @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("subscriptions")
}

enum SubscriptionPlan {
  FREE
  PREMIUM
}

enum SubscriptionStatus {
  ACTIVE
  PAST_DUE
  CANCELED
  INCOMPLETE
}

model Trade {
  id              String   @id @default(cuid())
  userId          String
  date            String
  instrument      String
  direction       String?
  assetClass      String?
  timeframe       String?
  strategy        String?
  setup           String?
  entry           String?
  exit            String?
  sl              String?
  tp1             String?
  tp2             String?
  tp3             String?
  size            String?
  pnl             String?
  emotion         String?
  rulesRespected  String?
  note            String?
  chartsJson      Json?    // Stockage des images en base64 ou URLs S3
  // Si import automatique depuis broker
  brokerTradeId   String?
  importedAt      DateTime?
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([userId, date])
  @@map("trades")
}

model TradingAccount {
  id            String   @id @default(cuid())
  userId        String
  brokerType    String   // "mt4", "mt5", "tradelocker", "tradingview"
  accountName   String
  // IMPORTANT: les clés API sont chiffrées en AES-256
  encryptedApiKey    String?
  encryptedApiSecret String?
  encryptedServer    String?
  encryptedLogin     String?
  // Jamais le mot de passe en clair — hash ou token OAuth uniquement
  lastSyncAt    DateTime?
  isActive      Boolean  @default(true)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("trading_accounts")
}
```

---

## 4. CODE BACKEND ESSENTIEL

### 4.1 — Authentification : Register

```typescript
// app/api/auth/register/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/db";
import { createEmailVerificationToken } from "@/lib/auth/tokens";
import { sendVerificationEmail } from "@/lib/email";

const RegisterSchema = z.object({
  email: z.string().email("Email invalide"),
  password: z.string()
    .min(8, "Minimum 8 caractères")
    .regex(/[A-Z]/, "Au moins une majuscule")
    .regex(/[0-9]/, "Au moins un chiffre"),
  name: z.string().min(2).max(50).optional(),
});

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const parsed = RegisterSchema.safeParse(body);

    if (!parsed.success) {
      return NextResponse.json(
        { error: "Données invalides", details: parsed.error.flatten() },
        { status: 400 }
      );
    }

    const { email, password, name } = parsed.data;
    const normalizedEmail = email.toLowerCase().trim();

    // Vérification email existant — message générique (anti-enumeration)
    const existing = await prisma.user.findUnique({
      where: { email: normalizedEmail },
      select: { id: true },
    });

    if (existing) {
      // Réponse identique pour ne pas révéler l'existence du compte
      return NextResponse.json(
        { message: "Si cet email n'est pas déjà utilisé, vous recevrez un email de confirmation." },
        { status: 200 }
      );
    }

    // Hash bcrypt — cost factor 12 (bon compromis sécurité/perf)
    const passwordHash = await bcrypt.hash(password, 12);

    // Création utilisateur + token en transaction
    const { user, token } = await prisma.$transaction(async (tx) => {
      const user = await tx.user.create({
        data: {
          email: normalizedEmail,
          passwordHash,
          name,
          subscription: {
            create: { plan: "FREE", stripeCustomerId: `temp_${Date.now()}` },
          },
        },
      });

      const token = await createEmailVerificationToken(tx, user.id, "EMAIL_VERIFICATION");
      return { user, token };
    });

    // Envoi email asynchrone (ne pas bloquer la réponse)
    sendVerificationEmail(normalizedEmail, token).catch(console.error);

    return NextResponse.json(
      { message: "Compte créé. Vérifiez votre email pour activer votre compte." },
      { status: 201 }
    );

  } catch (error) {
    console.error("[AUTH_REGISTER]", error);
    return NextResponse.json({ error: "Erreur interne" }, { status: 500 });
  }
}
```

### 4.2 — Authentification : Login avec session JWT

```typescript
// app/api/auth/login/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/db";
import { createSession, setSessionCookie } from "@/lib/auth/session";
import { rateLimit } from "@/lib/middleware/rateLimit";

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
});

export async function POST(req: NextRequest) {
  // Rate limiting : 5 tentatives par IP par 15 minutes
  const ip = req.headers.get("x-forwarded-for") ?? "unknown";
  const limited = await rateLimit(`login:${ip}`, 5, 900);
  if (limited) {
    return NextResponse.json(
      { error: "Trop de tentatives. Réessayez dans 15 minutes." },
      { status: 429 }
    );
  }

  try {
    const body = await req.json();
    const parsed = LoginSchema.safeParse(body);
    if (!parsed.success) {
      return NextResponse.json({ error: "Données invalides" }, { status: 400 });
    }

    const { email, password } = parsed.data;
    const user = await prisma.user.findUnique({
      where: { email: email.toLowerCase().trim() },
      include: { subscription: true },
    });

    // Message générique (anti-enumeration + timing attack protection)
    const GENERIC_ERROR = { error: "Email ou mot de passe incorrect" };

    if (!user) {
      // Simuler le coût bcrypt pour éviter les timing attacks
      await bcrypt.compare(password, "$2a$12$fakehashfortimingreasonsonlydontusethis");
      return NextResponse.json(GENERIC_ERROR, { status: 401 });
    }

    const isValid = await bcrypt.compare(password, user.passwordHash);
    if (!isValid) {
      return NextResponse.json(GENERIC_ERROR, { status: 401 });
    }

    if (!user.emailVerified) {
      return NextResponse.json(
        { error: "Veuillez vérifier votre email avant de vous connecter." },
        { status: 403 }
      );
    }

    // Créer la session
    const sessionToken = await createSession(user.id, req);

    // Réponse avec cookie httpOnly sécurisé
    const response = NextResponse.json({
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        plan: user.subscription?.plan ?? "FREE",
        emailVerified: user.emailVerified,
      },
    });

    setSessionCookie(response, sessionToken);
    return response;

  } catch (error) {
    console.error("[AUTH_LOGIN]", error);
    return NextResponse.json({ error: "Erreur interne" }, { status: 500 });
  }
}
```

### 4.3 — Gestion des sessions (cookies httpOnly)

```typescript
// lib/auth/session.ts
import { SignJWT, jwtVerify } from "jose";
import { cookies } from "next/headers";
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/db";

const JWT_SECRET = new TextEncoder().encode(process.env.JWT_SECRET!);
const SESSION_DURATION = 7 * 24 * 60 * 60; // 7 jours en secondes

export async function createSession(userId: string, req: NextRequest): Promise<string> {
  const expiresAt = new Date(Date.now() + SESSION_DURATION * 1000);

  // Token JWT signé
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: "HS256" })
    .setIssuedAt()
    .setExpirationTime(expiresAt)
    .sign(JWT_SECRET);

  // Stocker en base pour invalidation possible
  await prisma.session.create({
    data: {
      userId,
      token,
      expiresAt,
      ipAddress: req.headers.get("x-forwarded-for") ?? undefined,
      userAgent: req.headers.get("user-agent") ?? undefined,
    },
  });

  return token;
}

export function setSessionCookie(response: NextResponse, token: string) {
  response.cookies.set("session", token, {
    httpOnly: true,         // Inaccessible au JavaScript client
    secure: process.env.NODE_ENV === "production",  // HTTPS uniquement en prod
    sameSite: "lax",        // Protection CSRF
    maxAge: SESSION_DURATION,
    path: "/",
  });
}

export async function getSession(req: NextRequest) {
  const token = req.cookies.get("session")?.value;
  if (!token) return null;

  try {
    const { payload } = await jwtVerify(token, JWT_SECRET);
    const userId = payload.userId as string;

    // Vérifier que la session existe en base (permet l'invalidation)
    const session = await prisma.session.findUnique({
      where: { token },
      include: {
        user: {
          include: { subscription: true },
        },
      },
    });

    if (!session || session.expiresAt < new Date()) return null;

    return session.user;
  } catch {
    return null;
  }
}

export async function deleteSession(token: string) {
  await prisma.session.delete({ where: { token } }).catch(() => {});
}
```

### 4.4 — Stripe : Création de session de paiement

```typescript
// app/api/payment/create-checkout/route.ts
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { getSession } from "@/lib/auth/session";
import { prisma } from "@/lib/db";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-06-20",
  typescript: true,
});

export async function POST(req: NextRequest) {
  const user = await getSession(req);
  if (!user) {
    return NextResponse.json({ error: "Non authentifié" }, { status: 401 });
  }

  // Vérifier que l'utilisateur n'est pas déjà premium
  if (user.subscription?.plan === "PREMIUM") {
    return NextResponse.json({ error: "Déjà abonné premium" }, { status: 400 });
  }

  try {
    // Récupérer ou créer le customer Stripe
    let stripeCustomerId = user.subscription?.stripeCustomerId;

    if (!stripeCustomerId || stripeCustomerId.startsWith("temp_")) {
      const customer = await stripe.customers.create({
        email: user.email,
        name: user.name ?? undefined,
        metadata: { userId: user.id },
      });
      stripeCustomerId = customer.id;

      await prisma.subscription.upsert({
        where: { userId: user.id },
        update: { stripeCustomerId },
        create: { userId: user.id, stripeCustomerId, plan: "FREE" },
      });
    }

    const baseUrl = process.env.NEXT_PUBLIC_APP_URL!;

    // Créer la session Stripe Checkout
    const session = await stripe.checkout.sessions.create({
      customer: stripeCustomerId,
      mode: "payment",  // Paiement unique (ou "subscription" pour abonnement récurrent)
      line_items: [
        {
          price_data: {
            currency: "eur",
            product_data: {
              name: "TradeLog Pro — Premium",
              description: "Accès à vie aux fonctionnalités premium : liaison compte de trading, import automatique, statistiques avancées",
              images: [`${baseUrl}/og-premium.png`],
            },
            unit_amount: 1299, // 12.99€ en centimes
          },
          quantity: 1,
        },
      ],
      payment_method_types: ["card"],
      // Apple Pay / Google Pay activés automatiquement si configurés dans Stripe Dashboard
      success_url: `${baseUrl}/dashboard/premium?success=1&session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${baseUrl}/premium?canceled=1`,
      metadata: { userId: user.id },
      allow_promotion_codes: true,
      billing_address_collection: "auto",
    });

    return NextResponse.json({ url: session.url });

  } catch (error) {
    console.error("[STRIPE_CHECKOUT]", error);
    return NextResponse.json({ error: "Erreur création paiement" }, { status: 500 });
  }
}
```

### 4.5 — Stripe Webhook : Activation premium automatique

```typescript
// app/api/payment/webhook/route.ts
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { prisma } from "@/lib/db";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-06-20",
});

// IMPORTANT: Ne pas parser le body en JSON — Stripe nécessite le raw body pour la signature
export async function POST(req: NextRequest) {
  const body = await req.text();  // Raw body
  const signature = req.headers.get("stripe-signature");

  if (!signature) {
    return NextResponse.json({ error: "No signature" }, { status: 400 });
  }

  let event: Stripe.Event;

  try {
    // Vérification cryptographique de la signature Stripe
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error("[STRIPE_WEBHOOK] Signature invalide:", err);
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  // Traiter les événements pertinents
  switch (event.type) {

    case "checkout.session.completed": {
      const session = event.data.object as Stripe.Checkout.Session;

      if (session.payment_status === "paid") {
        const userId = session.metadata?.userId;
        if (!userId) break;

        await prisma.subscription.upsert({
          where: { userId },
          update: {
            plan: "PREMIUM",
            status: "ACTIVE",
            stripePaymentIntentId: session.payment_intent as string,
          },
          create: {
            userId,
            stripeCustomerId: session.customer as string,
            stripePaymentIntentId: session.payment_intent as string,
            plan: "PREMIUM",
            status: "ACTIVE",
          },
        });

        console.log(`[STRIPE] Premium activé pour userId: ${userId}`);
      }
      break;
    }

    case "payment_intent.payment_failed": {
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      const customerId = paymentIntent.customer as string;
      if (customerId) {
        await prisma.subscription.updateMany({
          where: { stripeCustomerId: customerId },
          data: { status: "PAST_DUE" },
        });
      }
      break;
    }

    case "customer.subscription.deleted": {
      // Pour les abonnements récurrents
      const subscription = event.data.object as Stripe.Subscription;
      await prisma.subscription.updateMany({
        where: { stripeSubscriptionId: subscription.id },
        data: { plan: "FREE", status: "CANCELED" },
      });
      break;
    }
  }

  return NextResponse.json({ received: true });
}
```

### 4.6 — Chiffrement des clés API de trading

```typescript
// lib/trading/encryption.ts
import { createCipheriv, createDecipheriv, randomBytes, scrypt } from "crypto";
import { promisify } from "util";

const scryptAsync = promisify(scrypt);
const ALGORITHM = "aes-256-gcm";

// La clé maître est en variable d'environnement, JAMAIS en base de données
const MASTER_KEY = process.env.ENCRYPTION_MASTER_KEY!;

if (!MASTER_KEY || MASTER_KEY.length < 32) {
  throw new Error("ENCRYPTION_MASTER_KEY must be at least 32 chars");
}

export async function encryptApiKey(plaintext: string): Promise<string> {
  // IV aléatoire unique pour chaque chiffrement
  const iv = randomBytes(16);
  const salt = randomBytes(32);

  // Dérivation de clé via scrypt (résistant aux attaques brute-force)
  const key = (await scryptAsync(MASTER_KEY, salt, 32)) as Buffer;

  const cipher = createCipheriv(ALGORITHM, key, iv);
  const encrypted = Buffer.concat([
    cipher.update(plaintext, "utf8"),
    cipher.final(),
  ]);

  const authTag = cipher.getAuthTag();

  // Stocker: salt + iv + authTag + données chiffrées (tout en hex)
  return [
    salt.toString("hex"),
    iv.toString("hex"),
    authTag.toString("hex"),
    encrypted.toString("hex"),
  ].join(":");
}

export async function decryptApiKey(encryptedData: string): Promise<string> {
  const [saltHex, ivHex, authTagHex, encryptedHex] = encryptedData.split(":");

  const salt = Buffer.from(saltHex, "hex");
  const iv = Buffer.from(ivHex, "hex");
  const authTag = Buffer.from(authTagHex, "hex");
  const encrypted = Buffer.from(encryptedHex, "hex");

  const key = (await scryptAsync(MASTER_KEY, salt, 32)) as Buffer;

  const decipher = createDecipheriv(ALGORITHM, key, iv);
  decipher.setAuthTag(authTag);

  const decrypted = Buffer.concat([
    decipher.update(encrypted),
    decipher.final(),
  ]);

  return decrypted.toString("utf8");
}
```

### 4.7 — Middleware d'authentification global

```typescript
// middleware.ts (à la racine du projet)
import { NextRequest, NextResponse } from "next/server";
import { jwtVerify } from "jose";

const JWT_SECRET = new TextEncoder().encode(process.env.JWT_SECRET!);

const PUBLIC_PATHS = ["/", "/login", "/register", "/verify-email", "/pricing",
  "/forgot-password", "/reset-password", "/api/auth/", "/api/payment/webhook"];

export async function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;

  // Laisser passer les routes publiques
  if (PUBLIC_PATHS.some(p => pathname.startsWith(p))) {
    return NextResponse.next();
  }

  const token = req.cookies.get("session")?.value;

  if (!token) {
    const loginUrl = new URL("/login", req.url);
    loginUrl.searchParams.set("redirect", pathname);
    return NextResponse.redirect(loginUrl);
  }

  try {
    const { payload } = await jwtVerify(token, JWT_SECRET);

    // Injecter l'userId dans les headers pour les API routes
    const response = NextResponse.next();
    response.headers.set("x-user-id", payload.userId as string);
    return response;
  } catch {
    const loginUrl = new URL("/login", req.url);
    return NextResponse.redirect(loginUrl);
  }
}

export const config = {
  matcher: ["/dashboard/:path*", "/premium/:path*", "/api/trades/:path*",
            "/api/user/:path*", "/api/payment/create-checkout",
            "/api/trading/:path*"],
};
```

### 4.8 — Protection des routes Premium

```typescript
// lib/middleware/premium.ts
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/db";

export async function requirePremium(req: NextRequest) {
  // L'userId a été injecté par le middleware global
  const userId = req.headers.get("x-user-id");
  if (!userId) {
    return NextResponse.json({ error: "Non authentifié" }, { status: 401 });
  }

  const subscription = await prisma.subscription.findUnique({
    where: { userId },
    select: { plan: true, status: true },
  });

  if (!subscription || subscription.plan !== "PREMIUM" || subscription.status !== "ACTIVE") {
    return NextResponse.json(
      { error: "Fonctionnalité réservée aux membres Premium", upgrade: true },
      { status: 403 }
    );
  }

  return null; // null = autorisé
}

// Utilisation dans une route premium
// app/api/trading/sync-trades/route.ts
export async function POST(req: NextRequest) {
  const premiumCheck = await requirePremium(req);
  if (premiumCheck) return premiumCheck;  // 403 si non premium

  // Logique de synchronisation...
}
```

---

## 5. CODE FRONTEND ESSENTIEL

### 5.1 — Composant de paiement (CheckoutButton)

```tsx
// components/payment/CheckoutButton.tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";

interface CheckoutButtonProps {
  className?: string;
}

export function CheckoutButton({ className }: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const router = useRouter();

  async function handleCheckout() {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/payment/create-checkout", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.error || "Erreur lors de la création du paiement");
      }

      const { url } = await response.json();

      // Redirection vers Stripe Checkout
      window.location.href = url;

    } catch (err) {
      setError(err instanceof Error ? err.message : "Erreur inconnue");
      setLoading(false);
    }
  }

  return (
    <div>
      <button onClick={handleCheckout} disabled={loading} className={className}>
        {loading ? "Chargement…" : "🚀 Passer Premium — 12,99€"}
      </button>
      {error && <p style={{ color: "red", fontSize: 12, marginTop: 8 }}>{error}</p>}
    </div>
  );
}
```

### 5.2 — Formulaire d'inscription

```tsx
// components/auth/RegisterForm.tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";

export function RegisterForm() {
  const [form, setForm] = useState({ email: "", password: "", name: "" });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [loading, setLoading] = useState(false);
  const [success, setSuccess] = useState(false);
  const router = useRouter();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setLoading(true);
    setErrors({});

    try {
      const res = await fetch("/api/auth/register", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form),
      });

      const data = await res.json();

      if (!res.ok) {
        if (data.details?.fieldErrors) {
          setErrors(Object.fromEntries(
            Object.entries(data.details.fieldErrors).map(([k, v]) => [k, (v as string[])[0]])
          ));
        } else {
          setErrors({ global: data.error });
        }
        return;
      }

      setSuccess(true);
    } finally {
      setLoading(false);
    }
  }

  if (success) {
    return (
      <div>
        <h2>✅ Compte créé avec succès !</h2>
        <p>Vérifiez votre email <strong>{form.email}</strong> pour activer votre compte.</p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      {errors.global && <div className="error">{errors.global}</div>}

      <div>
        <label>Prénom (optionnel)</label>
        <input
          value={form.name}
          onChange={e => setForm(f => ({ ...f, name: e.target.value }))}
          placeholder="Votre prénom"
        />
      </div>

      <div>
        <label>Email *</label>
        <input
          type="email"
          value={form.email}
          onChange={e => setForm(f => ({ ...f, email: e.target.value }))}
          required
        />
        {errors.email && <span className="field-error">{errors.email}</span>}
      </div>

      <div>
        <label>Mot de passe *</label>
        <input
          type="password"
          value={form.password}
          onChange={e => setForm(f => ({ ...f, password: e.target.value }))}
          required
          minLength={8}
        />
        {errors.password && <span className="field-error">{errors.password}</span>}
        <small>Minimum 8 caractères, une majuscule, un chiffre</small>
      </div>

      <button type="submit" disabled={loading}>
        {loading ? "Création du compte…" : "Créer mon compte"}
      </button>
    </form>
  );
}
```

### 5.3 — Hook d'authentification

```typescript
// hooks/useAuth.ts
"use client";
import { createContext, useContext, useEffect, useState } from "react";
import { useRouter } from "next/navigation";

interface User {
  id: string;
  email: string;
  name?: string;
  plan: "FREE" | "PREMIUM";
  emailVerified: boolean;
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  isPremium: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const router = useRouter();

  useEffect(() => {
    fetch("/api/user/profile")
      .then(res => res.ok ? res.json() : null)
      .then(data => {
        setUser(data?.user ?? null);
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, []);

  async function login(email: string, password: string) {
    const res = await fetch("/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });

    if (!res.ok) {
      const data = await res.json();
      throw new Error(data.error);
    }

    const data = await res.json();
    setUser(data.user);
    router.push("/dashboard");
  }

  async function logout() {
    await fetch("/api/auth/logout", { method: "POST" });
    setUser(null);
    router.push("/login");
  }

  return (
    <AuthContext.Provider value={{
      user,
      loading,
      isPremium: user?.plan === "PREMIUM",
      login,
      logout,
    }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}
```

### 5.4 — Composant PremiumGate

```tsx
// components/dashboard/PremiumGate.tsx
"use client";
import { useAuth } from "@/hooks/useAuth";
import { CheckoutButton } from "@/components/payment/CheckoutButton";

interface PremiumGateProps {
  children: React.ReactNode;
  featureName: string;
}

export function PremiumGate({ children, featureName }: PremiumGateProps) {
  const { isPremium, loading } = useAuth();

  if (loading) return <div>Chargement…</div>;

  if (!isPremium) {
    return (
      <div style={{
        border: "1px solid #7c6fff44",
        borderRadius: 12,
        padding: 32,
        textAlign: "center",
        background: "rgba(124,111,255,0.05)"
      }}>
        <div style={{ fontSize: 32, marginBottom: 12 }}>🔒</div>
        <h3>Fonctionnalité Premium</h3>
        <p style={{ color: "#8888bb", marginBottom: 20 }}>
          <strong>{featureName}</strong> est réservé aux membres Premium.
          Accédez à toutes les fonctionnalités pour 12,99€ — paiement unique.
        </p>
        <div style={{ display: "flex", gap: 16, justifyContent: "center", flexWrap: "wrap" }}>
          <div style={{ fontSize: 12, color: "#8888bb" }}>✅ Liaison compte de trading</div>
          <div style={{ fontSize: 12, color: "#8888bb" }}>✅ Import automatique des trades</div>
          <div style={{ fontSize: 12, color: "#8888bb" }}>✅ Statistiques avancées</div>
          <div style={{ fontSize: 12, color: "#8888bb" }}>✅ Synchronisation en temps réel</div>
        </div>
        <CheckoutButton style={{ marginTop: 24 }} />
      </div>
    );
  }

  return <>{children}</>;
}
```

---

## 6. VARIABLES D'ENVIRONNEMENT

```bash
# .env.example — NE JAMAIS COMMITTER .env.local

# Base de données
DATABASE_URL="postgresql://user:password@host:5432/tradelog_prod"

# Authentification
JWT_SECRET="votre-secret-jwt-tres-long-et-aleatoire-min-64-chars"
NEXTAUTH_URL="https://votre-domaine.com"

# Chiffrement des clés API trading
ENCRYPTION_MASTER_KEY="votre-cle-chiffrement-aes256-exactement-32chars-"

# Stripe
STRIPE_SECRET_KEY="sk_live_..."          # Jamais exposé au frontend
STRIPE_PUBLISHABLE_KEY="pk_live_..."     # Peut être exposé
STRIPE_WEBHOOK_SECRET="whsec_..."        # Pour valider les webhooks

# Email (Resend)
RESEND_API_KEY="re_..."
EMAIL_FROM="noreply@votre-domaine.com"

# App
NEXT_PUBLIC_APP_URL="https://votre-domaine.com"
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_live_..."

# Redis (pour rate limiting)
REDIS_URL="redis://..."
```

---

## 7. SÉCURITÉ — CHECKLIST FINTECH

### Authentification & Sessions
- [x] **Mots de passe** : bcrypt avec cost factor ≥ 12
- [x] **Sessions** : JWT signés HS256 + stockés en base (invalidation possible)
- [x] **Cookies** : httpOnly + Secure + SameSite=Lax
- [x] **Timing attacks** : hash bcrypt même si l'utilisateur n'existe pas
- [x] **Anti-enumeration** : messages d'erreur génériques (login/register)
- [x] **Rate limiting** : 5 tentatives/15 min par IP sur /login
- [x] **Email verification** : obligatoire avant première connexion
- [x] **Tokens** : expiration courte (24h), usage unique, hash en base

### Données Sensibles
- [x] **Clés API trading** : AES-256-GCM avec dérivation scrypt (jamais en clair)
- [x] **Clé maître** : variable d'environnement uniquement (jamais en base)
- [x] **Stripe** : logique de paiement 100% côté serveur
- [x] **Webhook** : vérification signature cryptographique Stripe
- [x] **SQL Injection** : requêtes paramétrées via Prisma ORM

### Infrastructure
- [x] **HTTPS** : obligatoire (Vercel gère automatiquement)
- [x] **Headers** : Content-Security-Policy, X-Frame-Options, HSTS
- [x] **CORS** : configuré strictement (domaine propre uniquement)
- [x] **Variables d'env** : .env.local jamais committé (.gitignore)
- [x] **Logs** : pas de PII (emails, mots de passe) dans les logs

### Paiements (PCI-DSS Lite)
- [x] **Données carte** : jamais touchées par notre serveur (Stripe Elements)
- [x] **Validation côté serveur** : toujours vérifier via Stripe API
- [x] **Idempotence** : vérifier l'état en base avant d'activer premium
- [x] **Webhook** : signature obligatoire, replay attacks impossibles

---

## 8. DÉPLOIEMENT VERCEL

### vercel.json

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self' 'unsafe-eval' https://js.stripe.com; frame-src https://js.stripe.com; connect-src 'self' https://api.stripe.com; img-src 'self' data: blob:;"
        },
        {
          "key": "Strict-Transport-Security",
          "value": "max-age=63072000; includeSubDomains; preload"
        }
      ]
    }
  ],
  "regions": ["cdg1"],
  "functions": {
    "app/api/payment/webhook/route.ts": {
      "maxDuration": 10
    }
  }
}
```

### Configuration Stripe Dashboard
1. **Webhook endpoint** : `https://votre-domaine.com/api/payment/webhook`
2. **Événements à écouter** :
   - `checkout.session.completed`
   - `payment_intent.payment_failed`
   - `customer.subscription.deleted`
3. **Apple Pay** : activer dans Stripe Dashboard → Settings → Payment methods

---

## 9. ARCHITECTURE FUTURES FONCTIONNALITÉS

### Application Mobile (React Native)
```
tradelog-mobile/
├── Utilise les mêmes API REST (/api/*)
├── Stockage token : SecureStore (Expo)
├── Refresh token silencieux
└── Notifications push (Expo Notifications)
```

### Synchronisation Temps Réel
```typescript
// Approche recommandée : Supabase Realtime ou Server-Sent Events
// app/api/trading/sync-stream/route.ts

export async function GET(req: NextRequest) {
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      // Polling Supabase/DB toutes les 30s pour les nouveaux trades
      const interval = setInterval(async () => {
        const newTrades = await fetchNewTradesFromBroker(userId);
        if (newTrades.length > 0) {
          controller.enqueue(encoder.encode(`data: ${JSON.stringify(newTrades)}\n\n`));
        }
      }, 30000);

      req.signal.addEventListener("abort", () => clearInterval(interval));
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive",
    },
  });
}
```

### Abonnement Mensuel (Stripe Subscriptions)
```typescript
// Changer mode: "payment" → mode: "subscription"
// Créer un Price récurrent dans Stripe Dashboard
// Gérer les webhooks: invoice.payment_failed, customer.subscription.updated
```

---

## 10. COMMANDES DE DÉMARRAGE

```bash
# Installation
npx create-next-app@latest tradelog-pro --typescript --tailwind --app
cd tradelog-pro

# Dépendances
npm install prisma @prisma/client bcryptjs jose zod stripe resend
npm install -D @types/bcryptjs

# Initialisation Prisma
npx prisma init
npx prisma generate
npx prisma db push  # En développement
npx prisma migrate deploy  # En production

# Développement
npm run dev

# Déploiement Vercel
npx vercel --prod
```

---

*Architecture conçue pour une mise en production immédiate. Stack : Next.js 14, PostgreSQL/Supabase, Stripe, Resend, Vercel. Conformité RGPD assurée par le stockage EU (région cdg1 Paris).*
