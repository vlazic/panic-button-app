# PoC - Implementacija

## Struktura projekta

```
panic-button-poc/
├── convex/
│   ├── _generated/           # Auto-generisano od Convex-a
│   ├── schema.ts             # Data model
│   ├── alarms.ts             # Queries i mutations za alarme
│   └── telegram.ts           # Telegram integracija
│
├── src/
│   ├── app/
│   │   ├── page.tsx          # Login stranica (PIN)
│   │   ├── dashboard/
│   │   │   └── page.tsx      # Glavni ekran (panic + lista)
│   │   └── layout.tsx        # Root layout
│   │
│   ├── components/
│   │   ├── PanicButton.tsx   # Panic dugme sa hold logikom
│   │   ├── AlarmCard.tsx     # Prikaz jednog alarma
│   │   ├── AlarmForm.tsx     # Forma za slanje (ime, opis)
│   │   └── AlarmList.tsx     # Lista aktivnih alarma
│   │
│   └── lib/
│       └── auth.ts           # PIN provera (localStorage)
│
├── public/
│   └── manifest.json         # PWA manifest (opciono)
│
├── .env.local                # Environment varijable (NE COMMITOVATI)
├── package.json
├── next.config.js
├── tailwind.config.js
└── tsconfig.json
```

---

## Ključni fajlovi

### 1. Convex Schema

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
  }),
});
```

### 2. Alarms API

```typescript
// convex/alarms.ts

import { mutation, query } from "./_generated/server";
import { v } from "convex/values";

// ════════════════════════════════════════════════════════════
// QUERIES
// ════════════════════════════════════════════════════════════

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

// ════════════════════════════════════════════════════════════
// MUTATIONS
// ════════════════════════════════════════════════════════════

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

    // Pošalji u Telegram
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

export const takeAlarm = mutation({
  args: {
    alarm_id: v.id("alarms"),
    taken_by: v.string(),
  },
  handler: async (ctx, { alarm_id, taken_by }) => {
    const alarm = await ctx.db.get(alarm_id);

    if (!alarm) {
      throw new Error("Alarm ne postoji");
    }

    if (alarm.status !== "ACTIVE") {
      throw new Error("Alarm nije aktivan");
    }

    await ctx.db.patch(alarm_id, {
      status: "TAKEN",
      taken_by,
      taken_at: Date.now(),
    });

    await ctx.scheduler.runAfter(0, "telegram:sendUpdate", {
      message: `✅ ${taken_by} je preuzeo alarm`,
    });
  },
});

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
    const alarm = await ctx.db.get(alarm_id);

    if (!alarm) {
      throw new Error("Alarm ne postoji");
    }

    await ctx.db.patch(alarm_id, {
      status: resolution,
      resolved_at: Date.now(),
    });

    const emoji =
      resolution === "RESOLVED"
        ? "✅"
        : resolution === "FALSE_ALARM"
        ? "⚠️"
        : "🚫";

    await ctx.scheduler.runAfter(0, "telegram:sendUpdate", {
      message: `${emoji} Alarm zatvoren: ${resolution}`,
    });
  },
});
```

### 3. Telegram Integracija

```typescript
// convex/telegram.ts

import { action } from "./_generated/server";
import { v } from "convex/values";

