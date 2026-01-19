# Deployment Arhitektura

## Pregled

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT ARHITEKTURA                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              INTERNET                                           │
│                                  │                                              │
│                                  ▼                                              │
│                         ┌───────────────┐                                       │
│                         │  CLOUDFLARE   │                                       │
│                         │    (DNS)      │                                       │
│                         └───────┬───────┘                                       │
│                                 │                                               │
│                    ┌────────────┴────────────┐                                  │
│                    │                         │                                  │
│                    ▼                         ▼                                  │
│           ┌───────────────┐         ┌───────────────┐                           │
│           │    VERCEL     │         │    CONVEX     │                           │
│           │   (Frontend)  │◄───────►│   (Backend)   │                           │
│           └───────────────┘         └───────┬───────┘                           │
│                                             │                                   │
│                              ┌──────────────┼──────────────┐                    │
│                              │              │              │                    │
│                              ▼              ▼              ▼                    │
│                       ┌──────────┐   ┌──────────┐   ┌──────────┐                │
│                       │ TELEGRAM │   │  TWILIO  │   │  SENTRY  │                │
│                       │   API    │   │  (SMS)   │   │ (Errors) │                │
│                       └──────────┘   └──────────┘   └──────────┘                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Komponente

### 1. DNS (Cloudflare)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CLOUDFLARE DNS                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DOMENA: patrola.rs (primer)                                                    │
│                                                                                 │
│  DNS RECORDS:                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  TYPE    NAME           VALUE                         PROXY             │    │
│  │  ─────────────────────────────────────────────────────────────────────  │    │
│  │  CNAME   @              cname.vercel-dns.com          ✅ Proxied        │    │
│  │  CNAME   www            cname.vercel-dns.com          ✅ Proxied        │    │
│  │  TXT     @              v=spf1 include:_spf.mx...     -                 │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  CLOUDFLARE SETTINGS:                                                           │
│  • SSL: Full (strict)                                                           │
│  • Always Use HTTPS: On                                                         │
│  • Auto Minify: JS, CSS, HTML                                                   │
│  • Brotli: On                                                                   │
│                                                                                 │
│  ZAŠTO CLOUDFLARE:                                                              │
│  ✓ Besplatan DNS                                                                │
│  ✓ DDoS zaštita                                                                 │
│  ✓ Edge caching                                                                 │
│  ✓ SSL certificates                                                             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. Frontend (Vercel)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  VERCEL DEPLOYMENT                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PROJECT SETUP:                                                                 │
│  1. Import from GitHub repo                                                     │
│  2. Framework: Next.js (auto-detected)                                          │
│  3. Build command: next build                                                   │
│  4. Output: .next folder                                                        │
│                                                                                 │
│  ENVIRONMENT VARIABLES:                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  NEXT_PUBLIC_CONVEX_URL=https://xxx.convex.cloud                        │    │
│  │  NEXT_PUBLIC_APP_URL=https://patrola.rs                                 │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  DEPLOYMENTS:                                                                   │
│  • Production: main branch → patrola.rs                                         │
│  • Preview: PR branches → xxx.vercel.app                                        │
│  • Development: localhost:3000                                                  │
│                                                                                 │
│  BUILD SETTINGS:                                                                │
│  • Node.js: 20.x                                                                │
│  • Install: npm ci                                                              │
│  • Build: npm run build                                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 3. Backend (Convex)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CONVEX DEPLOYMENT                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PROJECT SETUP:                                                                 │
│  1. npx convex init                                                             │
│  2. Povezivanje sa Convex dashboard                                             │
│  3. npx convex deploy                                                           │
│                                                                                 │
│  ENVIRONMENTS:                                                                  │
│  • Production: https://xxx-prod.convex.cloud                                    │
│  • Development: https://xxx-dev.convex.cloud                                    │
│                                                                                 │
│  ENVIRONMENT VARIABLES (u Convex dashboard):                                    │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  TELEGRAM_BOT_TOKEN=123456:ABC...                                       │    │
│  │  TWILIO_ACCOUNT_SID=ACxxx                                               │    │
│  │  TWILIO_AUTH_TOKEN=xxx                                                  │    │
│  │  TWILIO_PHONE_NUMBER=+1234567890                                        │    │
│  │  APP_URL=https://patrola.rs                                             │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  DEPLOYMENT FLOW:                                                               │
│  1. Lokalne promene u convex/ folderu                                           │
│  2. npx convex deploy (push schema + functions)                                 │
│  3. Automatski type generation                                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/deploy.yml

