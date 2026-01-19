# Frontend Components - React/Next.js

## Pregled

Ovaj dokument opisuje React komponente koje čine frontend Panic Button aplikacije.

---

## Struktura Komponenti

```
src/
├── app/                           # Next.js App Router
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Login/PIN page
│   ├── panic/
│   │   └── page.tsx             # Panic button screen
│   ├── alarm/
│   │   └── [id]/
│   │       └── page.tsx         # Alarm details
│   └── admin/                    # (Pun sistem)
│       └── page.tsx
│
├── components/
│   ├── ui/                       # Bazične UI komponente
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   └── Modal.tsx
│   │
│   ├── layout/                   # Layout komponente
│   │   ├── Header.tsx
│   │   └── Footer.tsx
│   │
│   ├── alarm/                    # Alarm-related
│   │   ├── PanicButton.tsx
│   │   ├── AlarmCard.tsx
│   │   ├── AlarmList.tsx
│   │   └── AlarmDetails.tsx
│   │
│   ├── map/                      # Map komponente
│   │   ├── MapView.tsx
│   │   └── LocationMarker.tsx
│   │
│   └── providers/                # Context providers
│       └── ConvexClientProvider.tsx
│
└── lib/
    ├── utils.ts                  # Utility funkcije
    └── hooks/
        ├── useGeolocation.ts
        └── useAuth.ts
```

---

## Ključne Komponente

### PanicButton.tsx

```typescript
// src/components/alarm/PanicButton.tsx
"use client";

import { useState, useRef, useEffect } from "react";
import { useMutation } from "convex/react";
import { api } from "../../../convex/_generated/api";
import { useGeolocation } from "@/lib/hooks/useGeolocation";

const HOLD_DURATION = 3000; // 3 sekunde

interface PanicButtonProps {
  userName: string;
  onSuccess?: (alarmId: string) => void;
  onError?: (error: Error) => void;
}

export function PanicButton({ userName, onSuccess, onError }: PanicButtonProps) {
  const [isHolding, setIsHolding] = useState(false);
  const [progress, setProgress] = useState(0);
  const [isSending, setIsSending] = useState(false);

  const holdTimer = useRef<NodeJS.Timeout | null>(null);
  const progressInterval = useRef<NodeJS.Timeout | null>(null);

  const createAlarm = useMutation(api.alarms.createAlarm);
  const { getCurrentPosition, isSupported } = useGeolocation();

  // Cleanup na unmount
  useEffect(() => {
    return () => {
      if (holdTimer.current) clearTimeout(holdTimer.current);
      if (progressInterval.current) clearInterval(progressInterval.current);
    };
  }, []);

  const handleStart = () => {
    setIsHolding(true);
    setProgress(0);

    // Progress bar animation
    const startTime = Date.now();
    progressInterval.current = setInterval(() => {
      const elapsed = Date.now() - startTime;
      const newProgress = Math.min((elapsed / HOLD_DURATION) * 100, 100);
      setProgress(newProgress);
    }, 50);

    // Hold timer
    holdTimer.current = setTimeout(async () => {
      await triggerAlarm();
    }, HOLD_DURATION);

    // Vibrate if supported
    if (navigator.vibrate) {
      navigator.vibrate(100);
    }
  };

  const handleEnd = () => {
    setIsHolding(false);
    setProgress(0);

    if (holdTimer.current) {
      clearTimeout(holdTimer.current);
      holdTimer.current = null;
    }
    if (progressInterval.current) {
      clearInterval(progressInterval.current);
      progressInterval.current = null;
    }
  };

  const triggerAlarm = async () => {
    setIsSending(true);
    handleEnd();

    try {
      // Get location
      const position = await getCurrentPosition();

      // Create alarm
      const alarmId = await createAlarm({
        sender_name: userName,
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      });

      // Vibrate success
      if (navigator.vibrate) {
        navigator.vibrate([200, 100, 200]);
      }

      onSuccess?.(alarmId);
    } catch (error) {
      console.error("Failed to create alarm:", error);
      onError?.(error as Error);
    } finally {
      setIsSending(false);
    }
  };

  if (!isSupported) {
    return (
      <div className="text-center text-red-500">
        Geolokacija nije podržana na ovom uređaju.
      </div>
    );
  }

  return (
    <div className="flex flex-col items-center gap-4">
      {/* Main Button */}
      <button
        onMouseDown={handleStart}
        onMouseUp={handleEnd}
        onMouseLeave={handleEnd}
        onTouchStart={handleStart}
        onTouchEnd={handleEnd}
        disabled={isSending}
        className={`
          relative w-64 h-64 rounded-full
          transition-all duration-200
          ${isHolding
            ? "bg-red-700 scale-95"
            : "bg-red-600 hover:bg-red-500"
          }
          ${isSending ? "opacity-50 cursor-not-allowed" : "cursor-pointer"}
          shadow-lg
          flex items-center justify-center
          select-none
        `}
      >
        {/* Progress Ring */}
        <svg
          className="absolute inset-0 w-full h-full -rotate-90"
          viewBox="0 0 100 100"
        >
          <circle
            cx="50"
            cy="50"
            r="48"
            fill="none"
            stroke="rgba(255,255,255,0.3)"
            strokeWidth="4"
          />
          <circle
            cx="50"
            cy="50"
            r="48"
            fill="none"
            stroke="white"
            strokeWidth="4"
            strokeDasharray={`${progress * 3.02} 302`}
            className="transition-all duration-50"
          />
        </svg>

        {/* Button Content */}
        <div className="text-white text-center z-10">
          {isSending ? (
            <span className="text-xl">Šaljem...</span>
          ) : isHolding ? (
            <span className="text-xl">Drži...</span>
          ) : (
            <>
              <span className="text-3xl font-bold block">PANIC</span>
              <span className="text-sm opacity-75">Drži 3 sekunde</span>
            </>
          )}
        </div>
      </button>

      {/* Instructions */}
      <p className="text-gray-500 text-sm text-center max-w-xs">
        Drži dugme 3 sekunde da pošalješ alarm sa svojom lokacijom
      </p>
    </div>
  );
}
```

