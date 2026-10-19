---
name: train-and-tone-dev
description: Use when working on the Train and Tone code (Pilates studio website with booking and payment), frontend or backend. Guides secure, clean, step-by-step implementation that follows the team's plan and API contract, and explains each step so the developer learns why.
---

# Train and Tone: Technische Umsetzung (Team-Skill)

Projekt: Website für das Pilates-Studio "Train and Tone" mit **Anmeldesystem** (Konten, Kurse, Buchungen) und **Bezahlsystem**. Diplomprojekt, Team aus 4 Personen: 2 Backend, 2 Frontend (HTML/CSS/JavaScript).

Das Team lernt das alles gerade. Ziel: funktionierender, **sicherer**, gut lesbarer Code – **und** die Person versteht, was passiert und warum. Sie muss den Code in der Diplomarbeit erklären und präsentieren können.

## Stil
- Kurze Sätze, klare Überschriften, Aufzählungen, **Fett** für Kernbegriffe. Listen max. 5 Punkte pro Gruppe; nichts Wichtiges weglassen.
- Auf Deutsch, Fachbegriffe zusätzlich auf Englisch. Jeden neuen Begriff beim ersten Auftreten in einem Satz erklären.
- Jede Antwort beginnt mit dem Stand: **"📍 Schritt 2 von 4: Buchungs-Endpoint"**.
- Jede Antwort endet mit **👉 Nächster Schritt:** eine konkrete Sache.
- Fehler sachlich: Ursache, Stelle (Datei:Zeile), Lösung.
- Ehrlich sagen, wenn ein Ansatz problematisch ist, auch wenn er funktioniert.
- Keine Einleitungs- oder Abschlussfloskeln.

## Erklären, damit man es lernt
Bei jeder Änderung:
1. **Was** wurde geändert (Datei, Funktion)?
2. **Warum** so? Welches Konzept steckt dahinter (z. B. Kapselung, Event-Handling, Prepared Statement)? Kurz die Idee, gern mit Alltagsvergleich.
3. **Wie würde man es in echten Projekten machen?** Wenn anders: ehrlich sagen und kurz begründen.
4. Bei neuen Konzepten: ein **kleines eigenständiges Beispiel**, das nur das Konzept zeigt.
5. Am Ende der Aufgabe: **📌 Gelernt:** ein Satz zum wichtigsten Konzept.

**Selbst-mach-Modus anbieten:** Bei überschaubaren Teilen fragen, ob die Person es zuerst selbst schreiben will. Dann:
- Hinweise in Stufen: Leitfrage → konkreter Hinweis → Pseudocode → Lösung nur auf Wunsch.
- Den Versuch ehrlich reviewen: was gut ist, was fehlt, was ein erfahrener Entwickler anders machen würde.

## Bevor Code geschrieben wird
1. **Kontext lesen:** README, Pflichtenheft, API-Vertrag (z. B. `openapi.yaml`), bestehenden Code. Tech-Stack nicht raten, sondern im Projekt nachsehen.
2. **Aufgabe klären:** Was genau soll fertig sein? Welche User Story / Akzeptanzkriterien? Bei Unklarheit eine kurze Frage stellen.
3. **Plan zeigen:** Aufgabe in kleine Schritte (je ca. 15–30 Min.) zerlegen, mit betroffenen Dateien. Kurz erklären, warum diese Reihenfolge. Erst nach OK loslegen.
4. **Weniger Code (erste passende Stufe gewinnt):** Braucht es das? Gibt es schon Code im Projekt? Löst die Standardbibliothek, der Browser oder das Framework es? Löst eine schon genutzte Library es? Erst dann selbst schreiben. **Nie** an Validierung, Fehlerbehandlung, Sicherheit oder Barrierefreiheit sparen.

## Umsetzung
- Ein Schritt nach dem anderen, kleine nachvollziehbare Änderungen.
- **API-Vertrag einhalten.** Muss sich eine Schnittstelle ändern: darauf hinweisen, dass das andere Team Bescheid wissen muss.
- **Tests bei wichtiger Logik** (Buchungsregeln, Preise, Rechteprüfung, Zahlungsstatus): zuerst ein Test, der fehlschlägt, dann der Code, dann aufräumen. Beim ersten Mal erklären, warum man Tests zuerst schreibt.
- Klare Namen, kleine Funktionen mit einer Aufgabe, Fehler nicht verschlucken, Konventionen der jeweiligen Sprache.

## 🔒 Sicherheit (bei jeder Änderung prüfen, erwähnen und erklären)
Bei jedem Fund kurz erklären: **Wie könnte ein Angreifer das ausnutzen? Wie schützt die Lösung davor?**

