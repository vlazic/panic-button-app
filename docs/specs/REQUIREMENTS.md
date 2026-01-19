# Zahtevi Sistema

## Funkcionalni Zahtevi

### FR1: Panic Button

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR1.1 | Korisnik može aktivirati panic alarm držanjem dugmeta 3 sekunde | MUST | PoC |
| FR1.2 | Sistem automatski preuzima GPS lokaciju | MUST | PoC |
| FR1.3 | Korisnik može dodati tekstualni opis (opciono) | SHOULD | PoC |
| FR1.4 | Korisnik može uneti svoje ime (opciono) | SHOULD | PoC |
| FR1.5 | Korisnik može otkazati alarm pre slanja | MUST | PoC |
| FR1.6 | Alarm se šalje u roku od 2 sekunde | MUST | PoC |
| FR1.7 | Korisnik dobija potvrdu da je alarm poslat | MUST | PoC |
| FR1.8 | Quick-select opcije za tip incidenta (prate me, tuča, kradu, preti mi) | COULD | MVP |

### FR2: Notifikacije

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR2.1 | Alarm se prosleđuje u Telegram grupu | MUST | PoC |
| FR2.2 | Telegram poruka sadrži lokaciju (Google Maps link) | MUST | PoC |
| FR2.3 | Telegram poruka sadrži opis ako je unet | MUST | PoC |
| FR2.4 | Telegram poruka sadrži link na web app | MUST | PoC |
| FR2.5 | Notifikacija o preuzimanju alarma | MUST | PoC |
| FR2.6 | Notifikacija o razrešenju alarma | SHOULD | PoC |
| FR2.7 | Ciljane notifikacije po smeni | SHOULD | MVP |
| FR2.8 | Ciljane notifikacije po blizini | COULD | Pun |

### FR3: Preuzimanje Alarma

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR3.1 | Korisnik može preuzeti aktivan alarm | MUST | PoC |
| FR3.2 | Mora uneti svoje ime pri preuzimanju | MUST | PoC |
| FR3.3 | Svi vide ko je preuzeo alarm (real-time) | MUST | PoC |
| FR3.4 | Korisnik može odustati od preuzimanja | SHOULD | PoC |
| FR3.5 | Više korisnika može preuzeti isti alarm (backup) | COULD | MVP |
| FR3.6 | ETA izračunavanje na osnovu lokacije | COULD | Pun |
| FR3.7 | "Stigao sam" potvrda | SHOULD | MVP |

### FR4: Razrešenje Alarma

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR4.1 | Alarm se može označiti kao REŠEN | MUST | PoC |
| FR4.2 | Alarm se može označiti kao LAŽNI | MUST | PoC |
| FR4.3 | Alarm se može OTKAZATI (od strane pošiljaoca) | MUST | PoC |
| FR4.4 | Unos beleški pri razrešenju (opciono) | SHOULD | MVP |
| FR4.5 | Kategorije razrešenja (sprovedeno kući, pozvana policija, itd.) | COULD | Pun |

### FR5: Autentifikacija (PoC)

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR5.1 | Pristup aplikaciji sa PIN-om | MUST | PoC |
| FR5.2 | PIN se čuva u localStorage | MUST | PoC |
| FR5.3 | Korisnik može da se odjavi | SHOULD | PoC |

### FR6: Autentifikacija (MVP/Pun)

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR6.1 | Registracija sa brojem telefona | MUST | MVP |
| FR6.2 | SMS verifikacija (6-cifreni kod) | MUST | MVP |
| FR6.3 | Kod ističe za 10 minuta | MUST | MVP |
| FR6.4 | Max 3 SMS-a po broju na sat (rate limit) | MUST | MVP |
| FR6.5 | JWT tokeni za sesiju | MUST | MVP |
| FR6.6 | Refresh token mehanizam | SHOULD | Pun |