### AlarmCard.tsx

```typescript
// src/components/alarm/AlarmCard.tsx
"use client";

import { formatDistanceToNow } from "date-fns";
import { sr } from "date-fns/locale";
import Link from "next/link";

interface Alarm {
  _id: string;
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
  showActions?: boolean;
  onTake?: () => void;
}

const statusColors = {
  ACTIVE: "bg-red-100 border-red-500 text-red-800",
  TAKEN: "bg-yellow-100 border-yellow-500 text-yellow-800",
  RESOLVED: "bg-green-100 border-green-500 text-green-800",
  CANCELLED: "bg-gray-100 border-gray-500 text-gray-800",
  FALSE_ALARM: "bg-orange-100 border-orange-500 text-orange-800",
};

const statusLabels = {
  ACTIVE: "🚨 Aktivan",
  TAKEN: "🏃 Preuzeto",
  RESOLVED: "✅ Rešeno",
  CANCELLED: "❌ Otkazano",
  FALSE_ALARM: "⚠️ Lažni alarm",
};

export function AlarmCard({ alarm, showActions = true, onTake }: AlarmCardProps) {
  const timeAgo = formatDistanceToNow(new Date(alarm.created_at), {
    addSuffix: true,
    locale: sr,
  });

  const mapsUrl = `https://maps.google.com/?q=${alarm.lat},${alarm.lng}`;

  return (
    <div
      className={`
        border-l-4 rounded-lg p-4 shadow-sm
        ${statusColors[alarm.status]}
      `}
    >
      {/* Header */}
      <div className="flex justify-between items-start mb-2">
        <div>
          <span className="font-semibold">{alarm.sender_name}</span>
          <span className="text-sm opacity-75 ml-2">{timeAgo}</span>
        </div>
        <span className="text-sm font-medium">
          {statusLabels[alarm.status]}
        </span>
      </div>

      {/* Message */}
      {alarm.message && (
        <p className="text-sm mb-2 italic">"{alarm.message}"</p>
      )}

      {/* Location */}
      <div className="flex items-center gap-2 text-sm mb-3">
        <span>📍</span>
        <a
          href={mapsUrl}
          target="_blank"
          rel="noopener noreferrer"
          className="underline hover:no-underline"
        >
          Otvori mapu
        </a>
      </div>

      {/* Taken info */}
      {alarm.taken_by && (
        <div className="text-sm mb-3">
          <span className="font-medium">{alarm.taken_by}</span> je preuzeo
          {alarm.taken_at && (
            <span className="opacity-75">
              {" "}
              {formatDistanceToNow(new Date(alarm.taken_at), {
                addSuffix: true,
                locale: sr,
              })}
            </span>
          )}
        </div>
      )}

      {/* Actions */}
      {showActions && alarm.status === "ACTIVE" && (
        <div className="flex gap-2 mt-3">
          <button
            onClick={onTake}
            className="flex-1 bg-blue-600 text-white py-2 px-4 rounded-lg
                     hover:bg-blue-700 transition-colors font-medium"
          >
            🏃 Preuzimam
          </button>
          <Link
            href={`/alarm/${alarm._id}`}
            className="py-2 px-4 border border-current rounded-lg
                     hover:bg-white/50 transition-colors"
          >
            Detalji
          </Link>
        </div>
      )}
    </div>
  );
}
```

### AlarmList.tsx

```typescript
// src/components/alarm/AlarmList.tsx
"use client";