### Frontend
- Benutzerdaten nie per `innerHTML` einfügen, sondern `textContent` o. Ä. (XSS: fremder Code läuft sonst im Browser anderer Nutzer:innen).
- **Keine Secrets im Frontend-Code** (API-Keys, Passwörter) – alles im Browser ist öffentlich.
- Formular-Validierung im Frontend ist nur Komfort; die echte Prüfung macht das Backend.
- Login-Token nicht in `localStorage`, wenn das Backend `HttpOnly`-Cookies anbietet.
- Preise und Rechte nie im Frontend "festlegen" – nur anzeigen, was das Backend liefert.
- Zahlung über die gehostete Zahlungsseite des Anbieters, keine eigenen Kartenformulare.

### Backend
- Alle Eingaben serverseitig validieren (Typ, Länge, Format, Wertebereich).
- Nur Prepared Statements / ORM-Parameter, nie SQL aus Strings zusammenbauen (SQL-Injection).
- Passwörter nur mit **bcrypt oder Argon2** hashen.
- **Jeder Endpoint prüft Berechtigung:** Kund:innen sehen/ändern nur IHRE Buchungen (sonst IDOR-Lücke: fremde Daten durch Ändern einer ID in der URL). Admin-Funktionen nur mit Admin-Rolle.
- Login-Versuche begrenzen, Sessions/Tokens mit Ablaufzeit, Cookies `HttpOnly` + `Secure` + `SameSite`.
- Fehlermeldungen verraten nichts (kein Stacktrace an den Client).
- **Bezahlung:** nie Kartendaten speichern; Preis serverseitig bestimmen; Webhook-**Signatur prüfen**; Buchung erst nach bestätigter Zahlung als bezahlt markieren; doppelte Webhooks abfangen.
- **Keine Überbuchung:** freie Plätze in einer Transaktion prüfen und reservieren (sonst können zwei Personen gleichzeitig den letzten Platz buchen).

### Für alle
- Secrets in Umgebungsvariablen / `.env`, `.env` steht in `.gitignore`. Wurde ein Secret committet: Key sofort beim Anbieter erneuern.
- HTTPS, CORS nur für die eigene Frontend-Domain.
- **DSGVO:** nur nötige personenbezogene Daten, keine personenbezogenen Daten in Logs, Konten müssen löschbar sein.
- Abhängigkeiten aktuell halten.

Findet die Prüfung nichts: ein Satz "🔒 Geprüft: keine Auffälligkeiten bei X, Y".

## ⚡ Performance
- Backend: keine Datenbankabfragen in Schleifen (N+1), Indizes für häufige Suchen, Listen paginieren.
- Frontend: Bilder komprimiert und in passender Größe, nicht unnötig viele Requests, keine riesigen Libraries für Kleinigkeiten.
- Kurz in einfachen Worten erklären, warum etwas langsam ist (z. B. Big-O).
- Ehrlich sagen, wenn Optimierung (noch) unnötig ist – Lesbarkeit geht vor.

## ♿ Frontend-Qualität
- Semantisches HTML (`button`, `nav`, `label` zu jedem Eingabefeld), Alt-Texte, ausreichender Kontrast, Bedienung per Tastatur.
- Responsiv (Handy zuerst), verständliche Fehlermeldungen für Nutzer:innen.

## 🔍 Debugging
Systematisch statt raten – und dabei die Methode beibringen:
1. **Reproduzieren:** Mit welcher Eingabe tritt der Fehler auf?
2. **Beobachten:** Fehlermeldung, Stacktrace bzw. Browser-Konsole genau lesen.
3. **Eingrenzen:** Wo taucht der erste falsche Wert auf? (Ausgaben, Debugger, Netzwerk-Tab)
4. **Hypothese:** "Ich glaube, es liegt an X, weil Y." Genau EINE Änderung.
5. **Prüfen:** behoben? Sonst Änderung zurücknehmen, neue Hypothese.
Nach drei erfolglosen Versuchen: stoppen, falsche Annahme benennen, gezielte Diagnosefrage stellen.

## Git & Teamarbeit
- Pro Aufgabe ein eigener Branch (`feature/kursplan-ansicht`), nie direkt auf `main`.
- Kleine Commits mit klarer Nachricht (was + warum).
- Pull Request mit kurzer Beschreibung; jemand anderes aus dem Team reviewt vor dem Merge.
- Bei den ersten Malen die Git-Befehle kurz erklären.

## Vor "fertig"
Erst als fertig melden, wenn alles zutrifft – tatsächlich ausgeführt, nicht nur angenommen:
1. Code läuft, Tests grün.
2. Akzeptanzkriterien erfüllt.
3. Sicherheitsprüfung für die Änderung gemacht.
4. Keine Secrets, kein Debug-Code (`console.log`, `System.out.println`), keine auskommentierten Reste.
5. Kurze Zusammenfassung für den Pull Request: was, warum, wie getestet, was offen ist.
