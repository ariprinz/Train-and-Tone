# API-Vertrag verstehen – Anleitung fürs Team

Nov 23, 2026 · @Ari

## Was ist ein API-Vertrag?

Ein **API-Vertrag** (API contract) ist eine schriftliche Abmachung zwischen Frontend und Backend. Darin steht: **Welche Anfragen darf das Frontend schicken, und was antwortet das Backend darauf?**

**Vergleich aus dem Alltag:** Der Vertrag ist wie die Speisekarte in einem Restaurant. Der Gast (Frontend) sieht, was er bestellen kann und was er bekommt. Die Küche (Backend) weiß, was sie liefern muss. Wie die Küche kocht, ist dem Gast egal. Hauptsache, das Gericht kommt so, wie es auf der Karte steht.

**Für jeden Endpunkt legt der Vertrag fest:**

- **Methode und Pfad**, z. B. POST /api/v1/ich/buchungen

- **Was mitgeschickt wird** (Request Body), z. B. { "kursterminId": 42 }

- **Was zurückkommt** (Response), z. B. die Buchung als JSON

- **Welche Fehler möglich sind**, z. B. 409 „ausgebucht“

- **Wer ihn nutzen darf**, z. B. nur angemeldete Kund:innen

**Warum das so wichtig ist:** Ohne Vertrag wartet das Frontend aufs Backend, oder beide bauen aneinander vorbei („Ich dachte, das Feld heißt datum“ – „Nein, beginn“). Mit Vertrag arbeiten beide **gleichzeitig**. Am Ende passt alles zusammen, weil beide sich an dasselbe Dokument halten.

Unser Vertrag ist die Datei **openapi.yaml**. **OpenAPI** ist ein weltweiter Standard, um solche Verträge zu schreiben. Viele Werkzeuge können ihn lesen, zum Beispiel der Swagger Editor im nächsten Abschnitt.

## Anschauen im Swagger Editor (5 Min.)

Die YAML-Datei liest sich roh eher mühsam. Der **Swagger Editor** macht daraus eine übersichtliche Seite, auf der man jeden Endpunkt aufklappen kann. Er läuft im Browser, ohne Installation und ohne Konto.

1.  Lade openapi.yaml herunter (aus unserem Chat bzw. Repo) und speichere sie z. B. auf dem Desktop.