import { useQuery, useMutation } from "convex/react";
import { api } from "../../../convex/_generated/api";
import { AlarmCard } from "./AlarmCard";
import { useState } from "react";

interface AlarmListProps {
  userName: string;
}

export function AlarmList({ userName }: AlarmListProps) {
  const alarms = useQuery(api.alarms.getActiveAlarms);
  const takeAlarm = useMutation(api.alarms.takeAlarm);
  const [taking, setTaking] = useState<string | null>(null);

  const handleTake = async (alarmId: string) => {
    setTaking(alarmId);
    try {
      await takeAlarm({
        id: alarmId,
        taken_by: userName,
      });
    } catch (error) {
      console.error("Failed to take alarm:", error);
      alert("Greška pri preuzimanju alarma");
    } finally {
      setTaking(null);
    }
  };

  if (alarms === undefined) {
    return (
      <div className="flex justify-center p-8">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
      </div>
    );
  }

  if (alarms.length === 0) {
    return (
      <div className="text-center p-8 text-gray-500">
        <span className="text-4xl block mb-2">✅</span>
        Nema aktivnih alarma
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <h2 className="text-lg font-semibold text-gray-800">
        Aktivni alarmi ({alarms.length})
      </h2>

      {alarms.map((alarm) => (
        <AlarmCard
          key={alarm._id}
          alarm={alarm}
          onTake={() => handleTake(alarm._id)}
        />
      ))}
    </div>
  );
}
```

### MapView.tsx

```typescript
// src/components/map/MapView.tsx
"use client";

import { useEffect, useRef } from "react";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

interface MapViewProps {
  lat: number;
  lng: number;
  zoom?: number;
  className?: string;
}

export function MapView({ lat, lng, zoom = 15, className = "" }: MapViewProps) {
  const mapRef = useRef<HTMLDivElement>(null);
  const mapInstanceRef = useRef<L.Map | null>(null);

  useEffect(() => {
    if (!mapRef.current || mapInstanceRef.current) return;

    // Initialize map
    const map = L.map(mapRef.current).setView([lat, lng], zoom);

    // Add tile layer (OpenStreetMap)
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    // Add marker
    const icon = L.divIcon({
      className: "custom-marker",
      html: `
        <div class="w-8 h-8 bg-red-600 rounded-full border-4 border-white
                    shadow-lg flex items-center justify-center">
          <span class="text-white text-xs">📍</span>
        </div>
      `,
      iconSize: [32, 32],
      iconAnchor: [16, 32],
    });

    L.marker([lat, lng], { icon }).addTo(map);

    mapInstanceRef.current = map;

    // Cleanup
    return () => {
      map.remove();
      mapInstanceRef.current = null;
    };
  }, [lat, lng, zoom]);

  return (
    <div
      ref={mapRef}
      className={`w-full h-64 rounded-lg overflow-hidden ${className}`}
    />
  );
}
```

### useGeolocation.ts

```typescript
// src/lib/hooks/useGeolocation.ts
"use client";

import { useState, useCallback } from "react";

interface GeolocationState {
  isSupported: boolean;
  isLoading: boolean;
  error: string | null;
  position: GeolocationPosition | null;
}

export function useGeolocation() {
  const [state, setState] = useState<GeolocationState>({
    isSupported: typeof navigator !== "undefined" && "geolocation" in navigator,
    isLoading: false,
    error: null,
    position: null,
  });

  const getCurrentPosition = useCallback((): Promise<GeolocationPosition> => {
    return new Promise((resolve, reject) => {
      if (!state.isSupported) {
        reject(new Error("Geolocation not supported"));
        return;
      }

      setState((prev) => ({ ...prev, isLoading: true, error: null }));

      navigator.geolocation.getCurrentPosition(
        (position) => {
          setState((prev) => ({
            ...prev,
            isLoading: false,
            position,
          }));
          resolve(position);
        },
        (error) => {
          let errorMessage = "Unknown error";
          switch (error.code) {
            case error.PERMISSION_DENIED:
              errorMessage = "Dozvola za lokaciju je odbijena";
              break;
            case error.POSITION_UNAVAILABLE:
              errorMessage = "Lokacija nije dostupna";
              break;
            case error.TIMEOUT:
              errorMessage = "Zahtev je istekao";
              break;
          }
          setState((prev) => ({
            ...prev,
            isLoading: false,
            error: errorMessage,
          }));
          reject(new Error(errorMessage));
        },
        {
          enableHighAccuracy: true,
          timeout: 10000,
          maximumAge: 0,
        }
      );
    });
  }, [state.isSupported]);

  return {
    ...state,
    getCurrentPosition,
  };
}
```

---

## Stranice

### Login Page (PIN)

```typescript
// src/app/page.tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";

const CORRECT_PIN = process.env.NEXT_PUBLIC_GROUP_PIN || "123456";

export default function LoginPage() {
  const [pin, setPin] = useState("");
  const [name, setName] = useState("");
  const [error, setError] = useState("");
  const router = useRouter();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    if (!name.trim()) {
      setError("Unesite vaše ime");
      return;
    }

    if (pin !== CORRECT_PIN) {
      setError("Pogrešan PIN");
      return;
    }

    // Store name in localStorage
    localStorage.setItem("userName", name.trim());
    router.push("/panic");
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100 p-4">
      <div className="bg-white rounded-xl shadow-lg p-8 w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6">
          🚨 Patrola
        </h1>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Vaše ime
            </label>
            <input
              type="text"
              value={name}
              onChange={(e) => setName(e.target.value)}
              placeholder="npr. Marko Petrović"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2
                       focus:ring-blue-500 focus:border-transparent"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              PIN grupe
            </label>
            <input
              type="password"
              inputMode="numeric"
              value={pin}
              onChange={(e) => setPin(e.target.value.replace(/\D/g, ""))}
              maxLength={6}
              placeholder="••••••"
              className="w-full px-4 py-2 border rounded-lg focus:ring-2
                       focus:ring-blue-500 focus:border-transparent
                       text-center text-2xl tracking-widest"
            />
          </div>

          {error && (
            <p className="text-red-500 text-sm text-center">{error}</p>
          )}

          <button
            type="submit"
            className="w-full bg-blue-600 text-white py-3 rounded-lg
                     font-medium hover:bg-blue-700 transition-colors"
          >
            Prijavi se
          </button>
        </form>
      </div>
    </div>
  );
}
```

### Panic Page

```typescript
// src/app/panic/page.tsx
"use client";

