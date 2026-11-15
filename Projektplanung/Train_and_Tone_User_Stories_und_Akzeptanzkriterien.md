# Train and Tone – User Stories und Akzeptanzkriterien

## Verwaltung

### US-01 Kurstypen und Produkte anlegen · 5 Punkte

Als **Admin** möchte ich **Kurstypen und Produkte anlegen**, damit Kund:innen Produkte kaufen und damit Kurse buchen können.

- Gegeben ich bin angemeldet, wenn ich einen Kurstyp mit Namen speichere, dann ist er bei Produkten und Kursen auswählbar.
- Gegeben ein Kurstyp mit diesem Namen existiert, wenn ich denselben Namen speichere, dann wird nicht gespeichert und ich sehe einen Hinweis.
- Gegeben es gibt mindestens einen Kurstyp, wenn ich Name, Preis, Anzahl Einheiten (1, 5 oder 10), Gültigkeit in Tagen und gültige Kurstypen eingebe und speichere, dann ist das Produkt gespeichert und ich sehe eine Bestätigung.
- Gegeben ein Pflichtfeld fehlt oder der Preis ist kleiner oder gleich 0, wenn ich speichere, dann wird nicht gespeichert und das Feld ist markiert.

### US-02 Trainer-Konto anlegen · 3 Punkte

Als **Admin** möchte ich **Konten für Trainer:innen anlegen**, damit sie Kurse planen können.

- Gegeben ich bin angemeldet, wenn ich Name, E-Mail, Telefonnummer und ein Startpasswort eingebe, dann wird ein Konto mit der Rolle Trainer:in angelegt.
- Gegeben die E-Mail ist bereits vergeben, wenn ich speichere, dann wird kein Konto angelegt und ich sehe einen Hinweis.
- Gegeben eine Trainer:in meldet sich zum ersten Mal an, dann muss sie ein eigenes Passwort festlegen.

### US-03 Kurse anlegen und bearbeiten · 8 Punkte

Als **Trainer:in** möchte ich **meine Kurse anlegen und bearbeiten**, damit Kund:innen sie buchen können und Änderungen sichtbar sind.

- Gegeben ich bin angemeldet, wenn ich Name, Kurstyp, Standort, Datum, Startzeit, Dauer und maximale Teilnehmerzahl eingebe und speichere, dann ist der Kurs mir zugeordnet, gespeichert und im Kalender sichtbar.
- Gegeben ein Pflichtfeld fehlt oder das Datum liegt in der Vergangenheit, wenn ich speichere, dann wird nicht gespeichert und ich sehe einen Hinweis.
- Gegeben ich ändere einen eigenen Kurs, wenn ich speichere, dann sind die Änderungen im Kalender sichtbar.
- Gegeben der Kurs hat Buchungen, wenn ich die maximale Teilnehmerzahl unter die Zahl der Buchungen setze, dann wird nicht gespeichert.
- Gegeben der Kurs gehört einer anderen Trainer:in, dann kann ich ihn nicht bearbeiten. Der Admin kann alle Kurse bearbeiten.

## Konto und Kalender

### US-04 Registrieren und anmelden · 8 Punkte

Als **Kund:in** möchte ich **mich registrieren und anmelden**, damit ich Produkte kaufen und Kurse buchen kann. Die Anmeldung gilt gleich für Trainer:innen und Admin.

- Gegeben ich bin nicht angemeldet, wenn ich Name, E-Mail, Telefonnummer und ein Passwort mit mindestens 8 Zeichen eingebe, dann wird mein Konto angelegt und ich bin angemeldet.
- Gegeben die E-Mail ist bereits vergeben, wenn ich mich registriere, dann wird kein Konto angelegt und ich sehe einen Hinweis.
- Gegeben mein Konto existiert, wenn ich korrekte Anmeldedaten eingebe, dann bin ich angemeldet und sehe nur die Funktionen meiner Rolle.
- Gegeben ich gebe falsche Anmeldedaten ein, dann sehe ich eine allgemeine Fehlermeldung, die nicht verrät, ob die E-Mail existiert.
- Gegeben ich habe nicht die nötige Rolle, wenn ich eine Funktion trotzdem aufrufe (z. B. direkt über die API), dann wird der Zugriff verweigert.

### US-05 Kurskalender ansehen · 5 Punkte

Als **Besucher:in oder Kund:in** möchte ich **den Kurskalender ansehen**, damit ich einen passenden Kurs finde.