2.  Öffne [**editor.swagger.io**](https://editor.swagger.io) im Browser. Links steht ein Beispiel („Petstore“), das ersetzen wir.

3.  Klicke oben im Menü auf **File → Import file** und wähle openapi.yaml. Alternativ ziehst du die Datei einfach ins Browserfenster ([Anleitung](https://codeclimate.com/github/swagger-api/swagger-editor/docs/import.md/source)).

4.  **Links** siehst du jetzt den YAML-Code, **rechts** die fertige Ansicht. Die Endpunkte sind nach Bereichen gruppiert: Öffentlich, Auth, Ich, Trainer, Admin, Stripe.

5.  Klicke rechts auf einen Endpunkt, z. B. **POST /api/v1/ich/buchungen**. Du siehst Beschreibung, Request Body, alle Antworten und Beispiel-JSON. Ganz unten unter **Schemas** stehen alle Datenobjekte, z. B. Kurstermin.

**Gut zu wissen:**

- Der Button **„Try it out“** funktioniert noch nicht, weil unser Backend noch nicht läuft. Zum Lesen braucht ihr ihn nicht.

- Zeigt der Editor eine Warnung zu „OpenAPI 3.1“, öffnet die Datei stattdessen in [**editor-next.swagger.io**](https://editor-next.swagger.io). Das ist die neue Version mit voller 3.1-Unterstützung, die Schritte sind gleich.

- Änderungen im Editor speichert nur euer Browser. Sie landen **nicht** automatisch in unserer Datei. Änderungen bitte immer über Git (siehe Spielregeln).

## So liest du einen Endpunkt

Am Beispiel **„Kurs buchen“** (US-07), stark gekürzt aus openapi.yaml:

/api/v1/ich/buchungen: \# 1. Pfad  
post: \# 2. HTTP-Methode  
summary: Kurstermin mit Guthaben buchen (US-07)  
security: \[ { cookieAuth: \[\], csrfHeader: \[\] } \] \# 3. Wer darf?  
requestBody: \# 4. Was schickt das Frontend?  
content:  
application/json:  
schema:  
required: \[kursterminId\]  
properties:  
kursterminId: { type: integer }  
responses: \# 5. Was kann zurückkommen?  
'201': Gebucht → Buchung als JSON  
'409': bereits-gebucht \| ausgebucht \| kurs-abgesagt  
'422': kein-guthaben

1.  **Pfad:** die Adresse. Alle unsere Pfade beginnen mit /api/v1/.

2.  **Methode:** was passiert. GET = lesen, POST = neu anlegen, PUT = ändern, DELETE = löschen.

3.  **security:** Hier braucht es das Login-Cookie und den CSRF-Header. Bei öffentlichen Endpunkten steht security: \[\].

4.  **requestBody:** das JSON, das das Frontend mitschickt. required = Pflichtfelder.

5.  **responses:** jede mögliche Antwort mit **Statuscode**. 2xx = Erfolg, 4xx = Fehler, den der Client auslöst.

**\$ref verstehen:** Oft steht dort nur \$ref: '#/components/schemas/Buchung'. Das ist ein Verweis wie ein Link: Das Objekt Buchung ist einmal unten unter components beschrieben und wird überall wiederverwendet. Im Swagger Editor klickst du einfach darauf.

**Beispiel-Antwort (201):**

{  
"id": 17,  
"status": "AKTIV",  
"erstelltAm": "2026-12-01T09:12:44Z",  
"kurstermin": {  
"id": 42,  
"name": "Reformer am Abend",  
"beginn": "2027-01-05T17:00:00Z",  
"freiePlaetze": 2,  
"buchbar": true  
}  
}

(gekürzt: kurstermin enthält noch Kurstyp, Standort, Trainer:in usw.)

## So nutzt das Frontend den Vertrag

**Kernidee:** Ihr müsst nicht aufs Backend warten. Ein **Mock-Server** (Attrappe) liest openapi.yaml und antwortet mit Beispieldaten, als gäbe es das Backend schon.

**1. Mock-Server starten (2 Min.)**

npx @stoplight/prism-cli mock openapi.yaml

Danach antwortet http://localhost:4010/api/v1/kurstermine?von=2027-01-04&bis=2027-01-10 mit Beispiel-JSON. Später tauscht ihr nur die Basis-URL gegen das echte Backend aus.

**Tipp zum Login im Mock:** Prism prüft, ob Login-Cookie und CSRF-Header vorhanden sind, aber nicht, ob sie echt sind. Sonst kommt 401. Zum Testen reicht im Browser einmal document.cookie = 'TrainTone.Auth=test' und als Header ein beliebiger Wert, z. B. X-XSRF-TOKEN: test.

**2. Anfragen schicken (fetch)**

// Einmal beim Start: CSRF-Token holen  
const { token } = await (await fetch('/api/v1/auth/csrf', { credentials: 'include' })).json();  
  
// Kurs buchen  
const res = await fetch('/api/v1/ich/buchungen', {  
method: 'POST',  
credentials: 'include', // Login-Cookie mitschicken  
headers: {  
'Content-Type': 'application/json',  
'X-XSRF-TOKEN': token // Pflicht bei POST/PUT/DELETE  
},  
body: JSON.stringify({ kursterminId: 42 })  
});

**3. Fehler behandeln**

Bei einem Fehler kommt immer ein ProblemDetails-Objekt. Entscheidet anhand von status und type:

if (!res.ok) {  
const problem = await res.json();  
if (problem.type?.endsWith('/kein-guthaben')) zeigeLinkZuProdukten();  
else zeigeMeldung(problem.title); // z. B. "Kurs ist ausgebucht"  
}

**4. Drei Regeln fürs Frontend**

- **Uhrzeiten immer in Wiener Zeit anzeigen:** new Date(t.beginn).toLocaleString('de-AT', { timeZone: 'Europe/Vienna' }). Die API liefert UTC.

- **Geld in Cent:** 25000 heißt 250,00 €. Zum Anzeigen durch 100 teilen und mit Intl.NumberFormat formatieren.

- **Buttons sperren**, solange eine Anfrage läuft, damit kein Doppelklick passiert.

**Optional:** npx openapi-typescript openapi.yaml -o api-typen.ts erzeugt TypeScript-Typen aus dem Vertrag. Dann warnt der Editor bei Tippfehlern in Feldnamen.

## So nutzt das Backend den Vertrag

**Kernidee:** Für das Backend ist der Vertrag eine **Bauanleitung und ein Versprechen**. Jeder Endpunkt muss genau so antworten, wie es dort steht: gleiche Feldnamen, gleiche Statuscodes.

**1. Vom Vertrag zum Code (ASP.NET Core)**

- Jeder Bereich wird ein Controller: /ich/… → IchController, /trainer/… → TrainerController, /admin/… → AdminController.

- Jedes Schema wird eine **DTO-Klasse** (Data Transfer Object), z. B. Kurstermin → KursterminDto. DTOs sind **nicht** dieselben Klassen wie die Datenbank-Entities. So landet z. B. der Passwort-Hash nie in einer Antwort.

- Die Rollen prüft ASP.NET Core mit \[Authorize(Roles = "ADMIN")\] am Controller.

- Die Fachlogik (z. B. „Termin bearbeiten“) liegt in **Services**. Trainer- und Admin-Controller rufen denselben Service auf, damit die Regeln nur einmal existieren.

**2. Swagger UI zum Testen**

ASP.NET Core erzeugt aus dem Code automatisch eine Swagger-Seite unter /swagger. Dort kann man jeden Endpunkt mit „Try it out“ direkt ausprobieren.

**3. Vertrag und Code vergleichen**

Vor jedem Pull Request prüfen: Stimmt die Swagger-Seite aus dem Code mit openapi.yaml überein (Pfade, Felder, Statuscodes)? Weicht etwas ab, ist entweder der Code falsch, oder der Vertrag muss bewusst geändert werden (siehe Spielregeln).

**4. Was nur das Backend sieht**

Der Stripe-Webhook (/stripe/webhook) ist nicht fürs Frontend. Stripe ruft ihn auf, wenn eine Zahlung bestätigt ist. Das Backend prüft dann die Signatur und schreibt das Guthaben gut.

## Unsere Entscheidungen erklärt

Ari hat diese Punkte am 23.11.2026 festgelegt. Hier steht jeweils, **was** gilt, und **warum** das sinnvoll ist.

**Sprache und Form**

- **Deutsche Pfade und Felder** (/kurstermine, maxTeilnehmer): Wir verwenden dieselben Begriffe wie im Datenmodell, in den User Stories und im Domain-Storytelling-Diagramm. Niemand muss überlegen, ob „Session“ jetzt „Kurstermin“ bedeutet.

- **Versionsnummer /api/v1/:** Wenn wir später etwas grundlegend ändern, kann es parallel eine v2 geben, ohne dass Altes kaputtgeht. Das kostet uns nichts und ist in Firmen üblich.

- **Zeiten in UTC** (…Z am Ende): Die Datenbank speichert genauso. Bei der Sommerzeit-Umstellung gibt es so keine falschen Uhrzeiten. Die Umrechnung in Wiener Zeit macht nur das Frontend beim Anzeigen.

- **Fehler als ProblemDetails:** Ein offizieller Standard. Jeder Fehler hat dieselbe Form, also braucht das Frontend nur **eine** Fehlerbehandlung für alles.

**Sicherheit**

- **Login per HttpOnly-Cookie statt Token im Browser-Speicher:** JavaScript kann dieses Cookie nicht lesen. Selbst wenn sich schädlicher Code auf die Seite schleicht (XSS), kann er das Login nicht stehlen.

- **CSRF-Schutz (SameSite + X-XSRF-TOKEN):** Cookies schickt der Browser automatisch mit. Eine böse Website könnte sonst im Namen einer eingeloggten Kundin buchen. Das Token beweist, dass die Anfrage wirklich von unserer Seite kommt. Bei einem Bezahlsystem nehmen wir den doppelten Schutz.

- **Eigene Daten über /ich/…:** Das Backend nimmt die Person aus dem Cookie, nicht aus der URL. Niemand kann /benutzer/43/buchungen eintippen und fremde Buchungen sehen. Diese Lücke heißt IDOR und ist eine der häufigsten in echten Apps.

- **Öffentlich nur der Vorname der Trainer:in:** DSGVO-Grundsatz der Datenminimierung. E-Mail und Telefon stehen nie in öffentlichen Antworten.

**Ablauf und Umfang**

- **Backend wählt das Guthaben:** Beim Buchen schickt das Frontend nur die Termin-ID. Das Backend nimmt den passenden Block, der zuerst abläuft. So verfällt am wenigsten, und das Frontend braucht keine Auswahl.

- **Zahlungsstatus abfragen statt dem Redirect vertrauen:** Die Erfolgs-URL von Stripe kann jede:r einfach aufrufen. Nur die Meldung von Stripe direkt ans Backend (Webhook) ist ein echter Beweis.

- **Doppelbuchung = 409:** Klare Fehlermeldung. Zusätzlich ist der Button während der Anfrage gesperrt, und die Datenbank verhindert es sowieso.

- **Rollen-Bereiche /trainer/ und /admin/:** Man sieht am Pfad sofort, wer etwas darf, und kann ganze Bereiche auf einmal schützen.

- **Kalender mit von/bis (max. 31 Tage):** Passt zum Wochen-Blättern. Freie Plätze rechnet das Backend aus, damit sie immer stimmen.

- **Serien im MVP nicht bearbeitbar:** Eine Serie zu ändern, die schon Buchungen hat, ist kompliziert. Fürs MVP reicht: einzelne Termine bearbeiten oder Serie beenden und neu anlegen.

- **Nur das MVP ist ausgearbeitet, keine Paginierung:** Wir legen nichts fest, was wir noch nicht genau kennen. Listen sind im MVP klein genug.

## Spielregeln für Änderungen

1.  **Der Vertrag gewinnt.** Wenn Code und openapi.yaml nicht zusammenpassen, gilt die YAML, bis wir sie bewusst ändern.

2.  **Änderungen nur per Pull Request** mit kurzer Begründung. Mindestens eine Person aus Frontend **und** eine aus Backend stimmen zu.

3.  **Nichts heimlich umbenennen.** Ein geändertes Feld bricht die andere Seite. Lieber im Team-Chat kurz fragen.

4.  **Unklar? Fragen statt raten.** Gerade an der Schnittstelle Frontend ↔ Backend spart eine Frage oft Stunden.

5.  **Fragen und Ideen als Kommentar** direkt in diesem Doc oder im Doc „Train and Tone – API-Vertrag“.

**Mehr Details:** Alle Endpunkte als Tabellen, die Fehlercodes und den Ablauf des Stripe-Kaufs findet ihr im Doc „Train and Tone – API-Vertrag“.