import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import { PanicButton } from "@/components/alarm/PanicButton";
import { AlarmList } from "@/components/alarm/AlarmList";

export default function PanicPage() {
  const [userName, setUserName] = useState<string | null>(null);
  const [showSuccess, setShowSuccess] = useState(false);
  const router = useRouter();

  useEffect(() => {
    const storedName = localStorage.getItem("userName");
    if (!storedName) {
      router.replace("/");
      return;
    }
    setUserName(storedName);
  }, [router]);

  const handleSuccess = (alarmId: string) => {
    setShowSuccess(true);
    setTimeout(() => {
      router.push(`/alarm/${alarmId}`);
    }, 2000);
  };

  if (!userName) {
    return null; // Loading
  }

  if (showSuccess) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-green-500">
        <div className="text-center text-white">
          <span className="text-6xl block mb-4">✅</span>
          <h1 className="text-2xl font-bold">Alarm poslat!</h1>
          <p className="opacity-75">Pomoć je na putu...</p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100">
      {/* Header */}
      <header className="bg-white shadow-sm p-4">
        <div className="flex justify-between items-center max-w-lg mx-auto">
          <h1 className="text-lg font-semibold">🚨 Patrola</h1>
          <span className="text-sm text-gray-500">{userName}</span>
        </div>
      </header>

      {/* Main Content */}
      <main className="max-w-lg mx-auto p-4 space-y-8">
        {/* Panic Button */}
        <section className="bg-white rounded-xl shadow-lg p-8">
          <PanicButton
            userName={userName}
            onSuccess={handleSuccess}
            onError={(e) => alert(`Greška: ${e.message}`)}
          />
        </section>

        {/* Active Alarms */}
        <section className="bg-white rounded-xl shadow-lg p-4">
          <AlarmList userName={userName} />
        </section>
      </main>
    </div>
  );
}
```

### Alarm Details Page

```typescript
// src/app/alarm/[id]/page.tsx
"use client";

import { useQuery, useMutation } from "convex/react";
import { api } from "../../../../convex/_generated/api";
import { MapView } from "@/components/map/MapView";
import { useParams, useRouter } from "next/navigation";
import { useState, useEffect } from "react";

