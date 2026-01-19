# Uporedni Pregled Funkcionalnosti

## Matrica: PoC vs MVP vs Pun Sistem

### Legenda
- ✅ Uključeno
- ❌ Nije uključeno
- ⚡ Pojednostavljeno

---

## Alarm Sistem

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Panic button (drži 3 sek) | ✅ | ✅ | ✅ |
| GPS lokacija | ✅ | ✅ | ✅ |
| Tekstualni opis (opciono) | ✅ | ✅ | ✅ |
| Quick-select tip incidenta | ❌ | ⚡ | ✅ |
| Slanje u Telegram | ✅ | ✅ | ✅ |
| "Preuzimam" dugme | ✅ | ✅ | ✅ |
| "Stigao sam" potvrda | ❌ | ✅ | ✅ |
| Razrešenje alarma | ⚡ | ✅ | ✅ |
| Kategorije razrešenja | ❌ | ❌ | ✅ |
| Beleške pri razrešenju | ❌ | ⚡ | ✅ |
| Cooldown (5 min) | ❌ | ✅ | ✅ |

---

## Eskalacija

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Nivo 1: On-shift responderi | ❌ | ✅ | ✅ |
| Nivo 2: Svi responderi (T+90s) | ❌ | ✅ | ✅ |
| Nivo 3: Svi članovi (T+150s) | ❌ | ❌ | ✅ |
| Konfigurabilan timeout | ❌ | ❌ | ✅ |
| Countdown do eskalacije (vidljiv) | ❌ | ⚡ | ✅ |
| Automatski ping admin-u | ❌ | ❌ | ⚡ |

---

## Autentifikacija

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Pristup sa PIN-om | ✅ | ✅ | ✅ |
| SMS verifikacija | ❌ | ✅ | ✅ |
| Korisnički profili | ❌ | ✅ | ✅ |
| JWT tokeni | ❌ | ✅ | ✅ |
| Refresh tokeni | ❌ | ❌ | ✅ |
| "Zapamti me" opcija | ✅ (localStorage) | ✅ | ✅ |
| Odjava | ⚡ | ✅ | ✅ |

---

## Grupe i Članstvo

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Jedna grupa (hardcoded) | ✅ | ✅ | ❌ |
| Više grupa | ❌ | ❌ | ✅ |
| Kreiranje grupe | ❌ | ✅ | ✅ |
| PIN za pridruživanje | ✅ | ✅ | ✅ |
| Approval workflow | ❌ | ✅ | ✅ |
| Suspend/Ban članova | ❌ | ⚡ | ✅ |
| Više admina po grupi | ❌ | ❌ | ✅ |
| Automatska rotacija PIN-a | ❌ | ❌ | ✅ |

---

## Uloge Korisnika

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Svi jednaki | ✅ | ❌ | ❌ |
| ADMIN uloga | ❌ | ✅ | ✅ |
| RODITELJ uloga | ❌ | ✅ | ✅ |
| RESPONDER uloga | ❌ | ✅ | ✅ |
| DETE uloga | ❌ | ❌ | ✅ |
| Promena uloge | ❌ | ⚡ | ✅ |
| Povezivanje roditelj-dete | ❌ | ❌ | ✅ |

---

## Sistem Smena

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Svi primaju sve alarme | ✅ | ❌ | ❌ |
| Prijava dostupnosti | ❌ | ✅ | ✅ |
| Nedeljni raspored | ❌ | ✅ | ✅ |
| Vremenski slotovi (jutro/ručak/posle) | ❌ | ✅ | ✅ |
| Custom slotovi | ❌ | ❌ | ✅ |
| Upozorenje za nepopunjene smene | ❌ | ⚡ | ✅ |
| Automatski podsjetnici | ❌ | ❌ | ✅ |
| Vikend/praznici podešavanja | ❌ | ❌ | ✅ |

---

## Bystander Effect Mitigacija

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| "Ti si 1 od N" poruka | ❌ | ⚡ | ✅ |
| "0 preuzelo, N videlo" prikaz | ❌ | ⚡ | ✅ |
| Imenovanje dostupnih | ❌ | ❌ | ✅ |
| Prioritet po blizini | ❌ | ❌ | ✅ |
| Countdown pritisak | ❌ | ⚡ | ✅ |
| "Ne mogu" sa razlogom | ❌ | ❌ | ✅ |
| Statistike po responderu | ❌ | ❌ | ✅ |

---

## Admin Panel

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Admin panel postoji | ❌ | ⚡ | ✅ |
| Pregled članova | ❌ | ⚡ | ✅ |
| Approve/Reject zahteva | ❌ | ✅ | ✅ |
| Pregled smena | ❌ | ⚡ | ✅ |
| Istorija alarma | ❌ | ❌ | ✅ |
| Detalji alarma | ❌ | ❌ | ✅ |
| Podešavanja grupe | ❌ | ❌ | ✅ |
| Telegram konekcija UI | ❌ | ⚡ | ✅ |
| Audit log | ❌ | ❌ | ✅ |

