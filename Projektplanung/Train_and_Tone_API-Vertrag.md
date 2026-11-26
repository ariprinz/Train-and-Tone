# Train and Tone – API-Vertrag

Nov 23, 2026 · @Ari

## Überblick

Der API-Vertrag (API contract) legt **alle MVP-Endpunkte (US-01 bis US-08)** fest: Pfade, JSON-Daten, Rollen und Fehlercodes. Er ist **Entwurf v0.1** und baut auf dem Dokument „Train and Tone – Datenmodell“ auf.

**Der eigentliche Vertrag ist die Datei openapi.yaml** (OpenAPI 3.1). Dieses Doc erklärt sie fürs Team und den Lehrer. Bei Widersprüchen gilt die YAML.

**So arbeiten wir damit:**

1.  Alle vier lesen den Vertrag und kommentieren hier im Doc (20 Min.).

2.  Im Team-Meeting klären wir offene Punkte und setzen die Version auf v1.0.

3.  **Frontend** startet mit einem Mock-Server aus der YAML, z. B. npx @stoplight/prism-cli mock openapi.yaml, und muss nicht auf das Backend warten.

4.  **Backend** baut die Endpunkte in ASP.NET Core. Swagger UI zeigt sie unter /swagger an.

5.  Änderungen am Vertrag nur per Pull Request, in dem beide Seiten zustimmen.

## Grundsatz-Entscheidungen

Diese 15 Entscheidungen hat Ari am 23.11.2026 getroffen. Sie gelten für jeden Endpunkt.

| Thema                | Entscheidung                                                                    | Warum                                                                       |
|----------------------|---------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| Format               | OpenAPI 3.1 (openapi.yaml) + dieses Doc                                         | Industriestandard, Swagger UI in ASP.NET Core, Mocks fürs Frontend          |
| Sprache              | Deutsche Pfade und Felder (camelCase)                                           | Gleiche Begriffe wie Datenmodell und Domain Story, keine Übersetzungsfehler |
| Version              | Alle Pfade beginnen mit /api/v1/                                                | Spätere Änderungen ohne alte Clients zu brechen                             |
| Login                | HttpOnly-Cookie (ASP.NET Core Identity)                                         | JavaScript kann das Cookie nicht lesen → Schutz gegen XSS                   |
| CSRF                 | Cookie mit SameSite=Strict + Header X-XSRF-TOKEN                                | Doppelter Schutz, wichtig bei einem Bezahlsystem                            |
| Fehler               | ProblemDetails (RFC 9457)                                                       | Standard, in ASP.NET Core eingebaut, gleiche Struktur für jeden Fehler      |
| Datum/Zeit           | ISO 8601 in UTC, z. B. 2027-01-05T17:00:00Z                                     | Passt zu TIMESTAMPTZ; Frontend zeigt mit timeZone: 'Europe/Vienna' an       |
| Rollen-Pfade         | /ich/, /trainer/, /admin/                                                       | Jeder Bereich hat eine klare Rolle, Rechte leicht zu prüfen                 |
| Eigene Daten         | Über /api/v1/ich/…, Person aus dem Cookie                                       | Keine fremden IDs in der URL möglich (Schutz gegen IDOR)                    |
| Guthaben beim Buchen | Backend wählt automatisch den Block, der zuerst abläuft                         | Weniger verfallenes Guthaben, einfacheres Frontend                          |
| Serie ändern         | Im MVP nicht bearbeitbar, nur Einzeltermine + „Serie beenden“                   | Spart viel Zeit, keine Konflikte mit Buchungen                              |
| Kalender             | ?von=&bis=, max. 31 Tage, inkl. freiePlaetze und buchbar                        | Passt zum Wochen-Blättern (US-05), Frontend rechnet nichts                  |
| Zahlungsstatus       | Erfolgsseite fragt den Kauf-Status ab (alle 2 s, max. 30 s)                     | Nur der Stripe-Webhook ist ein Beweis, der Redirect nicht                   |
| Doppelbuchung        | 409 Conflict + Button während der Anfrage gesperrt                              | Eindeutig; Datenbank verhindert es zusätzlich                               |
| Umfang & Datenschutz | Nur MVP ausgearbeitet; öffentlich nur Vorname der Trainer:in; keine Paginierung | Überschaubar; DSGVO-Datenminimierung                                        |

## Endpunkte

