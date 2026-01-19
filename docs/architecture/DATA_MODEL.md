# Data Model - Convex Schema

## Pregled

Ovaj dokument opisuje kompletnu strukturu baze podataka za Panic Button aplikaciju.

---

## PoC Schema (Minimalna)

```typescript
// convex/schema.ts - PoC verzija

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Jedina tabela u PoC-u
  alarms: defineTable({
    // Ko je poslao alarm
    sender_name: v.string(),

    // Opciona poruka
    message: v.optional(v.string()),

    // GPS koordinate
    lat: v.number(),
    lng: v.number(),

    // Status alarma
    status: v.union(
      v.literal("ACTIVE"),      // Aktivan, čeka reakciju
      v.literal("TAKEN"),       // Neko je preuzeo
      v.literal("RESOLVED"),    // Rešeno
      v.literal("CANCELLED"),   // Otkazano od strane pošiljaoca
      v.literal("FALSE_ALARM")  // Lažni alarm
    ),

    // Ko je preuzeo (ime)
    taken_by: v.optional(v.string()),
    taken_at: v.optional(v.number()),

    // Timestamps
    created_at: v.number(),
    resolved_at: v.optional(v.number()),
  })
    .index("by_status", ["status"])
    .index("by_created", ["created_at"]),
});
```

---

## Full System Schema

