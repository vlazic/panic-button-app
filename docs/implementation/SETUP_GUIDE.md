# Setup Guide - Korak po Korak

## Preduslovi

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  PREDUSLOVI                                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  OBAVEZNO:                                                                      │
│  ✓ Node.js 18+ (preporučeno 20.x LTS)                                           │
│  ✓ npm 9+ ili pnpm                                                              │
│  ✓ Git                                                                          │
│  ✓ Telegram nalog (za kreiranje bota)                                           │
│                                                                                 │
│  OPCIONO (za pun sistem):                                                       │
│  ○ Twilio nalog (SMS verifikacija)                                              │
│  ○ Sentry nalog (error tracking)                                                │
│                                                                                 │
│  PROVERA:                                                                       │
│  node --version    # v20.x.x                                                    │
│  npm --version     # 10.x.x                                                     │
│  git --version     # 2.x.x                                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Korak 1: Kreiranje Projekta

### 1.1 Kloniranje ili kreiranje

```bash
# Opcija A: Kloniraj postojeći repo
git clone https://github.com/your-org/panic-button-app.git
cd panic-button-app

# Opcija B: Kreiraj novi Next.js projekat
npx create-next-app@latest panic-button-app --typescript --tailwind --eslint --app --src-dir
cd panic-button-app
```

### 1.2 Instalacija dependency-ja

```bash
# Osnovne dependencies
npm install convex
npm install next-pwa
npm install leaflet react-leaflet
npm install @types/leaflet

# Dev dependencies
npm install -D @types/react @types/node

# Opciono: Sentry
npm install @sentry/nextjs
```

### 1.3 Struktura foldera

```
panic-button-app/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx
│   │   ├── page.tsx           # Login/PIN stranica
│   │   ├── panic/
│   │   │   └── page.tsx       # Panic button
│   │   ├── alarm/
│   │   │   └── [id]/
│   │   │       └── page.tsx   # Alarm detalji
│   │   └── admin/             # (Pun sistem)
│   │       └── page.tsx
│   ├── components/
│   │   ├── PanicButton.tsx
│   │   ├── AlarmCard.tsx
│   │   ├── MapView.tsx
│   │   └── ...
│   └── lib/
│       └── utils.ts
├── convex/
│   ├── schema.ts              # Database schema
│   ├── alarms.ts              # Alarm functions
│   ├── telegram.ts            # Telegram integration
│   └── _generated/            # Auto-generated types
├── public/
│   ├── manifest.json          # PWA manifest
│   └── icons/
└── next.config.js
```

---

## Korak 2: Convex Setup

### 2.1 Inicijalizacija Convex-a

```bash
# Inicijalizuj Convex
npx convex init

# Ovo će:
# 1. Kreirati convex/ folder
# 2. Kreirati Convex projekat na convex.dev
# 3. Dodati NEXT_PUBLIC_CONVEX_URL u .env.local
```

### 2.2 Schema (PoC verzija)

```typescript
// convex/schema.ts

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  alarms: defineTable({
    sender_name: v.string(),
    message: v.optional(v.string()),
    lat: v.number(),
    lng: v.number(),
    status: v.union(
      v.literal("ACTIVE"),
      v.literal("TAKEN"),
      v.literal("RESOLVED"),
      v.literal("CANCELLED"),
      v.literal("FALSE_ALARM")
    ),
    taken_by: v.optional(v.string()),
    taken_at: v.optional(v.number()),
    created_at: v.number(),
    resolved_at: v.optional(v.number()),
  })
    .index("by_status", ["status"])
    .index("by_created", ["created_at"]),
});
```

### 2.3 Push Schema

```bash
# Deploy schema u Convex
npx convex dev

# Ovo će:
# 1. Push-ovati schema
# 2. Generisati TypeScript tipove
# 3. Startovati file watcher
```

---

## Korak 3: Environment Variables

### 3.1 Lokalni development

```bash
# .env.local (NE COMMITOVATI!)

# Convex (automatski dodato)
NEXT_PUBLIC_CONVEX_URL=https://xxx.convex.cloud

# App config
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_GROUP_PIN=123456

# Telegram (dodaj ručno)
TELEGRAM_BOT_TOKEN=your_bot_token_here
TELEGRAM_CHAT_ID=-100123456789
```