### FR7: Grupe

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR7.1 | Korisnik može kreirati novu grupu | MUST | MVP |
| FR7.2 | Korisnik može da se pridruži grupi sa PIN-om | MUST | MVP |
| FR7.3 | Admin mora da odobri novog člana | MUST | MVP |
| FR7.4 | Admin može da ukloni člana | MUST | MVP |
| FR7.5 | Admin može da suspenduje člana | SHOULD | MVP |
| FR7.6 | Grupa može imati max 3 admina | SHOULD | Pun |
| FR7.7 | PIN se može regenerisati | SHOULD | MVP |
| FR7.8 | PIN se automatski menja (nedeljno/mesečno) | COULD | Pun |

### FR8: Uloge

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR8.1 | ADMIN uloga - puna kontrola nad grupom | MUST | MVP |
| FR8.2 | RODITELJ uloga - može slati alarme | MUST | MVP |
| FR8.3 | RESPONDER uloga - može preuzimati alarme, ima smene | MUST | MVP |
| FR8.4 | DETE uloga - samo panic button, ograničen prikaz | COULD | Pun |
| FR8.5 | Korisnik može promeniti ulogu (RODITELJ ↔ RESPONDER) | SHOULD | Pun |

### FR9: Smene

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR9.1 | RESPONDER može prijaviti dostupnost po danima/slotovima | MUST | MVP |
| FR9.2 | Pregled nedeljnog rasporeda | MUST | MVP |
| FR9.3 | Upozorenje ako slot ima <2 respondera | SHOULD | MVP |
| FR9.4 | Podsjetnik da popune smene (automatski) | COULD | Pun |
| FR9.5 | Custom vremenski slotovi po grupi | COULD | Pun |

### FR10: Eskalacija

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR10.1 | Ako niko ne preuzme za 90s → notifikacija svim responderima | MUST | MVP |
| FR10.2 | Ako niko ne preuzme za 150s → notifikacija SVIM članovima | SHOULD | Pun |
| FR10.3 | Konfigurabilan timeout eskalacije | COULD | Pun |
| FR10.4 | Automatski poziv admin-u (opciono) | COULD | Pun |

### FR11: Admin Panel

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR11.1 | Pregled svih članova | MUST | Pun |
| FR11.2 | Approve/Reject zahteva za pristup | MUST | MVP |
| FR11.3 | Pregled istorije alarma | MUST | Pun |
| FR11.4 | Podešavanja grupe | SHOULD | Pun |
| FR11.5 | Upravljanje Telegram konekcijom | SHOULD | Pun |
| FR11.6 | Export podataka | COULD | Pun |

### FR12: Statistike

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR12.1 | Broj alarma po periodu | MUST | Pun |
| FR12.2 | Prosečno vreme reakcije | MUST | Pun |
| FR12.3 | Distribucija po vremenu dana | SHOULD | Pun |
| FR12.4 | Mapa "vrućih tačaka" | SHOULD | Pun |
| FR12.5 | Trend analiza | COULD | Pun |
| FR12.6 | Export u PDF/CSV | SHOULD | Pun |

### FR13: Telegram Integracija

| ID | Zahtev | Prioritet | Verzija |
|----|--------|-----------|---------|
| FR13.1 | Bot šalje poruke u grupu | MUST | PoC |
| FR13.2 | Kod-bazirano povezivanje grupe | MUST | MVP |
| FR13.3 | 1:1 mapiranje (jedna TG grupa = jedna app grupa) | MUST | MVP |
| FR13.4 | Detekcija ako je bot uklonjen iz grupe | SHOULD | Pun |
| FR13.5 | Test konekcije komanda | SHOULD | MVP |

---

## Nefunkcionalni Zahtevi

### NFR1: Performanse

| ID | Zahtev | Kriterijum |
|----|--------|------------|
| NFR1.1 | Alarm se šalje u <2 sekunde | Response time |
| NFR1.2 | Telegram notifikacija stiže u <5 sekundi | Latency |
| NFR1.3 | Real-time update u <1 sekunda | WebSocket latency |
| NFR1.4 | Podrška za 100+ istovremenih korisnika | Scalability |
| NFR1.5 | 99.9% uptime | Availability |