```typescript
// convex/schema.ts - Pun sistem

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // ═══════════════════════════════════════════════════════════════════════════
  // USERS - Registrovani korisnici
  // ═══════════════════════════════════════════════════════════════════════════
  users: defineTable({
    // Osnovni podaci
    phone: v.string(),              // "+381631234567"
    name: v.string(),               // "Marko Petrović"

    // Verifikacija
    phone_verified: v.boolean(),
    verified_at: v.optional(v.number()),

    // Status
    status: v.union(
      v.literal("ACTIVE"),
      v.literal("SUSPENDED"),
      v.literal("BANNED")
    ),

    // Timestamps
    created_at: v.number(),
    last_login_at: v.optional(v.number()),
  })
    .index("by_phone", ["phone"])
    .index("by_status", ["status"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // GROUPS - Škole/grupe roditelja
  // ═══════════════════════════════════════════════════════════════════════════
  groups: defineTable({
    // Osnovni podaci
    name: v.string(),               // "OŠ Ivan Goran Kovačić"
    description: v.optional(v.string()),

    // PIN za pristup (PoC kompatibilnost)
    access_pin: v.string(),         // "482916"
    pin_rotated_at: v.optional(v.number()),

    // Telegram integracija
    telegram_chat_id: v.optional(v.string()),
    telegram_connected_at: v.optional(v.number()),
    telegram_connected_by: v.optional(v.id("users")),

    // Geofence (opciono)
    center_lat: v.optional(v.number()),
    center_lng: v.optional(v.number()),
    radius_meters: v.optional(v.number()),

    // Podešavanja eskalacije
    escalation_1_seconds: v.number(),  // Default: 90
    escalation_2_seconds: v.number(),  // Default: 60 (nakon prve)
    min_responders_per_shift: v.number(), // Default: 2

    // Radno vreme
    active_from: v.string(),        // "07:00"
    active_to: v.string(),          // "18:00"
    weekend_active: v.boolean(),

    // Status
    status: v.union(
      v.literal("ACTIVE"),
      v.literal("INACTIVE"),
      v.literal("DELETED")
    ),

    // Timestamps
    created_at: v.number(),
    created_by: v.id("users"),
  })
    .index("by_pin", ["access_pin"])
    .index("by_telegram", ["telegram_chat_id"])
    .index("by_status", ["status"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // MEMBERSHIPS - Veza korisnik-grupa sa ulogom
  // ═══════════════════════════════════════════════════════════════════════════
  memberships: defineTable({
    user_id: v.id("users"),
    group_id: v.id("groups"),

    // Uloga u grupi
    role: v.union(
      v.literal("ADMIN"),       // Može sve
      v.literal("RESPONDER"),   // Može reagovati na alarme
      v.literal("RODITELJ"),    // Može slati alarme + ograničen pogled
      v.literal("DETE")         // Može samo slati alarme
    ),

    // Status članstva
    status: v.union(
      v.literal("PENDING"),     // Čeka odobrenje
      v.literal("ACTIVE"),      // Aktivno
      v.literal("SUSPENDED"),   // Suspenovan
      v.literal("BANNED"),      // Banovan
      v.literal("LEFT")         // Napustio grupu
    ),

    // Za DETE ulogu - povezan roditelj
    parent_membership_id: v.optional(v.id("memberships")),

    // Approval workflow
    approved_by: v.optional(v.id("users")),
    approved_at: v.optional(v.number()),
    rejected_by: v.optional(v.id("users")),
    rejected_at: v.optional(v.number()),
    rejection_reason: v.optional(v.string()),

    // Timestamps
    joined_at: v.number(),
    left_at: v.optional(v.number()),
  })
    .index("by_user", ["user_id"])
    .index("by_group", ["group_id"])
    .index("by_user_group", ["user_id", "group_id"])
    .index("by_role", ["group_id", "role"])
    .index("by_status", ["group_id", "status"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // SHIFTS - Raspored smena respondera
  // ═══════════════════════════════════════════════════════════════════════════
  shifts: defineTable({
    membership_id: v.id("memberships"),
    group_id: v.id("groups"),

    // Datum i slot
    date: v.string(),              // "2025-01-20"
    time_slot: v.union(
      v.literal("MORNING"),        // 07:00-09:00
      v.literal("MIDDAY"),         // 12:00-14:00
      v.literal("AFTERNOON")       // 14:00-17:00
    ),

    // Ili custom vreme
    custom_start: v.optional(v.string()),  // "08:30"
    custom_end: v.optional(v.string()),    // "09:30"

    // Status smene
    status: v.union(
      v.literal("SCHEDULED"),      // Zakazano
      v.literal("ACTIVE"),         // Trenutno aktivna
      v.literal("COMPLETED"),      // Završena
      v.literal("MISSED"),         // Propuštena
      v.literal("CANCELLED")       // Otkazana
    ),

    // Timestamps
    created_at: v.number(),
  })
    .index("by_group_date", ["group_id", "date"])
    .index("by_membership", ["membership_id"])
    .index("by_status", ["group_id", "status"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // ALARMS - Alarmi
  // ═══════════════════════════════════════════════════════════════════════════
  alarms: defineTable({
    group_id: v.id("groups"),
    sender_id: v.id("users"),

    // Poruka
    message: v.optional(v.string()),

    // GPS lokacija
    lat: v.number(),
    lng: v.number(),
    location_accuracy: v.optional(v.number()),

    // Status
    status: v.union(
      v.literal("ACTIVE"),
      v.literal("ESCALATED_1"),    // Eskaliran na sve respondere
      v.literal("ESCALATED_2"),    // Eskaliran na sve članove
      v.literal("TAKEN"),          // Neko je preuzeo
      v.literal("RESOLVED"),
      v.literal("CANCELLED"),
      v.literal("FALSE_ALARM")
    ),

    // Eskalacija timestamps
    escalated_1_at: v.optional(v.number()),
    escalated_2_at: v.optional(v.number()),

    // Preuzimanje
    taken_by_id: v.optional(v.id("users")),
    taken_at: v.optional(v.number()),
    taken_lat: v.optional(v.number()),
    taken_lng: v.optional(v.number()),
    estimated_arrival: v.optional(v.number()), // sekunde

    // Dolazak na lokaciju
    arrived_at: v.optional(v.number()),

    // Rezolucija
    resolved_by_id: v.optional(v.id("users")),
    resolved_at: v.optional(v.number()),
    resolution_type: v.optional(v.union(
      v.literal("ESCORTED_HOME"),     // Sprovedeno do kuće
      v.literal("THREAT_DISPERSED"),  // Pretnja se razišla
      v.literal("POLICE_CALLED"),     // Pozvana policija
      v.literal("FALSE_ALARM"),       // Lažni alarm
      v.literal("CANCELLED"),         // Otkazano
      v.literal("OTHER")              // Drugo
    )),
    resolution_notes: v.optional(v.string()),

    // Scheduled job IDs za otkazivanje
    escalation_1_job_id: v.optional(v.string()),
    escalation_2_job_id: v.optional(v.string()),

    // Timestamps
    created_at: v.number(),
  })
    .index("by_group", ["group_id"])
    .index("by_sender", ["sender_id"])
    .index("by_status", ["status"])
    .index("by_group_status", ["group_id", "status"])
    .index("by_created", ["created_at"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // ALARM_RESPONSES - Reakcije na alarm
  // ═══════════════════════════════════════════════════════════════════════════
  alarm_responses: defineTable({
    alarm_id: v.id("alarms"),
    user_id: v.id("users"),

    // Tip reakcije
    response_type: v.union(
      v.literal("SEEN"),          // Video alarm
      v.literal("TAKING"),        // Preuzima (na putu)
      v.literal("ARRIVED"),       // Stigao
      v.literal("BACKUP"),        // Ide kao backup
      v.literal("DECLINED"),      // Odbio
      v.literal("IGNORED")        // Nije reagovao (sistem označava)
    ),

    // Za DECLINED - razlog
    decline_reason: v.optional(v.union(
      v.literal("AT_WORK"),
      v.literal("NOT_NEARBY"),
      v.literal("HEALTH"),
      v.literal("ANOTHER_ALARM"),
      v.literal("OTHER")
    )),
    decline_note: v.optional(v.string()),

    // Lokacija kada je reagovao
    response_lat: v.optional(v.number()),
    response_lng: v.optional(v.number()),
    distance_meters: v.optional(v.number()),

    // Timestamps
    created_at: v.number(),
  })
    .index("by_alarm", ["alarm_id"])
    .index("by_user", ["user_id"])
    .index("by_type", ["alarm_id", "response_type"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // TELEGRAM_LINK_CODES - Kodovi za povezivanje TG grupe
  // ═══════════════════════════════════════════════════════════════════════════
  telegram_link_codes: defineTable({
    code: v.string(),             // "TG-7X9K2M"
    chat_id: v.string(),          // "-100123456789"
    chat_title: v.string(),       // "OŠ Kovačić Patrola"

    // Status
    used: v.boolean(),
    used_by_group: v.optional(v.id("groups")),

    // Timestamps
    created_at: v.number(),
    expires_at: v.number(),       // created_at + 15min
    used_at: v.optional(v.number()),
  })
    .index("by_code", ["code"])
    .index("by_chat", ["chat_id"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // SMS_CODES - Verifikacioni kodovi
  // ═══════════════════════════════════════════════════════════════════════════
  sms_codes: defineTable({
    phone: v.string(),            // "+381631234567"
    code: v.string(),             // "482916"

    // Rate limiting
    attempts: v.number(),         // Broj pokušaja unosa

    // Timestamps
    created_at: v.number(),
    expires_at: v.number(),       // created_at + 10min
    verified_at: v.optional(v.number()),
  })
    .index("by_phone", ["phone"])
    .index("by_phone_code", ["phone", "code"]),

  // ═══════════════════════════════════════════════════════════════════════════
  // AUDIT_LOG - Log admin akcija
  // ═══════════════════════════════════════════════════════════════════════════
  audit_log: defineTable({
    // Ko je izvršio akciju
    user_id: v.id("users"),
    group_id: v.optional(v.id("groups")),

    // Akcija
    action: v.string(),           // "MEMBER_BANNED", "ROLE_CHANGED", etc.

    // Detalji
    details: v.optional(v.any()), // { target_user: "...", reason: "..." }

    // Metadata
    ip_address: v.optional(v.string()),
    user_agent: v.optional(v.string()),

    // Timestamp
    created_at: v.number(),
  })
    .index("by_user", ["user_id"])
    .index("by_group", ["group_id"])
    .index("by_action", ["action"])
    .index("by_created", ["created_at"]),
});
```