name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # ═══════════════════════════════════════════════════════════════════════════
  # LINT & TYPE CHECK
  # ═══════════════════════════════════════════════════════════════════════════
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck

  # ═══════════════════════════════════════════════════════════════════════════
  # CONVEX DEPLOYMENT
  # ═══════════════════════════════════════════════════════════════════════════
  deploy-convex:
    needs: lint
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Deploy Convex
        run: npx convex deploy
        env:
          CONVEX_DEPLOY_KEY: ${{ secrets.CONVEX_DEPLOY_KEY }}

  # ═══════════════════════════════════════════════════════════════════════════
  # VERCEL DEPLOYMENT (automatic via Vercel GitHub integration)
  # ═══════════════════════════════════════════════════════════════════════════
  # Vercel automatski deploya kada se push-a na main
  # Ovde samo triggerujemo rebuild ako je potrebno

  notify:
    needs: [deploy-convex]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Notify Telegram
        run: |
          curl -X POST \
            "https://api.telegram.org/bot${{ secrets.TELEGRAM_BOT_TOKEN }}/sendMessage" \
            -d "chat_id=${{ secrets.TELEGRAM_ADMIN_CHAT_ID }}" \
            -d "text=✅ Deployment successful: ${{ github.sha }}"
```

---

## Environment Configuration

### Development

```bash
# .env.local (NE COMMITOVATI!)

# Convex
NEXT_PUBLIC_CONVEX_URL=https://xxx-dev.convex.cloud

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Debug
NEXT_PUBLIC_DEBUG=true
```

### Production

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  PRODUCTION ENVIRONMENT                                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERCEL ENVIRONMENT VARIABLES:                                                  │
│  ├── NEXT_PUBLIC_CONVEX_URL    (Production Convex URL)                          │
│  ├── NEXT_PUBLIC_APP_URL       (https://patrola.rs)                             │
│  └── NEXT_PUBLIC_DEBUG         (false)                                          │
│                                                                                 │
│  CONVEX ENVIRONMENT VARIABLES:                                                  │
│  ├── TELEGRAM_BOT_TOKEN        (Bot token from @BotFather)                      │
│  ├── TWILIO_ACCOUNT_SID        (Twilio credentials)                             │
│  ├── TWILIO_AUTH_TOKEN         (Twilio credentials)                             │
│  ├── TWILIO_PHONE_NUMBER       (Twilio phone number)                            │
│  └── APP_URL                   (https://patrola.rs)                             │
│                                                                                 │
│  SECRETS (NE ČUVATI U KODU):                                                    │
│  • Telegram bot token                                                           │
│  • Twilio credentials                                                           │
│  • Convex deploy key                                                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Monitoring

### Error Tracking (Sentry)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  SENTRY SETUP                                                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  FRONTEND:                                                                      │
│  • @sentry/nextjs package                                                       │
│  • Automatic error capture                                                      │
│  • Performance monitoring                                                       │
│  • Source maps upload                                                           │
│                                                                                 │
│  BACKEND (Convex):                                                              │
│  • Manual error reporting u action-ima                                          │
│  • try/catch sa Sentry.captureException                                         │
│                                                                                 │
│  ALERTS:                                                                        │
│  • Slack/Email za critical errors                                               │
│  • Threshold: >5 errors/min                                                     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Health Checks

```typescript
// convex/health.ts

