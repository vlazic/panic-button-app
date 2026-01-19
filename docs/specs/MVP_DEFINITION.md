# Definicija MVP - PoC vs MVP vs Pun Sistem

## Pregled varijanti

Sistem ima tri razvojne faze, svaka sa jasno definisanim opsegom:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          RAZVOJNE FAZE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PoC                    MVP                     Pun Sistem                  │
│  (Proof of Concept)     (Minimum Viable)        (Production)               │
│  ─────────────────      ─────────────────       ─────────────               │
│                                                                             │
│  • 1 škola              • 1 škola               • Više škola                │
│  • PIN auth             • SMS auth              • Potpun auth               │
│  • Panic + TG           • + Uloge               • + Admin panel             │
│  • "Preuzimam"          • + Smene               • + Statistike              │
│                         • + Eskalacija (1)      • + Eskalacija (3)          │
│                                                                             │
│  Vreme: ~2 sata         Vreme: 1-2 nedelje      Vreme: 1-2 meseca          │
│  Cilj: Test koncepta    Cilj: Validacija        Cilj: Produkcija           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## PoC (Proof of Concept)

### Cilj
Testirati osnovni koncept sa minimalnim ulaganjem vremena. Odgovoriti na pitanje: **"Da li ovo uopšte radi?"**

### Šta je UKLJUČENO

| Funkcionalnost | Opis |
|----------------|------|
| Panic button | Držanje 3 sekunde aktivira alarm |
| GPS lokacija | Automatsko preuzimanje lokacije |
| Telegram notifikacija | Alarm se šalje u TG grupu |
| "Preuzimam" dugme | Neko može da preuzme alarm |
| Vidljivost statusa | Svi vide ko je preuzeo |
| Lista aktivnih alarma | Osnovna lista u web app-u |

### Šta NIJE uključeno

| Funkcionalnost | Razlog izostavljanja |
|----------------|----------------------|
| Više grupa/škola | Kompleksnost, nepotrebno za test |
| SMS verifikacija | Troškovi, kompleksnost |
| Uloge (ADMIN/RESPONDER/DETE) | Svi su jednaki u PoC |
| Sistem smena | Svi dobijaju sve notifikacije |
| Eskalacija (3 nivoa) | Jednostavnost |
| Admin panel | Ručna administracija |
| Statistike/izveštaji | Kasnije |
| PWA offline | Nice-to-have |
| Korisnički profili | Samo ime u localStorage |
| Approval workflow | Svi sa PIN-om mogu sve |

### Autentifikacija

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  POC AUTENTIFIKACIJA                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • Jedan PIN za celu školu (npr. "kovacic2025")                             │
│  • PIN se deli na roditeljskom sastanku ili lično                           │
│  • Čuva se u localStorage                                                   │
│  • Nema registracije, nema verifikacije                                     │
│  • Korisnik unosi samo ime (opciono)                                        │
│                                                                             │
│  RIZIK: Bilo ko sa PIN-om može sve                                          │
│  MITIGACIJA: PIN se deli samo pouzdanim osobama                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Procenjeno vreme implementacije
**~2 sata** za kompletnu funkcionalnu verziju

---

## MVP (Minimum Viable Product)

### Cilj
Validirati koncept sa pravim korisnicima i prikupiti feedback za dalji razvoj.

### Šta se DODAJE na PoC

| Funkcionalnost | Opis |
|----------------|------|
| SMS verifikacija | Pravi korisnički nalozi |
| RESPONDER uloga | Razlikovanje ko može da reaguje |
| Osnovne smene | Ko je dostupan kada |
| Eskalacija (1 nivo) | Ako niko ne reaguje za 90s → svi |
| Osnovna statistika | Broj alarma, prosečno vreme reakcije |
| Approval workflow | Admin mora da odobri nove članove |

### Šta i dalje NIJE uključeno

| Funkcionalnost | Razlog |
|----------------|--------|
| Više grupa/škola | Fokus na jednu školu |
| Admin panel | Još uvek ručno |
| Kompletan reporting | Osnovna statistika dovoljna |
| DETE uloga | Roditelji koriste za decu |
| Geofencing | Kasnije |
| 3 nivoa eskalacije | 1 nivo dovoljan za MVP |

### Autentifikacija

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MVP AUTENTIFIKACIJA                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. Korisnik unosi broj telefona                                            │
│  2. Dobija SMS kod (6 cifara)                                               │
│  3. Unosi kod → kreira se nalog                                             │
│  4. Unosi PIN grupe da se pridruži                                          │
│  5. Admin odobrava zahtev                                                   │
│                                                                             │
│  PREDNOST: Pravi identiteti, accountability                                 │
│  TROŠAK: ~$0.05 po SMS-u                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Procenjeno vreme implementacije
**1-2 nedelje** za kompletnu verziju