---

## Dijagram Relacija

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          ENTITY RELATIONSHIP DIAGRAM                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────┐       ┌──────────────┐       ┌─────────┐                          │
│   │  USERS  │───────│ MEMBERSHIPS  │───────│ GROUPS  │                          │
│   │         │  N:M  │              │  N:M  │         │                          │
│   └────┬────┘       └──────┬───────┘       └────┬────┘                          │
│        │                   │                    │                               │
│        │                   │                    │                               │
│        │            ┌──────┴───────┐            │                               │
│        │            │   SHIFTS     │            │                               │
│        │            │ (raspored)   │            │                               │
│        │            └──────────────┘            │                               │
│        │                                        │                               │
│        │                                        │                               │
│        │            ┌──────────────┐            │                               │
│        └───────────►│   ALARMS     │◄───────────┘                               │
│          sender     │              │    group                                   │
│                     └──────┬───────┘                                            │
│                            │                                                    │
│                            │                                                    │
│                     ┌──────┴───────┐                                            │
│                     │   ALARM_     │                                            │
│                     │  RESPONSES   │                                            │
│                     └──────────────┘                                            │
│                                                                                 │
│                                                                                 │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│   │  TELEGRAM_      │    │   SMS_CODES     │    │   AUDIT_LOG     │             │
│   │  LINK_CODES     │    │ (verifikacija)  │    │ (praćenje)      │             │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Indeksi i Performance