### 3.2 Convex Environment Variables

```bash
# Postavi Telegram token u Convex
npx convex env set TELEGRAM_BOT_TOKEN "your_token_here"
npx convex env set TELEGRAM_CHAT_ID "-100123456789"
npx convex env set APP_URL "http://localhost:3000"

# Proveri
npx convex env list
```

---

## Korak 4: Telegram Bot Setup

### 4.1 Kreiranje bota

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TELEGRAM BOT KREIRANJE                                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. Otvori Telegram i pronađi @BotFather                                        │
│                                                                                 │
│  2. Pošalji: /newbot                                                            │
│                                                                                 │
│  3. Unesi ime bota: Patrola Bot                                                 │
│                                                                                 │
│  4. Unesi username: patrola_skola_bot                                           │
│     (mora biti unique i završavati sa "bot")                                    │
│                                                                                 │
│  5. BotFather će ti dati TOKEN - SAČUVAJ GA!                                    │
│     Primer: 123456789:ABCdefGHIjklMNOpqrsTUVwxyz                                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Kreiranje Telegram grupe

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TELEGRAM GRUPA SETUP                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. Kreiraj novu grupu: "OŠ Kovačić - Patrola"                                  │
│                                                                                 │
│  2. Dodaj bota u grupu (pretraži po username-u)                                 │
│                                                                                 │
│  3. OBAVEZNO: Postavi bota kao ADMINA                                           │
│     (Settings → Edit → Administrators → Add Administrator)                      │
│                                                                                 │
│  4. Pošalji bilo koju poruku u grupu                                            │
│                                                                                 │
│  5. Dobij CHAT_ID:                                                              │
│     curl "https://api.telegram.org/bot<TOKEN>/getUpdates"                       │
│                                                                                 │
│  6. U JSON response-u, nađi "chat":{"id":-100123456789}                         │
│     Negativan broj je tvoj CHAT_ID                                              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Test Telegram konekcije

```bash
# Pošalji test poruku
curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "<CHAT_ID>",
    "text": "🚀 Patrola Bot je uspešno povezan!",
    "parse_mode": "Markdown"
  }'

# Ako dobiješ {"ok":true,...} → Uspeh!
```

---

## Korak 5: Frontend Setup

### 5.1 Convex Provider

```typescript
// src/app/layout.tsx

import { ConvexClientProvider } from "@/components/ConvexClientProvider";
import "./globals.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="sr">
      <body>
        <ConvexClientProvider>{children}</ConvexClientProvider>
      </body>
    </html>
  );
}
```

```typescript
// src/components/ConvexClientProvider.tsx
"use client";

import { ConvexProvider, ConvexReactClient } from "convex/react";
import { ReactNode } from "react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function ConvexClientProvider({ children }: { children: ReactNode }) {
  return <ConvexProvider client={convex}>{children}</ConvexProvider>;
}
```

### 5.2 PWA Manifest

```json
// public/manifest.json
{
  "name": "Patrola - Panic Button",
  "short_name": "Patrola",
  "description": "Aplikacija za bezbednost dece",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#EF4444",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### 5.3 Next.js Config za PWA

```javascript
// next.config.js
const withPWA = require("next-pwa")({
  dest: "public",
  disable: process.env.NODE_ENV === "development",
});

/** @type {import('next').NextConfig} */
const nextConfig = {
  // your config
};

module.exports = withPWA(nextConfig);
```

---

## Korak 6: Pokretanje Development Servera

### 6.1 Start

```bash
# Terminal 1: Convex dev server
npx convex dev

# Terminal 2: Next.js dev server
npm run dev
```

### 6.2 Provera

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CHECKLIST - SVE RADI?                                                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ☐ http://localhost:3000 se otvara                                              │
│  ☐ Convex dev pokazuje "Functions updated"                                      │
│  ☐ .env.local ima NEXT_PUBLIC_CONVEX_URL                                        │
│  ☐ Telegram bot odgovara na /getUpdates                                         │
│  ☐ Test poruka stiže u Telegram grupu                                           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Korak 7: Deployment

### 7.1 Vercel Deployment

```bash
# Instaliraj Vercel CLI (opciono)
npm i -g vercel