- Gegeben es gibt geplante Kurse, wenn ich den Kalender öffne (auch ohne Anmeldung), dann sehe ich pro Kurs Name, Kurstyp, Datum, Uhrzeit, Trainer:in, Standort und freie Plätze.
- Gegeben ein Kurs ist voll oder abgesagt, dann ist der Status erkennbar und der Kurs nicht buchbar.
- Gegeben ich bin in der aktuellen Woche, wenn ich blättere, dann sehe ich die Kurse der vorherigen oder nächsten Woche.
- Gegeben ich wähle einen Kurs, dann sehe ich seine Details.

## Kaufen und Buchen

### US-06 Produkt kaufen und online bezahlen · 13 Punkte

Als **Kund:in** möchte ich **ein Produkt kaufen und online per Karte bezahlen**, damit ich Guthaben für Buchungen habe.

- Gegeben ich bin angemeldet, wenn ich ein Produkt wähle, dann sehe ich den Preis und werde zur Zahlungsseite von Stripe weitergeleitet.
- Gegeben die Zahlung ist erfolgreich, dann wird das Guthaben gutgeschrieben, sobald Stripe die Zahlung bestätigt, und ich sehe eine Bestätigung.
- Gegeben die Zahlung wird abgelehnt oder abgebrochen, dann bekomme ich kein Guthaben und kann den Kauf erneut starten.
- Gegeben dieselbe Zahlung wird mehrfach gemeldet oder abgesendet, dann wird nur einmal abgebucht und nur einmal gutgeschrieben.
- Meine Kartendaten gebe ich nur bei Stripe ein; das System speichert sie nicht.

### US-07 Kurs mit Guthaben buchen · 13 Punkte

Als **Kund:in** möchte ich **mit meinem Guthaben einen Platz in einem Kurs buchen**, damit mir der Platz sicher ist.

- Gegeben ich habe gültiges Guthaben für den Kurstyp und der Kurs ist frei, wenn ich buche und bestätige, dann ist die Buchung gespeichert, 1 Einheit wird abgezogen und die freien Plätze sinken um 1.
- Gegeben ich habe kein passendes oder nur abgelaufenes Guthaben, wenn ich „Buchen“ wähle, dann sehe ich einen Hinweis und einen Link zu den Produkten.
- Gegeben der Kurs ist voll oder abgesagt, dann kann ich nicht buchen.
- Gegeben ich habe den Kurs bereits gebucht, wenn ich erneut absende (z. B. Doppelklick), dann entsteht keine zweite Buchung und es wird keine zweite Einheit abgezogen.
- Gegeben nur 1 Platz ist frei, wenn zwei Kund:innen gleichzeitig buchen, dann ist genau eine Buchung erfolgreich.

### US-08 Guthaben und Buchungen ansehen · 3 Punkte

Als **Kund:in** möchte ich **mein Guthaben und meine Buchungen sehen**, damit ich weiß, was ich noch buchen kann und wann meine Kurse sind.

- Gegeben ich bin angemeldet, wenn ich „Mein Konto“ öffne, dann sehe ich mein Guthaben pro Produkt mit Resteinheiten und Ablaufdatum.
- Gegeben ich habe Buchungen, dann sehe ich meine kommenden Kurse mit Datum, Uhrzeit und Standort.

## Backlog und Won't

Diese Punkte sind noch nicht als User Story ausgeschrieben.

**Release 2**

- Kurstypen und Produkte bearbeiten (Änderungen gelten nur für neue Käufe)
- Kurs absagen (Gebuchte bekommen Einheit zurück und eine E-Mail)
- Passwort zurücksetzen
- Probestunde kaufen (einmal pro Kund:in)
- Mehrere Plätze auf einmal buchen
- Bestätigungs-E-Mail nach Kauf und Buchung

**Release 3**

- Buchung stornieren bis 12 Stunden vorher (Einheit wird gutgeschrieben)
- Teilnehmerliste für Trainer:innen
- Profil bearbeiten und Konto löschen (DSGVO)
- Kalender nach Kurstyp, Trainer:in und Standort filtern

**Won't (nicht in der Diplomarbeit)**

- Abos (monatlich wiederkehrende Zahlungen)
- Rückerstattung in Geld im System (macht der Admin bei Bedarf direkt in Stripe)
- Bezahlung vor Ort, weitere Zahlarten außer Stripe (Karte, Klarna)
- Admin legt Kund:innen an oder ändert Rollen
- Dashboard, Auslastungs- und Finanzstatistiken
- Native App