### Najčešći Query Patterns

| Query | Index | Opis |
|-------|-------|------|
| Aktivni alarmi grupe | `by_group_status` | Dashboard, lista alarma |
| Članovi grupe | `by_group` (memberships) | Admin panel |
| Smene za datum | `by_group_date` | Raspored |
| Reakcije na alarm | `by_alarm` | Detalji alarma |
| Istorija alarma | `by_created` | Statistike |

### Optimizacije

```typescript
// Compound index za česte filtere
.index("by_group_status", ["group_id", "status"])

// Partial fetch sa projekcijom
const activeAlarms = await ctx.db
  .query("alarms")
  .withIndex("by_group_status", (q) =>
    q.eq("group_id", groupId).eq("status", "ACTIVE")
  )
  .collect();
```

---

## Migracije

### Dodavanje novog polja

```typescript
// Convex podržava dodavanje nullable polja bez migracije
// Stari dokumenti će imati undefined

// schema.ts
alarms: defineTable({
  // ...postojeća polja
  new_field: v.optional(v.string()), // ✅ Bezbedno dodati
})

// Za required polja, potrebna je migracija:
// 1. Dodaj kao optional
// 2. Backfill postojeće dokumente
// 3. Promeni u required
```

### Primer Backfill Migracije

```typescript
// convex/migrations/addNewField.ts
import { internalMutation } from "./_generated/server";

export const backfillNewField = internalMutation({
  handler: async (ctx) => {
    const alarms = await ctx.db.query("alarms").collect();

    for (const alarm of alarms) {
      if (alarm.new_field === undefined) {
        await ctx.db.patch(alarm._id, {
          new_field: "default_value",
        });
      }
    }
  },
});
```

---

## Validacija

### Convex Validators

```typescript
// Validators su ugrađeni u schema
// Automatski se primenjuju na svaki insert/update

// Custom validacija u mutation-u
export const createAlarm = mutation({
  args: {
    message: v.optional(v.string()),
    lat: v.number(),
    lng: v.number(),
  },
  handler: async (ctx, args) => {
    // Dodatna validacija
    if (args.lat < -90 || args.lat > 90) {
      throw new Error("Invalid latitude");
    }
    if (args.lng < -180 || args.lng > 180) {
      throw new Error("Invalid longitude");
    }
    if (args.message && args.message.length > 500) {
      throw new Error("Message too long");
    }

    // Insert
    return await ctx.db.insert("alarms", {
      ...args,
      status: "ACTIVE",
      created_at: Date.now(),
    });
  },
});
```

---

## Backup i Restore

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  BACKUP STRATEGIJA                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CONVEX BUILT-IN:                                                               │
│  • Automatski snapshots                                                         │
│  • Point-in-time recovery (Pro plan)                                            │
│                                                                                 │
│  MANUAL BACKUP:                                                                 │
│  npx convex export --path ./backup                                              │
│                                                                                 │
│  RESTORE:                                                                       │
│  npx convex import --path ./backup                                              │
│                                                                                 │
│  PREPORUKA:                                                                     │
│  • Weekly manual exports                                                        │
│  • Store u S3/GCS                                                               │
│  • Test restore procedure                                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

*Dokument kreiran: Januar 2026*