export const healthCheck = query({
  args: {},
  handler: async (ctx) => {
    // Test database
    const testQuery = await ctx.db.query("groups").first();

    return {
      status: "healthy",
      timestamp: Date.now(),
      database: testQuery !== undefined ? "connected" : "empty",
    };
  },
});
```

### Uptime Monitoring

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  UPTIME MONITORING                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  SERVICE: UptimeRobot (besplatan) ili Better Uptime                             │
│                                                                                 │
│  MONITORS:                                                                      │
│  • https://patrola.rs - Frontend availability                                   │
│  • Convex health endpoint - Backend availability                                │
│  • Telegram bot - Webhook response                                              │
│                                                                                 │
│  ALERTS:                                                                        │
│  • Telegram poruka adminu                                                       │
│  • Email backup                                                                 │
│                                                                                 │
│  CHECK INTERVAL: 5 minuta                                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Scaling

### Horizontal Scaling

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  SCALING STRATEGY                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERCEL:                                                                        │
│  • Automatski skalira                                                           │
│  • Edge functions za low latency                                                │
│  • CDN za static assets                                                         │
│                                                                                 │
│  CONVEX:                                                                        │
│  • Serverless - automatski skalira                                              │
│  • Nema manual scaling                                                          │
│  • Samo plaćaš više ako koristiš više                                           │
│                                                                                 │
│  TELEGRAM:                                                                      │
│  • 30 poruka/sek limit po botu                                                  │
│  • Za veći scale: queue sistem                                                  │
│  • Ili: jedan bot po školi                                                      │
│                                                                                 │
│  BOTTLENECKS:                                                                   │
│  1. Telegram rate limit → Queue                                                 │
│  2. Convex function timeout (60s) → Batch processing                            │
│  3. SMS costs → Budget alerts                                                   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Backup & Recovery

### Database Backup

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  BACKUP STRATEGY                                                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CONVEX BUILT-IN:                                                               │
│  • Automatic snapshots (Pro plan)                                               │
│  • Point-in-time recovery                                                       │
│                                                                                 │
│  MANUAL BACKUP:                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  # Export sve podatke                                                   │    │
│  │  npx convex export --path ./backup-$(date +%Y%m%d)                      │    │
│  │                                                                         │    │
│  │  # Upload to S3 (optional)                                              │    │
│  │  aws s3 cp ./backup-* s3://patrola-backups/                             │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  SCHEDULE:                                                                      │
│  • Daily: Automatic (Convex Pro)                                                │
│  • Weekly: Manual export to S3                                                  │
│  • Monthly: Verified restore test                                               │
│                                                                                 │
│  RESTORE:                                                                       │
│  npx convex import --path ./backup-20250119                                     │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Disaster Recovery

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  DISASTER RECOVERY PLAN                                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  SCENARIO 1: Frontend down                                                      │
│  ├── Vercel status check: status.vercel.com                                     │
│  ├── Alternative: Deploy to Cloudflare Pages                                    │
│  └── DNS switch: ~5 min TTL                                                     │
│                                                                                 │
│  SCENARIO 2: Convex down                                                        │
│  ├── Status: status.convex.dev                                                  │
│  ├── Fallback: Static "maintenance" page                                        │
│  └── Telegram still works (cached chat_id)                                      │
│                                                                                 │
│  SCENARIO 3: Telegram API down                                                  │
│  ├── Alarmi i dalje rade u app-u                                                │
│  ├── Notifikacije ne stižu                                                      │
│  └── Fallback: SMS (ako je konfigurisano)                                       │
│                                                                                 │
│  SCENARIO 4: Data loss                                                          │
│  ├── Restore from backup                                                        │
│  ├── Notify users about potential data loss                                     │
│  └── Audit what was lost                                                        │
│                                                                                 │
│  RTO (Recovery Time Objective): < 1 sat                                         │
│  RPO (Recovery Point Objective): < 24 sata                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Security Checklist

### Pre-deployment

- [ ] HTTPS enforced (Vercel/Cloudflare)
- [ ] Environment variables set (not in code)
- [ ] Secrets in secure storage
- [ ] CORS configured correctly
- [ ] Rate limiting enabled
- [ ] Input validation on all endpoints

### Post-deployment

- [ ] SSL certificate valid
- [ ] Health check passing
- [ ] Error tracking working
- [ ] Backup verified
- [ ] Monitoring alerts configured

---

## Deployment Commands

```bash
# ═══════════════════════════════════════════════════════════════════════════════
# DEVELOPMENT
# ═══════════════════════════════════════════════════════════════════════════════

# Start local dev
npm run dev                  # Next.js dev server
npx convex dev              # Convex dev (watches for changes)

# ═══════════════════════════════════════════════════════════════════════════════
# PRODUCTION DEPLOYMENT
# ═══════════════════════════════════════════════════════════════════════════════

# Deploy Convex (backend)
npx convex deploy

# Deploy Vercel (frontend) - automatic on git push
# Or manual:
vercel --prod

# ═══════════════════════════════════════════════════════════════════════════════
# MAINTENANCE
# ═══════════════════════════════════════════════════════════════════════════════

# Export data
npx convex export --path ./backup

# Import data
npx convex import --path ./backup

# View logs
npx convex logs

# Run migration
npx convex run migrations:backfillNewField
```

---

*Dokument kreiran: Januar 2025*