export default function AlarmDetailsPage() {
  const params = useParams();
  const router = useRouter();
  const alarmId = params.id as string;

  const alarm = useQuery(api.alarms.getAlarm, { id: alarmId as any });
  const takeAlarm = useMutation(api.alarms.takeAlarm);
  const resolveAlarm = useMutation(api.alarms.resolveAlarm);

  const [userName, setUserName] = useState<string>("");

  useEffect(() => {
    const storedName = localStorage.getItem("userName");
    if (storedName) setUserName(storedName);
  }, []);

  if (alarm === undefined) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
      </div>
    );
  }

  if (alarm === null) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <p className="text-gray-500">Alarm nije pronađen</p>
      </div>
    );
  }

  const handleTake = async () => {
    try {
      await takeAlarm({ id: alarm._id, taken_by: userName });
    } catch (error) {
      alert("Greška pri preuzimanju");
    }
  };

  const handleResolve = async () => {
    try {
      await resolveAlarm({ id: alarm._id });
      router.push("/panic");
    } catch (error) {
      alert("Greška pri rešavanju");
    }
  };

  return (
    <div className="min-h-screen bg-gray-100">
      {/* Header */}
      <header className="bg-white shadow-sm p-4">
        <div className="flex items-center gap-4 max-w-lg mx-auto">
          <button onClick={() => router.back()} className="text-gray-500">
            ← Nazad
          </button>
          <h1 className="text-lg font-semibold">Detalji alarma</h1>
        </div>
      </header>

      <main className="max-w-lg mx-auto p-4 space-y-4">
        {/* Status Badge */}
        <div className={`
          p-4 rounded-lg text-center font-medium
          ${alarm.status === "ACTIVE" ? "bg-red-100 text-red-800" : ""}
          ${alarm.status === "TAKEN" ? "bg-yellow-100 text-yellow-800" : ""}
          ${alarm.status === "RESOLVED" ? "bg-green-100 text-green-800" : ""}
        `}>
          {alarm.status === "ACTIVE" && "🚨 AKTIVAN ALARM"}
          {alarm.status === "TAKEN" && `🏃 ${alarm.taken_by} je na putu`}
          {alarm.status === "RESOLVED" && "✅ Rešeno"}
        </div>

        {/* Map */}
        <div className="bg-white rounded-xl shadow-lg overflow-hidden">
          <MapView lat={alarm.lat} lng={alarm.lng} />
        </div>

        {/* Info */}
        <div className="bg-white rounded-xl shadow-lg p-4 space-y-3">
          <div>
            <span className="text-sm text-gray-500">Poslao</span>
            <p className="font-medium">{alarm.sender_name}</p>
          </div>

          {alarm.message && (
            <div>
              <span className="text-sm text-gray-500">Poruka</span>
              <p className="italic">"{alarm.message}"</p>
            </div>
          )}

          <div>
            <span className="text-sm text-gray-500">Vreme</span>
            <p>{new Date(alarm.created_at).toLocaleString("sr-RS")}</p>
          </div>
        </div>

        {/* Actions */}
        <div className="space-y-2">
          {alarm.status === "ACTIVE" && (
            <button
              onClick={handleTake}
              className="w-full bg-blue-600 text-white py-3 rounded-lg
                       font-medium hover:bg-blue-700 transition-colors"
            >
              🏃 Preuzimam
            </button>
          )}

          {alarm.status === "TAKEN" && alarm.taken_by === userName && (
            <button
              onClick={handleResolve}
              className="w-full bg-green-600 text-white py-3 rounded-lg
                       font-medium hover:bg-green-700 transition-colors"
            >
              ✅ Označi kao rešeno
            </button>
          )}

          <a
            href={`https://maps.google.com/?q=${alarm.lat},${alarm.lng}`}
            target="_blank"
            rel="noopener noreferrer"
            className="block w-full text-center bg-gray-200 py-3 rounded-lg
                     font-medium hover:bg-gray-300 transition-colors"
          >
            🗺️ Otvori u Google Maps
          </a>
        </div>
      </main>
    </div>
  );
}
```

---

## Styling (Tailwind)

### tailwind.config.js

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: "#fef2f2",
          100: "#fee2e2",
          500: "#ef4444",
          600: "#dc2626",
          700: "#b91c1c",
        },
      },
    },
  },
  plugins: [
    require("@tailwindcss/forms"),
  ],
};
```

---

*Dokument kreiran: Januar 2026*