---

## Pun Sistem

### Cilj
Production-ready platforma spremna za skaliranje na više škola.

### Sve funkcionalnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PUN SISTEM - KOMPLETNE FUNKCIONALNOSTI                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  GRUPE I KORISNICI                                                          │
│  ─────────────────                                                          │
│  • Više škola/grupa                                                         │
│  • 4 uloge: ADMIN, RODITELJ, RESPONDER, DETE                                │
│  • Kompletan approval workflow                                              │
│  • Povezivanje roditelj-dete                                                │
│  • Profili sa statistikama                                                  │
│                                                                             │
│  ALARM SISTEM                                                               │
│  ────────────                                                               │
│  • 3 nivoa eskalacije (90s → 60s → broadcast)                               │
│  • Ciljane notifikacije (po blizini, smeni)                                 │
│  • "Preuzimam" sa ETA                                                       │
│  • "Stigao sam" potvrda                                                     │
│  • Rezolucija sa kategorijama                                               │
│                                                                             │
│  RESPONDER SISTEM                                                           │
│  ────────────────                                                           │
│  • Nedeljni raspored smena                                                  │
│  • Podsjetnici za popunjavanje smena                                        │
│  • Statistike po responderu                                                 │
│  • Bystander effect mitigacija                                              │
│                                                                             │
│  ADMIN PANEL                                                                │
│  ───────────                                                                │
│  • Upravljanje članovima                                                    │
│  • Pregled smena                                                            │
│  • Istorija alarma                                                          │
│  • Podešavanja grupe                                                        │
│  • Telegram konekcija                                                       │
│                                                                             │
│  STATISTIKE                                                                 │
│  ──────────                                                                 │
│  • Mesečni izveštaji                                                        │
│  • Mapa "vrućih tačaka"                                                     │
│  • Trend analiza                                                            │
│  • Export za policiju (PDF/CSV)                                             │
│                                                                             │
│  TELEGRAM INTEGRACIJA                                                       │
│  ─────────────────────                                                      │
│  • Kod-bazirano povezivanje                                                 │
│  • 1:1 mapiranje grupa                                                      │
│  • Detekcija uklanjanja bota                                                │
│  • Minimalna poruka + link                                                  │
│                                                                             │
│  BEZBEDNOST                                                                 │
│  ──────────                                                                 │
│  • Rate limiting                                                            │
│  • JWT + refresh tokeni                                                     │
│  • Audit log                                                                │
│  • GDPR compliance                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Procenjeno vreme implementacije
**1-2 meseca** za kompletnu verziju

---

## Preporuka za početak

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          PREPORUČENI PUT                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  NEDELJA 1-2: PoC                                                           │
│  ─────────────────                                                          │
│  • Implementiraj minimalni PoC (~2 sata)                                    │
│  • Testiraj sa 5-10 pouzdanih roditelja                                     │
│  • Prikupi feedback                                                         │
│  • Odluči: nastaviti ili pivotirati?                                        │
│                                                                             │
│  NEDELJA 3-4: MVP                                                           │
│  ─────────────────                                                          │
│  • Dodaj SMS verifikaciju                                                   │
│  • Implementiraj RESPONDER ulogu                                            │
│  • Dodaj osnovne smene                                                      │
│  • Proširi na 20-30 korisnika                                               │
│                                                                             │
│  MESEC 2-3: Pun sistem                                                      │
│  ─────────────────────                                                      │
│  • Admin panel                                                              │
│  • Statistike                                                               │
│  • Stabilizacija                                                            │
│  • Dokumentacija za druge škole                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Kriterijumi uspeha

### PoC
- [ ] Alarm se uspešno šalje i prima
- [ ] Neko može da "preuzme" alarm
- [ ] Real-time update statusa radi
- [ ] 5+ korisnika testiralo bez problema

### MVP
- [ ] SMS verifikacija radi
- [ ] Smene funkcionišu
- [ ] Eskalacija se triggeruje
- [ ] 20+ aktivnih korisnika
- [ ] <1 min prosečno vreme reakcije

### Pun sistem
- [ ] Sve uloge funkcionišu
- [ ] Admin panel kompletan
- [ ] Statistike se generišu
- [ ] 2+ škole koriste sistem
- [ ] Izveštaj predat policiji

---

*Dokument kreiran: Januar 2025*