# Deploy
vercel

# Ili: Connect GitHub repo na vercel.com
```

### 7.2 Vercel Environment Variables

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  VERCEL DASHBOARD → Settings → Environment Variables                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  NAME                        VALUE                   ENVIRONMENT                │
│  ────────────────────────────────────────────────────────────────────────────   │
│  NEXT_PUBLIC_CONVEX_URL      https://xxx.convex.cloud    Production            │
│  NEXT_PUBLIC_APP_URL         https://patrola.rs          Production            │
│  NEXT_PUBLIC_GROUP_PIN       (tvoj PIN)                  Production            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.3 Convex Production Deployment

```bash
# Deploy Convex functions to production
npx convex deploy

# Postavi production environment variables
npx convex env set TELEGRAM_BOT_TOKEN "xxx" --prod
npx convex env set TELEGRAM_CHAT_ID "-100xxx" --prod
npx convex env set APP_URL "https://patrola.rs" --prod
```

### 7.4 Custom Domain (Cloudflare)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CLOUDFLARE DNS SETUP                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. Dodaj sajt na Cloudflare                                                    │
│                                                                                 │
│  2. DNS Records:                                                                │
│     TYPE    NAME    VALUE                       PROXY                           │
│     CNAME   @       cname.vercel-dns.com        ✅ Proxied                      │
│     CNAME   www     cname.vercel-dns.com        ✅ Proxied                      │
│                                                                                 │
│  3. U Vercel: Add domain → patrola.rs                                           │
│                                                                                 │
│  4. Vercel će verifikovati DNS                                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Korak 8: Testing

### 8.1 Manual Testing

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TEST CHECKLIST                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  OSNOVNO:                                                                       │
│  ☐ Login sa PIN-om radi                                                         │
│  ☐ Panic button se drži 3 sekunde                                               │
│  ☐ GPS lokacija se uzima                                                        │
│  ☐ Alarm se kreira u bazi                                                       │
│  ☐ Telegram notifikacija stiže                                                  │
│                                                                                 │
│  PREUZIMANJE:                                                                   │
│  ☐ "Preuzimam" dugme menja status                                               │
│  ☐ Svi vide ko je preuzeo                                                       │
│  ☐ Real-time update radi                                                        │
│                                                                                 │
│  EDGE CASES:                                                                    │
│  ☐ Dva korisnika istovremeno preuzimaju                                         │
│  ☐ Offline pa online                                                            │
│  ☐ GPS nije dostupan                                                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Mobile Testing

```
1. Otvori sajt na telefonu
2. "Add to Home Screen"
3. Otvori iz home screen-a
4. Testiraj panic button
5. Proveri da li notifikacija stiže
```

---

## Troubleshooting

### Česti Problemi

| Problem | Rešenje |
|---------|---------|
| Convex ne connect-uje | Proveri NEXT_PUBLIC_CONVEX_URL |
| Telegram ne šalje | Proveri da li je bot ADMIN u grupi |
| GPS ne radi | Dozvoli location permission, koristi HTTPS |
| PWA ne instalira | Proveri manifest.json, koristi HTTPS |
| Real-time ne radi | Proveri WebSocket (neki firewalls blokiraju) |

### Debug Commands

```bash
# Proveri Convex logs
npx convex logs

# Proveri Convex functions
npx convex functions

# Proveri environment variables
npx convex env list

# Test Telegram API
curl "https://api.telegram.org/bot<TOKEN>/getMe"
```

---

## Sledeći Koraci

Nakon uspešnog setup-a:

1. **PoC Testing** - Testiraj sa malim brojem korisnika
2. **Feedback** - Prikupi feedback od test korisnika
3. **Iterate** - Ispravi probleme
4. **Expand** - Dodaj više funkcionalnosti (pun sistem)

---

*Dokument kreiran: Januar 2026*
