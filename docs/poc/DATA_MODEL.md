# PoC - Data Model

## Pregled

PoC koristi **minimalan data model** sa samo jednom tabelom: `alarms`.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POC DATA MODEL                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          ┌─────────────┐                                    │
│                          │   alarms    │                                    │
│                          └─────────────┘                                    │
│                                                                             │
│  To je sve. Nema users, nema groups, nema memberships.                      │
│  Maksimalna jednostavnost za brzi PoC.                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Convex Schema

```typescript
// convex/schema.ts

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({

  alarms: defineTable({
    // Ko/šta
    sender_name: v.string(),           // "Marko" ili "Anonimno"
    message: v.optional(v.string()),   // "Prate me"

    // Gde
    lat: v.number(),                   // GPS latitude
    lng: v.number(),                   // GPS longitude

    // Status
    status: v.union(
      v.literal("ACTIVE"),             // Aktivan, čeka se reakcija
      v.literal("TAKEN"),              // Neko preuzeo
      v.literal("RESOLVED"),           // Rešeno
      v.literal("CANCELLED"),          // Otkazano od pošiljaoca
      v.literal("FALSE_ALARM")         // Označeno kao lažni
    ),

    // Ko je preuzeo
    taken_by: v.optional(v.string()),  // "Petar"
    taken_at: v.optional(v.number()),  // timestamp

    // Vremena
    created_at: v.number(),            // timestamp kreiranja
    resolved_at: v.optional(v.number()), // timestamp razrešenja
  }),

});
```

---

## Tabela: alarms

### Opis polja

| Polje | Tip | Obavezno | Opis |
|-------|-----|----------|------|
| `sender_name` | string | Da | Ime pošiljaoca ("Marko" ili "Anonimno") |
| `message` | string | Ne | Opis situacije ("Prate me") |
| `lat` | number | Da | GPS latitude |
| `lng` | number | Da | GPS longitude |
| `status` | enum | Da | Status alarma (vidi ispod) |
| `taken_by` | string | Ne | Ko je preuzeo alarm |
| `taken_at` | number | Ne | Kada je preuzeto (timestamp) |
| `created_at` | number | Da | Kada je kreiran (timestamp) |
| `resolved_at` | number | Ne | Kada je razrešen (timestamp) |

### Status enum

| Vrednost | Opis |
|----------|------|
| `ACTIVE` | Alarm je aktivan, čeka se reakcija |
| `TAKEN` | Neko je preuzeo alarm |
| `RESOLVED` | Situacija je rešena |
| `CANCELLED` | Pošiljalac je otkazao alarm |
| `FALSE_ALARM` | Označeno kao lažni alarm |

### State machine

```
                    ┌───────────────────┐
                    │                   │
                    ▼                   │
┌────────┐     ┌────────┐     ┌────────────┐
│ ACTIVE │────►│ TAKEN  │────►│  RESOLVED  │
└────────┘     └────────┘     └────────────┘
    │              │
    │              │
    ▼              ▼
┌────────────┐  ┌────────────┐
│ CANCELLED  │  │FALSE_ALARM │
└────────────┘  └────────────┘

Dozvoljeni prelazi:
• ACTIVE → TAKEN (neko preuzeo)
• ACTIVE → CANCELLED (pošiljalac otkazao)
• TAKEN → RESOLVED (situacija rešena)
• TAKEN → FALSE_ALARM (lažni alarm)
```

---

## Primer podataka

### Aktivan alarm

```json
{
  "_id": "k1234567890",
  "sender_name": "Marko",
  "message": "Prate me",
  "lat": 44.8125,
  "lng": 20.4612,
  "status": "ACTIVE",
  "created_at": 1705671600000
}
```

### Preuzet alarm

```json
{
  "_id": "k1234567890",
  "sender_name": "Marko",
  "message": "Prate me",
  "lat": 44.8125,
  "lng": 20.4612,
  "status": "TAKEN",
  "taken_by": "Petar",
  "taken_at": 1705671647000,
  "created_at": 1705671600000
}
```

### Razrešen alarm

```json
{
  "_id": "k1234567890",
  "sender_name": "Marko",
  "message": "Prate me",
  "lat": 44.8125,
  "lng": 20.4612,
  "status": "RESOLVED",
  "taken_by": "Petar",
  "taken_at": 1705671647000,
  "created_at": 1705671600000,
  "resolved_at": 1705671900000
}
```

---

## Convex Funkcije

### Queries

```typescript
// convex/alarms.ts

import { query } from "./_generated/server";

// Dohvati aktivne alarme (ACTIVE ili TAKEN)
export const getActiveAlarms = query({
  handler: async (ctx) => {
    return await ctx.db
      .query("alarms")
      .filter((q) =>
        q.or(
          q.eq(q.field("status"), "ACTIVE"),
          q.eq(q.field("status"), "TAKEN")
        )
      )
      .order("desc")
      .collect();
  },
});
```

### Mutations

