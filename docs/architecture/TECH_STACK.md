# Tehnološki Stack

## Pregled

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            TECH STACK OVERVIEW                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  FRONTEND          BACKEND           SERVICES          HOSTING                  │
│  ────────          ───────           ────────          ───────                  │
│                                                                                 │
│  Next.js 14        Convex            Telegram API      Vercel                   │
│  React 18          (Serverless)      Twilio SMS        Cloudflare               │
│  TailwindCSS       TypeScript        Google Maps       Convex Cloud             │
│  PWA                                                                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Frontend

### Next.js 14

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  NEXT.JS 14                                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERZIJA: 14.x (App Router)                                                     │
│                                                                                 │
│  ZAŠTO NEXT.JS:                                                                 │
│  ✓ Server Components (bolje performance)                                        │
│  ✓ App Router (moderniji routing)                                               │
│  ✓ Built-in optimizacije (images, fonts)                                        │
│  ✓ Odličan Vercel deployment                                                    │
│  ✓ TypeScript first                                                             │
│  ✓ Velika zajednica                                                             │
│                                                                                 │
│  KONFIGURACIJA:                                                                 │
│  • Output: PWA (service worker)                                                 │
│  • Rendering: Client-side za interaktivnost                                     │
│  • API Routes: Nije potrebno (Convex)                                           │
│                                                                                 │
│  ALTERNATIVA RAZMATRANA:                                                        │
│  • Vite + React: Jednostavniji, ali manje optimizacija                          │
│  • Remix: Dobar, ali manje mature PWA support                                   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### React 18

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  REACT 18                                                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  KORIŠĆENE FEATURE:                                                             │
│  • Concurrent rendering                                                         │
│  • Suspense za data fetching                                                    │
│  • useTransition za smooth updates                                              │
│  • Automatic batching                                                           │
│                                                                                 │
│  STATE MANAGEMENT:                                                              │
│  • Convex useQuery/useMutation (primarno)                                       │
│  • React Context (UI state)                                                     │
│  • Bez Redux/Zustand (nepotrebno sa Convex)                                     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### TailwindCSS

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TAILWINDCSS                                                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERZIJA: 3.4.x                                                                 │
│                                                                                 │
│  ZAŠTO TAILWIND:                                                                │
│  ✓ Brz development                                                              │
│  ✓ Konzistentan dizajn                                                          │
│  ✓ Mali bundle size (purge unused)                                              │
│  ✓ Responsive utilities                                                         │
│  ✓ Dark mode support                                                            │
│                                                                                 │
│  PLUGINS:                                                                       │
│  • @tailwindcss/forms                                                           │
│  • @tailwindcss/typography                                                      │
│                                                                                 │
│  CUSTOM THEME:                                                                  │
│  • Primary: Red (#EF4444) - za panic button                                     │
│  • Secondary: Blue (#3B82F6) - za akcije                                        │
│  • Success: Green (#22C55E) - za resolved                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### PWA (Progressive Web App)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  PWA KONFIGURACIJA                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  LIBRARY: next-pwa                                                              │
│                                                                                 │
│  MANIFEST:                                                                      │
│  {                                                                              │
│    "name": "Patrola - Panic Button",                                            │
│    "short_name": "Patrola",                                                     │
│    "start_url": "/",                                                            │
│    "display": "standalone",                                                     │
│    "background_color": "#ffffff",                                               │
│    "theme_color": "#EF4444",                                                    │
│    "icons": [...]                                                               │
│  }                                                                              │
│                                                                                 │
│  SERVICE WORKER:                                                                │
│  • Cache static assets                                                          │
│  • Offline fallback page                                                        │
│  • Background sync za retry                                                     │
│                                                                                 │
│  CAPABILITIES:                                                                  │
│  ✓ Install na home screen                                                       │
│  ✓ Offline support (basic)                                                      │
│  ✓ Geolocation API                                                              │
│  ✓ Vibration API (za panic feedback)                                            │
│  ✗ Push notifications (koristimo Telegram)                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Backend

### Convex

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CONVEX                                                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ŠTA JE CONVEX:                                                                 │
│  Serverless backend platforma sa real-time bazom podataka                       │
│                                                                                 │
│  ZAŠTO CONVEX:                                                                  │
│  ✓ Zero-config real-time subscriptions                                          │
│  ✓ TypeScript end-to-end (type-safe)                                            │
│  ✓ Serverless (auto-scaling, no ops)                                            │
│  ✓ ACID transakcije                                                             │
│  ✓ Scheduled functions (cron)                                                   │
│  ✓ File storage                                                                 │
│  ✓ Generous free tier                                                           │
│  ✓ Excellent DX (developer experience)                                          │
│                                                                                 │
│  KOMPONENTE:                                                                    │
│  ├── Queries: Read operacije (real-time)                                        │
│  ├── Mutations: Write operacije                                                 │
│  ├── Actions: External API calls                                                │
│  ├── Scheduled: Cron/delayed jobs                                               │
│  └── Internal: Server-only functions                                            │
│                                                                                 │
│  ALTERNATIVA RAZMATRANA:                                                        │
│  • Firebase: Manje type-safe, složenija real-time                               │
│  • Supabase: Dobar, ali PostgreSQL overhead za simple app                       │
│  • PlanetScale + tRPC: Više setup, nema built-in real-time                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Convex Funkcije

```typescript
// Query - real-time, cached
export const getActiveAlarms = query({
  args: { groupId: v.id("groups") },
  handler: async (ctx, { groupId }) => {
    return await ctx.db
      .query("alarms")
      .withIndex("by_group_status", (q) =>
        q.eq("group_id", groupId).eq("status", "ACTIVE")
      )
      .collect();
  },
});

// Mutation - transactional write
export const createAlarm = mutation({
  args: {
    groupId: v.id("groups"),
    message: v.optional(v.string()),
    lat: v.number(),
    lng: v.number(),
  },
  handler: async (ctx, args) => {
    const alarmId = await ctx.db.insert("alarms", {
      ...args,
      status: "ACTIVE",
      created_at: Date.now(),
    });

    // Schedule escalation
    await ctx.scheduler.runAfter(
      90000, // 90 sekundi
      internal.escalation.level1,
      { alarmId }
    );

    return alarmId;
  },
});

// Action - external API
export const sendTelegramAlert = action({
  args: { chatId: v.string(), message: v.string() },
  handler: async (ctx, { chatId, message }) => {
    const response = await fetch(
      `https://api.telegram.org/bot${process.env.TELEGRAM_BOT_TOKEN}/sendMessage`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: chatId,
          text: message,
          parse_mode: "Markdown",
        }),
      }
    );
    return response.json();
  },
});
```

---

## Eksterni Servisi

### Telegram Bot API

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TELEGRAM BOT API                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ULOGA: Pouzdan kanal za notifikacije                                           │
│                                                                                 │
│  ZAŠTO TELEGRAM:                                                                │
│  ✓ Korisnici već imaju Telegram                                                 │
│  ✓ Notifikacije rade pouzdano (čak i battery saver)                             │
│  ✓ Besplatno                                                                    │
│  ✓ Jednostavan API                                                              │
│  ✓ Grupni chat support                                                          │
│                                                                                 │
│  KORIŠĆENI ENDPOINTS:                                                           │
│  • sendMessage - slanje poruke                                                  │
│  • editMessageText - update poruke                                              │
│  • getUpdates - polling za kodove                                               │
│                                                                                 │
│  LIMITI:                                                                        │
│  • 30 poruka/sek po botu                                                        │
│  • 20 poruka/min u istu grupu                                                   │
│                                                                                 │
│  ALTERNATIVA:                                                                   │
│  • Viber: Manje popularan u Srbiji za grupe                                     │
│  • WhatsApp: Business API je skup                                               │
│  • Push notifications: Nepouzdane na Android                                    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Twilio (SMS)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TWILIO SMS                                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ULOGA: SMS verifikacija telefona (pun sistem)                                  │
│                                                                                 │
│  KORIŠĆENJE:                                                                    │
│  • Slanje 6-cifrenog koda                                                       │
│  • Verifikacija pri registraciji                                                │
│  • Opciono: critical alerts                                                     │
│                                                                                 │
│  CENA:                                                                          │
│  • ~$0.05 po SMS-u za Srbiju                                                    │
│  • Free trial: $15 credit                                                       │
│                                                                                 │
│  ALTERNATIVA:                                                                   │
│  • Infobip: Lokalniji, slična cena                                              │
│  • Vonage: Slično Twilio                                                        │
│                                                                                 │
│  NAPOMENA:                                                                      │
│  PoC ne koristi SMS - samo zajednički PIN                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Google Maps / OpenStreetMap

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  MAPE                                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  OPCIJA 1: Google Maps (preporučeno za produkciju)                              │
│  ✓ Tačnije lokacije                                                             │
│  ✓ Street View                                                                  │
│  ✓ Directions API                                                               │
│  ✗ $200/mesec free credit (dovoljan za manje škole)                             │
│                                                                                 │
│  OPCIJA 2: Leaflet + OpenStreetMap (PoC)                                        │
│  ✓ Potpuno besplatno                                                            │
│  ✓ Open source                                                                  │
│  ✓ Dovoljno za prikaz lokacije                                                  │
│  ✗ Manje tačno                                                                  │
│                                                                                 │
│  KORIŠĆENJE:                                                                    │
│  • Prikaz lokacije alarma na mapi                                               │
│  • "Otvori u Google Maps" link za navigaciju                                    │
│  • Geofence prikaz (zona škole)                                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Hosting

### Vercel

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  VERCEL                                                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ULOGA: Frontend hosting + CDN                                                  │
│                                                                                 │
│  ZAŠTO VERCEL:                                                                  │
│  ✓ Native Next.js support                                                       │
│  ✓ Global CDN (Edge network)                                                    │
│  ✓ Automatic HTTPS                                                              │
│  ✓ Preview deployments (PR previews)                                            │
│  ✓ Generous free tier                                                           │
│  ✓ Zero-config deployment                                                       │
│                                                                                 │
│  FREE TIER UKLJUČUJE:                                                           │
│  • 100GB bandwidth/mesec                                                        │
│  • Unlimited deployments                                                        │
│  • SSL certificates                                                             │
│  • Custom domain                                                                │
│                                                                                 │
│  ALTERNATIVA:                                                                   │
│  • Cloudflare Pages: Slično, možda brži                                         │
│  • Netlify: Dobar, ali Vercel je bolji za Next.js                               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Convex Cloud

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CONVEX CLOUD                                                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ULOGA: Backend hosting                                                         │
│                                                                                 │
│  INCLUDED:                                                                      │
│  ✓ Database hosting                                                             │
│  ✓ Function execution                                                           │
│  ✓ Real-time infrastructure                                                     │
│  ✓ File storage                                                                 │
│  ✓ Scheduled functions                                                          │
│                                                                                 │
│  FREE TIER:                                                                     │
│  • 1M function calls/mesec                                                      │
│  • 1GB storage                                                                  │
│  • 1GB bandwidth                                                                │
│                                                                                 │
│  PRO TIER (~$25/mesec):                                                         │
│  • 10M function calls                                                           │
│  • 10GB storage                                                                 │
│  • Point-in-time recovery                                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Development Tools

### TypeScript

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TYPESCRIPT                                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERZIJA: 5.x                                                                   │
│                                                                                 │
│  STRICT MODE: Da                                                                │
│                                                                                 │
│  PREDNOSTI:                                                                     │
│  ✓ End-to-end type safety (Convex generiše tipove)                              │
│  ✓ IDE autocomplete                                                             │
│  ✓ Catch errors at compile time                                                 │
│  ✓ Self-documenting code                                                        │
│                                                                                 │
│  CONVEX TYPE GENERATION:                                                        │
│  npx convex dev  →  generiše tipove iz schema                                   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### ESLint + Prettier

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CODE QUALITY                                                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ESLINT CONFIG:                                                                 │
│  • next/core-web-vitals                                                         │
│  • @typescript-eslint                                                           │
│  • prettier                                                                     │
│                                                                                 │
│  PRETTIER CONFIG:                                                               │
│  {                                                                              │
│    "semi": true,                                                                │
│    "singleQuote": false,                                                        │
│    "tabWidth": 2,                                                               │
│    "trailingComma": "es5"                                                       │
│  }                                                                              │
│                                                                                 │
│  PRE-COMMIT HOOKS (Husky):                                                      │
│  • lint-staged                                                                  │
│  • prettier --check                                                             │
│  • tsc --noEmit                                                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Package Dependencies

### package.json (ključne)

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "convex": "^1.0.0",

    "tailwindcss": "^3.4.0",
    "next-pwa": "^5.6.0",

    "leaflet": "^1.9.0",
    "react-leaflet": "^4.2.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/react": "^18.0.0",
    "@types/node": "^20.0.0",

    "eslint": "^8.0.0",
    "eslint-config-next": "^14.0.0",
    "prettier": "^3.0.0",

    "husky": "^8.0.0",
    "lint-staged": "^15.0.0"
  }
}
```

---

## Version Requirements

| Tehnologija | Minimalna verzija | Preporučena |
|-------------|-------------------|-------------|
| Node.js | 18.x | 20.x LTS |
| npm | 9.x | 10.x |
| Next.js | 14.0 | 14.x latest |
| React | 18.2 | 18.x latest |
| TypeScript | 5.0 | 5.x latest |
| Convex | 1.0 | latest |

---

*Dokument kreiran: Januar 2026*