---

## Statistike i Izveštaji

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Broj alarma | ❌ | ⚡ | ✅ |
| Prosečno vreme reakcije | ❌ | ⚡ | ✅ |
| Distribucija po vremenu | ❌ | ❌ | ✅ |
| Mapa "vrućih tačaka" | ❌ | ❌ | ✅ |
| Trend analiza | ❌ | ❌ | ✅ |
| Tipovi incidenata | ❌ | ❌ | ✅ |
| Efikasnost sistema | ❌ | ❌ | ✅ |
| Export PDF | ❌ | ❌ | ✅ |
| Export CSV | ❌ | ❌ | ✅ |
| Izveštaj za policiju | ❌ | ❌ | ✅ |

---

## Telegram Integracija

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Slanje alarma u grupu | ✅ | ✅ | ✅ |
| Slanje update-a | ✅ | ✅ | ✅ |
| Hardcoded CHAT_ID | ✅ | ❌ | ❌ |
| Kod-bazirano povezivanje | ❌ | ✅ | ✅ |
| 1:1 mapiranje grupe | ❌ | ✅ | ✅ |
| Admin potvrda pri povezivanju | ❌ | ❌ | ✅ |
| Detekcija uklanjanja bota | ❌ | ⚡ | ✅ |
| Minimalna poruka + link | ⚡ | ✅ | ✅ |
| Rate limit handling | ❌ | ⚡ | ✅ |

---

## PWA Funkcionalnosti

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| Responsive dizajn | ✅ | ✅ | ✅ |
| Install prompt | ⚡ | ✅ | ✅ |
| Offline panic (queue) | ❌ | ⚡ | ✅ |
| Service worker caching | ❌ | ⚡ | ✅ |
| Web push notifikacije | ❌ | ⚡ | ✅ |
| "Offline" indikator | ❌ | ✅ | ✅ |

---

## Bezbednost

| Funkcionalnost | PoC | MVP | Pun |
|----------------|-----|-----|-----|
| HTTPS | ✅ | ✅ | ✅ |
| PIN zaštita | ✅ | ✅ | ✅ |
| SMS verifikacija | ❌ | ✅ | ✅ |
| JWT autentifikacija | ❌ | ✅ | ✅ |
| Rate limiting (SMS) | ❌ | ✅ | ✅ |
| Rate limiting (panic) | ❌ | ✅ | ✅ |
| PIN brute-force zaštita | ❌ | ✅ | ✅ |
| Audit log | ❌ | ❌ | ✅ |
| GDPR compliance | ⚡ | ✅ | ✅ |

---

## Data Model

| Tabela | PoC | MVP | Pun |
|--------|-----|-----|-----|
| alarms | ✅ | ✅ | ✅ |
| users | ❌ | ✅ | ✅ |
| groups | ❌ | ✅ | ✅ |
| memberships | ❌ | ✅ | ✅ |
| shifts | ❌ | ✅ | ✅ |
| alarm_responses | ❌ | ⚡ | ✅ |
| telegram_link_codes | ❌ | ✅ | ✅ |
| sms_codes | ❌ | ✅ | ✅ |
| audit_log | ❌ | ❌ | ✅ |

---

## Infrastruktura

| Komponenta | PoC | MVP | Pun |
|------------|-----|-----|-----|
| Vercel (frontend) | ✅ | ✅ | ✅ |
| Convex (backend) | ✅ | ✅ | ✅ |
| Telegram Bot | ✅ | ✅ | ✅ |
| Twilio (SMS) | ❌ | ✅ | ✅ |
| Cloudflare CDN | ⚡ | ✅ | ✅ |
| Sentry (errors) | ❌ | ⚡ | ✅ |
| UptimeRobot | ❌ | ⚡ | ✅ |

---

## Procena Troškova (mesečno)

| Komponenta | PoC | MVP | Pun (1 škola) | Pun (10 škola) |
|------------|-----|-----|---------------|----------------|
| Vercel | $0 | $0 | $0 | $20 |
| Convex | $0 | $0 | $0 | $25 |
| Twilio SMS | $0 | ~$10 | ~$10 | ~$50 |
| Cloudflare | $0 | $0 | $0 | $0 |
| Railway (bot) | $0 | $0-5 | $0-5 | $5 |
| **UKUPNO** | **$0** | **~$10-15** | **~$10-15** | **~$100** |

---

## Vreme Implementacije

| Faza | Procena |
|------|---------|
| PoC | ~2 sata |
| MVP | 1-2 nedelje |
| Pun sistem | 1-2 meseca |

---

## Preporuka

```
Za početak: PoC → Test sa 5-10 roditelja → Feedback → MVP
```

PoC služi da se DOKAŽE da koncept radi pre nego što se uloži više vremena u kompleksniju verziju.

---

*Dokument kreiran: Januar 2025*