29 Endpunkte, alle mit dem Präfix /api/v1. **Login** = Auth-Cookie nötig. Jede POST/PUT-Anfrage braucht zusätzlich den Header X-XSRF-TOKEN (außer beim Stripe-Webhook).

**Öffentlich (ohne Login)**

| Methode | Pfad                   | Zweck                            | Story |
|---------|------------------------|----------------------------------|-------|
| GET     | /kurstermine?von=&bis= | Kalender mit freien Plätzen      | US-05 |
| GET     | /kurstermine/{id}      | Details eines Termins            | US-05 |
| GET     | /produkte              | Aktive Produkte für den Shop     | US-06 |
| GET     | /kurstypen, /standorte | Listen für Anzeige und Formulare | –     |

**Auth**

| Methode | Pfad               | Zweck                                                               | Story |
|---------|--------------------|---------------------------------------------------------------------|-------|
| GET     | /auth/csrf         | CSRF-Token holen                                                    | –     |
| POST    | /auth/registrieren | Konto anlegen + direkt angemeldet (201, 409 bei vergebener E-Mail)  | US-04 |
| POST    | /auth/anmelden     | Anmelden (401 mit allgemeiner Meldung, 429 bei zu vielen Versuchen) | US-04 |
| POST    | /auth/abmelden     | Abmelden                                                            | US-04 |

**Ich (angemeldete Person)** – Kauf, Guthaben und Buchung nur für Rolle KUNDE

| Methode | Pfad                            | Zweck                                                | Story |
|---------|---------------------------------|------------------------------------------------------|-------|
| GET     | /ich                            | Eigene Daten + Rolle (für das Menü)                  | US-04 |
| PUT     | /ich/passwort                   | Passwort ändern (Pflicht beim 1. Trainer-Login)      | US-02 |
| GET     | /ich/guthaben                   | Gültige Guthaben-Blöcke                              | US-08 |
| GET     | /ich/buchungen?zeitraum=kommend | Eigene Buchungen                                     | US-08 |
| POST    | /ich/buchungen                  | Buchen, Body nur { kursterminId }                    | US-07 |
| POST    | /ich/kaeufe                     | Kauf starten, Antwort enthält checkoutUrl von Stripe | US-06 |
| GET     | /ich/kaeufe/{id}                | Status des Kaufs (Erfolgsseite fragt ab)             | US-06 |

**Trainer:in** – nur eigene Kurse

| Methode | Pfad                              | Zweck                                                   | Story |
|---------|-----------------------------------|---------------------------------------------------------|-------|
| GET     | /trainer/kurstermine?von=&bis=    | Eigene Termine                                          | US-03 |
| POST    | /trainer/kurstermine              | Einzeltermin anlegen                                    | US-03 |
| PUT     | /trainer/kurstermine/{id}         | Eigenen Termin bearbeiten (403 bei fremdem Termin)      | US-03 |
| GET     | /trainer/kurs-serien              | Eigene Serien                                           | US-03 |
| POST    | /trainer/kurs-serien              | Wöchentliche Serie anlegen, Backend erzeugt die Termine | US-03 |
| POST    | /trainer/kurs-serien/{id}/beenden | Serie vorzeitig beenden                                 | US-03 |

**Admin**

| Methode   | Pfad                    | Zweck                                             | Story |
|-----------|-------------------------|---------------------------------------------------|-------|
| POST      | /admin/kurstypen        | Kurstyp anlegen (409 bei gleichem Namen)          | US-01 |
| GET, POST | /admin/produkte         | Alle Produkte sehen, Produkt anlegen              | US-01 |
| GET, POST | /admin/trainer          | Trainer:innen sehen, Trainer-Konto anlegen        | US-02 |
| PUT       | /admin/kurstermine/{id} | Jeden Termin bearbeiten, auch Trainer:in wechseln | US-03 |

**Stripe (nicht fürs Frontend)**

| Methode | Pfad            | Zweck                                                                               | Story |
|---------|-----------------|-------------------------------------------------------------------------------------|-------|
| POST    | /stripe/webhook | Stripe meldet die Zahlung; Backend prüft die Signatur und schreibt das Guthaben gut | US-06 |

## Fehler und Statuscodes

Jeder Fehler kommt als ProblemDetails. Das Frontend entscheidet anhand von **status** und bei fachlichen Fehlern zusätzlich anhand von **type**, welche Meldung es zeigt.