### NFR2: Bezbednost

| ID | Zahtev | Opis |
|----|--------|------|
| NFR2.1 | HTTPS za sve komunikacije | TLS 1.3 |
| NFR2.2 | Rate limiting na kritične endpoint-e | DDoS protection |
| NFR2.3 | PIN brute-force zaštita (5 pokušaja) | Account security |
| NFR2.4 | SMS rate limiting (3/sat po broju) | Abuse prevention |
| NFR2.5 | Panic button cooldown (5 min) | Abuse prevention |
| NFR2.6 | JWT expiry (24h) sa refresh | Session security |
| NFR2.7 | Audit log svih admin akcija | Accountability |

### NFR3: Upotrebljivost

| ID | Zahtev | Opis |
|----|--------|------|
| NFR3.1 | Panic button aktivacija u <3 sekunde | Urgency |
| NFR3.2 | Responsivan dizajn (mobile-first) | Accessibility |
| NFR3.3 | Radi na iOS Safari i Android Chrome | Compatibility |
| NFR3.4 | Jednostavan onboarding (<2 minuta) | UX |
| NFR3.5 | Jasan vizuelni feedback za sve akcije | UX |

### NFR4: Pouzdanost

| ID | Zahtev | Opis |
|----|--------|------|
| NFR4.1 | Graceful degradation bez interneta | Offline support |
| NFR4.2 | Automatski retry za failed requests | Reliability |
| NFR4.3 | Error logging za debugging | Maintainability |
| NFR4.4 | Health check endpoint | Monitoring |

### NFR5: Privatnost (GDPR)

| ID | Zahtev | Opis |
|----|--------|------|
| NFR5.1 | Lokacija vidljiva samo članovima grupe | Data privacy |
| NFR5.2 | Mogućnost brisanja naloga i podataka | Right to erasure |
| NFR5.3 | Jasna politika privatnosti | Transparency |
| NFR5.4 | Minimalna poruka u Telegramu (detalji u app) | Data minimization |

### NFR6: Troškovi

| ID | Zahtev | Opis |
|----|--------|------|
| NFR6.1 | PoC: <$5/mesec za hosting | Budget |
| NFR6.2 | MVP: <$20/mesec ukupno | Budget |
| NFR6.3 | Pun sistem: <$100/mesec za 10 škola | Scalability |

---

## Ograničenja

### Tehnička ograničenja

1. **PWA Push Notifikacije** - iOS podrška limitirana (iOS 16.4+)
2. **GPS Tačnost** - Unutra može biti loša (do 50m)
3. **Telegram Rate Limit** - 30 poruka/sekunda po botu
4. **Convex Free Tier** - 1M funkcijskih poziva mesečno

### Poslovna ograničenja

1. **Volonterski projekat** - Ograničeni resursi
2. **Neprofitni karakter** - Bez prihoda
3. **Zavisnost od adopcije** - Sistem funkcioniše samo ako ga koriste

### Pravna ograničenja

1. **Disclaimer** - Sistem ne garantuje bezbednost
2. **Odgovornost** - Roditelji i dalje odgovorni za decu
3. **Privatnost** - GDPR compliance obavezan

---

## MoSCoW Prioriteti

### Must Have (PoC)
- Panic button sa lokacijom
- Telegram notifikacija
- "Preuzimam" funkcionalnost
- PIN autentifikacija
- Osnovna lista alarma

### Should Have (MVP)
- SMS verifikacija
- RESPONDER uloga
- Osnovne smene
- Eskalacija (1 nivo)
- Approval workflow

### Could Have (Pun)
- Admin panel
- Statistike
- 3 nivoa eskalacije
- Geofencing
- DETE uloga
- Istorija i izveštaji

### Won't Have (out of scope)
- Video streaming
- Chat unutar aplikacije
- Integracija sa policijom (automatska)
- Native iOS/Android aplikacije (za sada)

---

*Dokument kreiran: Januar 2026*