```typescript
// convex/alarms.ts

import { mutation } from "./_generated/server";
import { v } from "convex/values";

// Kreiraj alarm
export const createAlarm = mutation({
  args: {
    sender_name: v.string(),
    message: v.optional(v.string()),
    lat: v.number(),
    lng: v.number(),
  },
  handler: async (ctx, args) => {
    const alarm = await ctx.db.insert("alarms", {
      ...args,
      status: "ACTIVE",
      created_at: Date.now(),
    });

    // Zakaži slanje u Telegram
    await ctx.scheduler.runAfter(0, "telegram:sendAlert", {
      alarm_id: alarm,
      lat: args.lat,
      lng: args.lng,
      message: args.message,
      sender_name: args.sender_name,
    });

    return alarm;
  },
});

// Preuzmi alarm
export const takeAlarm = mutation({
  args: {
    alarm_id: v.id("alarms"),
    taken_by: v.string(),
  },
  handler: async (ctx, { alarm_id, taken_by }) => {
    const alarm = await ctx.db.get(alarm_id);
    if (!alarm || alarm.status !== "ACTIVE") {
      throw new Error("Alarm nije aktivan");
    }

    await ctx.db.patch(alarm_id, {
      status: "TAKEN",
      taken_by,
      taken_at: Date.now(),
    });

    // Telegram update
    await ctx.scheduler.runAfter(0, "telegram:sendUpdate", {
      message: `✅ ${taken_by} je preuzeo alarm`,
    });
  },
});

// Razreši alarm
export const resolveAlarm = mutation({
  args: {
    alarm_id: v.id("alarms"),
    resolution: v.union(
      v.literal("RESOLVED"),
      v.literal("FALSE_ALARM"),
      v.literal("CANCELLED")
    ),
  },
  handler: async (ctx, { alarm_id, resolution }) => {
    await ctx.db.patch(alarm_id, {
      status: resolution,
      resolved_at: Date.now(),
    });

    const emoji = resolution === "RESOLVED" ? "✅" :
                  resolution === "FALSE_ALARM" ? "⚠️" : "🚫";

    await ctx.scheduler.runAfter(0, "telegram:sendUpdate", {
      message: `${emoji} Alarm zatvoren: ${resolution}`,
    });
  },
});
```

---

## Telegram Action

```typescript
// convex/telegram.ts

import { action } from "./_generated/server";
import { v } from "convex/values";

const TELEGRAM_BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN!;
const TELEGRAM_CHAT_ID = process.env.TELEGRAM_CHAT_ID!;

export const sendAlert = action({
  args: {
    alarm_id: v.id("alarms"),
    lat: v.number(),
    lng: v.number(),
    message: v.optional(v.string()),
    sender_name: v.string(),
  },
  handler: async (ctx, { lat, lng, message, sender_name }) => {
    const mapsUrl = `https://maps.google.com/?q=${lat},${lng}`;
    const appUrl = process.env.APP_URL || "https://patrola-kovacic.vercel.app";

    const text = `🚨 *ALARM*

📍 Lokacija: ${mapsUrl}
💬 ${message || "(bez opisa)"}
👤 Od: ${sender_name}

👆 Otvori app: ${appUrl}`;

    await fetch(
      `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: TELEGRAM_CHAT_ID,
          text,
          parse_mode: "Markdown",
        }),
      }
    );
  },
});

export const sendUpdate = action({
  args: { message: v.string() },
  handler: async (ctx, { message }) => {
    await fetch(
      `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: TELEGRAM_CHAT_ID,
          text: message,
        }),
      }
    );
  },
});
```

---

## Environment Variables

```bash
# .env.local (za lokalni development)
# Ove vrednosti se NE COMMITUJU!

# Telegram
TELEGRAM_BOT_TOKEN=123456789:ABC-DEF_xyz...
TELEGRAM_CHAT_ID=-100123456789

# App URL (za link u Telegram porukama)
APP_URL=https://patrola-kovacic.vercel.app

# PIN za pristup (opciono - može i hardcoded)
NEXT_PUBLIC_SCHOOL_PIN=kovacic2025
NEXT_PUBLIC_SCHOOL_NAME=OŠ Ivan Goran Kovačić
```

---

## Šta PoC data model NE SADRŽI

| Tabela | Razlog izostavljanja |
|--------|----------------------|
| `users` | Nema pravih korisnika, samo imena |
| `groups` | Samo jedna škola (hardcoded) |
| `memberships` | Nema veze user-group |
| `shifts` | Nema smena |
| `alarm_responses` | Pojednostavljen "taken_by" |
| `telegram_link_codes` | Hardcoded CHAT_ID |
| `sms_codes` | Nema SMS verifikacije |
| `audit_log` | Nema logovanja |

---

## Migracija ka MVP

Kada se pređe na MVP, data model se proširuje:

```
PoC:                         MVP:
─────────                    ────────
alarms ──────────────────►   alarms (prošireno)
                             + users
                             + groups
                             + memberships
                             + shifts
                             + sms_codes
                             + telegram_link_codes
```

Migracija se radi tako što se:
1. Dodaju nove tabele
2. Postojeći `alarms` podaci se zadržavaju
3. Dodaju se nove kolone (sa default vrednostima)

---

*Dokument kreiran: Januar 2026*