{  
"type": "https://traintone.at/probleme/ausgebucht",  
"title": "Kurs ist ausgebucht",  
"status": 409,  
"detail": "Für diesen Termin ist kein Platz mehr frei."  
}

| Status | Bedeutung                                              | Wann                                                                                                          |
|--------|--------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| 400    | Validierungsfehler, errors enthält die Fehler pro Feld | Pflichtfeld fehlt, Preis ≤ 0, Datum in der Vergangenheit, Zeitraum \> 31 Tage                                 |
| 401    | Nicht angemeldet                                       | Cookie fehlt oder ist abgelaufen; bei falschem Login immer dieselbe Meldung                                   |
| 403    | Keine Berechtigung                                     | Falsche Rolle oder fremder Kurstermin                                                                         |
| 404    | Nicht gefunden                                         | Auch bei einem fremden Kauf, damit niemand erfährt, dass die ID existiert                                     |
| 409    | Konflikt                                               | bereits-gebucht, ausgebucht, kurs-abgesagt, kurs-vorbei, email-vergeben, name-vergeben, termine-mit-buchungen |
| 422    | Fachlich nicht möglich                                 | kein-guthaben (Frontend zeigt Link zu den Produkten), max-teilnehmer-zu-klein                                 |
| 429    | Zu viele Anfragen                                      | Zu viele Login-Versuche (Vorschlag: mehr als 5 pro Minute) (Schutz vor Passwort-Raten)                        |

## Ablauf: Produkt kaufen mit Stripe

Guthaben entsteht **nur durch den Stripe-Webhook**, nie durch den Redirect zurück ins Frontend.

sequenceDiagram  
autonumber  
actor K as Kund:in  
participant F as Frontend (React)  
participant B as Backend (.NET)  
participant S as Stripe  
K-\>\>F: Produkt wählen, "Kaufen"  
F-\>\>B: POST /api/v1/ich/kaeufe { produktId }  
B-\>\>S: Checkout Session anlegen  
S--\>\>B: sessionId + URL  
B--\>\>F: 201 { kaufId, checkoutUrl }  
F-\>\>S: Weiterleitung zur Zahlungsseite  
K-\>\>S: Karte eingeben, bezahlen  
S-\>\>B: POST /api/v1/stripe/webhook (signiert)  
B-\>\>B: Signatur prüfen, Kauf = BEZAHLT, Guthaben anlegen  
S--\>\>F: Redirect zur Erfolgsseite (?kaufId=…)  
loop alle 2 s, max. 30 s  
F-\>\>B: GET /api/v1/ich/kaeufe/{kaufId}  
B--\>\>F: status  
end  
F--\>\>K: "Bezahlt, Guthaben verfügbar"

Der Webhook kann vor oder nach dem Redirect ankommen. Deshalb fragt die Erfolgsseite den Status ab, statt sofort „bezahlt“ anzuzeigen. Kommt nach 30 s kein BEZAHLT, zeigt sie: „Zahlung wird noch geprüft, dein Guthaben erscheint in Kürze unter Mein Konto.“

## Ausblick und offene Fragen

**Spätere Releases** (noch nicht im Vertrag, nur geplant):

- Release 2: Kurstypen/Produkte bearbeiten, Kurstermin absagen (Einheit zurück + E-Mail), Passwort zurücksetzen, First Timer nur einmal kaufbar, mehrere Plätze buchen, Bestätigungs-E-Mails

- Release 3: Buchung bis 12 h vorher stornieren, Teilnehmerliste für Trainer:innen, Profil bearbeiten und Konto löschen, Kalender-Filter nach Kurstyp, Trainer:in und Standort

**Offene Fragen ans Team:**

- [ ] Standorte: Legt sie nur der Seed beim ersten Start an (3 fixe Studios), oder braucht der Admin POST /admin/standorte?

- [ ] Rate Limit beim Login: Vorschlag 5 Versuche pro Minute. Passt das?

- [ ] Registrieren meldet „E-Mail bereits vergeben“ (so verlangt es US-04). Das verrät, wer ein Konto hat. Nehmen wir das bewusst in Kauf?

- [ ] Laufen Frontend und API in Produktion unter derselben Domain? Das ist nötig für das SameSite=Strict-Cookie. Lokal löst das ein Vite-Proxy.

- [ ] Wer pflegt openapi.yaml, und wer reviewt Änderungen?