const TELEGRAM_BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN!;
const TELEGRAM_CHAT_ID = process.env.TELEGRAM_CHAT_ID!;
const APP_URL = process.env.APP_URL || "https://patrola.vercel.app";

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

    const text = `🚨 *ALARM*

📍 Lokacija: ${mapsUrl}
💬 ${message || "(bez opisa)"}
👤 Od: ${sender_name}

👆 Otvori app: ${APP_URL}`;

    try {
      const response = await fetch(
        `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            chat_id: TELEGRAM_CHAT_ID,
            text,
            parse_mode: "Markdown",
            disable_web_page_preview: false,
          }),
        }
      );

      if (!response.ok) {
        console.error("Telegram error:", await response.text());
      }
    } catch (error) {
      console.error("Failed to send Telegram message:", error);
    }
  },
});

export const sendUpdate = action({
  args: { message: v.string() },
  handler: async (ctx, { message }) => {
    try {
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
    } catch (error) {
      console.error("Failed to send Telegram update:", error);
    }
  },
});
```

### 4. PIN Auth Helper

```typescript
// src/lib/auth.ts

const PIN_KEY = "patrola_pin";
const NAME_KEY = "patrola_name";
const CORRECT_PIN = process.env.NEXT_PUBLIC_SCHOOL_PIN || "kovacic2025";

export function checkPin(pin: string): boolean {
  return pin.toLowerCase() === CORRECT_PIN.toLowerCase();
}

export function savePin(pin: string): void {
  if (typeof window !== "undefined") {
    localStorage.setItem(PIN_KEY, pin);
  }
}

export function getSavedPin(): string | null {
  if (typeof window !== "undefined") {
    return localStorage.getItem(PIN_KEY);
  }
  return null;
}

export function isLoggedIn(): boolean {
  const savedPin = getSavedPin();
  return savedPin !== null && checkPin(savedPin);
}

export function logout(): void {
  if (typeof window !== "undefined") {
    localStorage.removeItem(PIN_KEY);
  }
}

export function saveName(name: string): void {
  if (typeof window !== "undefined") {
    localStorage.setItem(NAME_KEY, name);
  }
}

export function getSavedName(): string | null {
  if (typeof window !== "undefined") {
    return localStorage.getItem(NAME_KEY);
  }
  return null;
}
```

### 5. Panic Button Component

```tsx
// src/components/PanicButton.tsx

"use client";

import { useState, useRef } from "react";

interface PanicButtonProps {
  onActivate: () => void;
  holdDuration?: number; // ms, default 3000
}

export function PanicButton({
  onActivate,
  holdDuration = 3000,
}: PanicButtonProps) {
  const [isHolding, setIsHolding] = useState(false);
  const [progress, setProgress] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  const startTimeRef = useRef<number>(0);

  const startHold = () => {
    setIsHolding(true);
    setProgress(0);
    startTimeRef.current = Date.now();

    intervalRef.current = setInterval(() => {
      const elapsed = Date.now() - startTimeRef.current;
      const newProgress = Math.min((elapsed / holdDuration) * 100, 100);
      setProgress(newProgress);

      if (elapsed >= holdDuration) {
        stopHold();
        onActivate();
      }
    }, 50);
  };

  const stopHold = () => {
    setIsHolding(false);
    setProgress(0);
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  return (
    <div className="flex flex-col items-center gap-4">
      <button
        className={`
          w-48 h-48 rounded-full
          flex items-center justify-center
          text-white font-bold text-xl
          transition-all duration-200
          ${isHolding ? "bg-red-700 scale-95" : "bg-red-500 hover:bg-red-600"}
        `}
        onMouseDown={startHold}
        onMouseUp={stopHold}
        onMouseLeave={stopHold}
        onTouchStart={startHold}
        onTouchEnd={stopHold}
      >
        {isHolding ? (
          <div className="text-center">
            <div className="text-4xl mb-2">🔴</div>
            <div>{Math.floor(progress)}%</div>
          </div>
        ) : (
          <div className="text-center">
            <div className="text-4xl mb-2">🔴</div>
            <div>DRŽI ZA ALARM</div>
            <div className="text-sm">(3 sekunde)</div>
          </div>
        )}
      </button>

      {isHolding && (
        <div className="w-48 h-2 bg-gray-200 rounded-full overflow-hidden">
          <div
            className="h-full bg-red-500 transition-all duration-50"
            style={{ width: `${progress}%` }}
          />
        </div>
      )}

      {isHolding && (
        <p className="text-gray-500 text-sm">Pusti za otkazivanje</p>
      )}
    </div>
  );
}
```

### 6. Alarm Card Component

```tsx
// src/components/AlarmCard.tsx

"use client";

import { useMutation } from "convex/react";
import { api } from "../../convex/_generated/api";
import { Id } from "../../convex/_generated/dataModel";
import { useState } from "react";
import { getSavedName, saveName } from "@/lib/auth";

interface Alarm {
  _id: Id<"alarms">;
  sender_name: string;
  message?: string;
  lat: number;
  lng: number;
  status: "ACTIVE" | "TAKEN" | "RESOLVED" | "CANCELLED" | "FALSE_ALARM";
  taken_by?: string;
  taken_at?: number;
  created_at: number;
}

interface AlarmCardProps {
  alarm: Alarm;
}

export function AlarmCard({ alarm }: AlarmCardProps) {
  const [showTakeForm, setShowTakeForm] = useState(false);
  const [name, setName] = useState(getSavedName() || "");

  const takeAlarm = useMutation(api.alarms.takeAlarm);
  const resolveAlarm = useMutation(api.alarms.resolveAlarm);

  const mapsUrl = `https://maps.google.com/?q=${alarm.lat},${alarm.lng}`;
  const timeAgo = getTimeAgo(alarm.created_at);

  const handleTake = async () => {
    if (!name.trim()) return;
    saveName(name);
    await takeAlarm({ alarm_id: alarm._id, taken_by: name });
    setShowTakeForm(false);
  };

  const handleResolve = async (
    resolution: "RESOLVED" | "FALSE_ALARM" | "CANCELLED"
  ) => {
    await resolveAlarm({ alarm_id: alarm._id, resolution });
  };

  return (
    <div className="border rounded-lg p-4 bg-white shadow-sm">
      {alarm.status === "ACTIVE" && (
        <div className="text-red-500 font-bold mb-2">🚨 AKTIVAN ALARM</div>
      )}

      {alarm.status === "TAKEN" && (
        <div className="text-green-600 font-bold mb-2">
          ✅ {alarm.taken_by} je preuzeo
        </div>
      )}

      <div className="space-y-1 text-sm">
        <p>
          📍{" "}
          <a
            href={mapsUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="text-blue-500 underline"
          >
            Lokacija na mapi
          </a>
        </p>
        {alarm.message && <p>💬 {alarm.message}</p>}
        <p>👤 Od: {alarm.sender_name}</p>
        <p>🕐 {timeAgo}</p>
      </div>

      {alarm.status === "ACTIVE" && !showTakeForm && (
        <div className="mt-4 flex gap-2">
          <a
            href={mapsUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="px-4 py-2 bg-blue-500 text-white rounded"
          >
            📍 MAPA
          </a>
          <button
            onClick={() => setShowTakeForm(true)}
            className="px-4 py-2 bg-green-500 text-white rounded"
          >
            🏃 PREUZIMAM
          </button>
        </div>
      )}

      {showTakeForm && (
        <div className="mt-4 space-y-2">
          <input
            type="text"
            placeholder="Tvoje ime"
            value={name}
            onChange={(e) => setName(e.target.value)}
            className="w-full px-3 py-2 border rounded"
          />
          <div className="flex gap-2">
            <button
              onClick={handleTake}
              disabled={!name.trim()}
              className="px-4 py-2 bg-green-500 text-white rounded disabled:opacity-50"
            >
              POTVRDI
            </button>
            <button
              onClick={() => setShowTakeForm(false)}
              className="px-4 py-2 bg-gray-300 rounded"
            >
              ODUSTANI
            </button>
          </div>
        </div>
      )}

      {alarm.status === "TAKEN" && (
        <div className="mt-4 flex gap-2">
          <button
            onClick={() => handleResolve("RESOLVED")}
            className="px-4 py-2 bg-green-500 text-white rounded"
          >
            ✅ REŠENO
          </button>
          <button
            onClick={() => handleResolve("FALSE_ALARM")}
            className="px-4 py-2 bg-yellow-500 text-white rounded"
          >
            ⚠️ LAŽNI
          </button>
        </div>
      )}
    </div>
  );
}

function getTimeAgo(timestamp: number): string {
  const seconds = Math.floor((Date.now() - timestamp) / 1000);

  if (seconds < 60) return `Pre ${seconds} sek`;
  if (seconds < 3600) return `Pre ${Math.floor(seconds / 60)} min`;
  if (seconds < 86400) return `Pre ${Math.floor(seconds / 3600)} h`;
  return `Pre ${Math.floor(seconds / 86400)} dana`;
}
```

---

## Environment Setup

### .env.local

```bash
# Telegram Bot (od @BotFather)
TELEGRAM_BOT_TOKEN=123456789:ABC-DEF_xyz

# Telegram Group ID (negativan broj)
TELEGRAM_CHAT_ID=-100123456789

# App URL
APP_URL=https://patrola-kovacic.vercel.app

# School PIN (može i hardcoded u kodu)
NEXT_PUBLIC_SCHOOL_PIN=kovacic2025

# School Name (za prikaz)
NEXT_PUBLIC_SCHOOL_NAME=OŠ Ivan Goran Kovačić
```

### Kako dobiti TELEGRAM_CHAT_ID

1. Dodaj bota u Telegram grupu
2. Pošalji poruku u grupu
3. Pozovi: `https://api.telegram.org/bot{TOKEN}/getUpdates`
4. Nađi `chat.id` (negativan broj za grupe)

---

## Pokretanje lokalno

```bash
# 1. Kloniraj repo
git clone https://github.com/username/panic-button-poc
cd panic-button-poc

# 2. Instaliraj dependencies
npm install

# 3. Pokreni Convex dev server
npx convex dev

# 4. U drugom terminalu, pokreni Next.js
npm run dev

# 5. Otvori http://localhost:3000
```

---

## Ukupno linija koda

| Fajl | Linije |
|------|--------|
| convex/schema.ts | ~20 |
| convex/alarms.ts | ~80 |
| convex/telegram.ts | ~60 |
| src/lib/auth.ts | ~40 |
| src/components/PanicButton.tsx | ~80 |
| src/components/AlarmCard.tsx | ~120 |
| src/app/page.tsx (login) | ~50 |
| src/app/dashboard/page.tsx | ~80 |
| **UKUPNO** | **~530 linija** |

---

*Dokument kreiran: Januar 2026*
