# Datenmodell verstehen – Anleitung fürs Team

Nov 23, 2026 · @Ari

## Was ist ein Datenmodell?

Ein **Datenmodell** (data model) beschreibt, **welche Daten unser System speichert und wie sie zusammenhängen**. Es ist der Bauplan für die Datenbank.

**Vergleich aus dem Alltag:** Stellt euch das Studio mit Karteikästen vor. Ein Kasten für Kund:innen, einer für Kurse, einer für Buchungen. Jede Karte hat feste Felder (Name, E-Mail …). Auf einer Buchungskarte steht nicht noch einmal die ganze Kundin, sondern nur ihre Kartennummer. Genau so funktioniert eine relationale Datenbank:

- **Kasten** = Tabelle (z. B. benutzer)

- **Karte** = Zeile, also ein Datensatz (z. B. Kim)

- **Feld** = Spalte (z. B. email)

- **Kartennummer** = Primärschlüssel id

- **Verweis auf eine andere Karte** = Fremdschlüssel (z. B. buchung.benutzer_id)

**Stand:** Entwurf v0.1 vom 23.11.2026 mit 10 Tabellen. Als Datenbank schlagen wir **PostgreSQL** vor, die Freigabe durch den Lehrer steht noch aus. Das Modell selbst gilt unabhängig davon.

**Wie hängt das mit der API zusammen?** Das Datenmodell beschreibt, was **gespeichert** wird. Der API-Vertrag beschreibt, was **nach außen** gezeigt wird. Das ist oft weniger: Den Passwort-Hash gibt es zum Beispiel nur in der Datenbank.

## So liest du das ER-Diagramm

Das Bild unten ist ein **ER-Diagramm** (Entity-Relationship). Jeder Kasten ist eine Tabelle, jede Linie eine Beziehung zwischen zwei Tabellen.

<img src="media/image1.png" style="width:6in;height:5.76428in" alt="ER-Diagramm Train and Tone, Datenmodell v0.1 mit 10 Tabellen" />

**In jedem Kasten** steht pro Zeile eine Spalte: links der Datentyp (z. B. bigint, varchar), dann der Name, dann ein Kürzel und ganz rechts eine Regel.

| Kürzel | Bedeutung                                                                     | Beispiel                          |
|--------|-------------------------------------------------------------------------------|-----------------------------------|
| **PK** | Primärschlüssel (primary key): die eindeutige Nummer jeder Zeile              | benutzer.id                       |
| **FK** | Fremdschlüssel (foreign key): Verweis auf eine Zeile in einer anderen Tabelle | buchung.benutzer_id → benutzer.id |
| **UK** | Eindeutig (unique key): derselbe Wert darf nur einmal vorkommen               | benutzer.email                    |

**An den Linien** steht, wie viele Zeilen zusammengehören (Kardinalität):

| Beschriftung | Bedeutung                                | Beispiel aus unserem Modell                                                   |
|--------------|------------------------------------------|-------------------------------------------------------------------------------|
| **1:n**      | Eine Zeile links gehört zu vielen rechts | Eine Kundin hat viele Buchungen, jede Buchung gehört genau einer Kundin       |
| **1:1**      | Genau eine zu höchstens einer            | Ein Kauf ergibt genau ein Guthaben                                            |
| **n:m**      | Viele zu vielen                          | Ein Produkt gilt für mehrere Kurstypen, ein Kurstyp ist in mehreren Produkten |

**Die Linien-Enden (Krähenfuß-Notation, crow's foot):** \|\| heißt genau eins, o\| null oder eins, o{ null oder viele. Der „Krähenfuß“ (die drei Striche) steht immer auf der „viele“-Seite.

**Wichtig zu n:m:** Eine Datenbank kann n:m nicht direkt speichern. Man löst es mit einer **Zwischentabelle** in zwei 1:n-Beziehungen auf. Bei uns ist das produkt_kurstyp: Jede Zeile sagt „Produkt X gilt für Kurstyp Y“.

## Die 10 Tabellen in einfachen Worten

| Bereich  | Tabelle         | Was steht drin?                                                                                                            |
|----------|-----------------|----------------------------------------------------------------------------------------------------------------------------|
| Personen | benutzer        | Alle Konten: Kund:innen, Trainer:innen und Admin. Die Spalte rolle sagt, wer was ist.                                      |
| Angebot  | standort        | Die Studios (aktuell 3) mit Adresse                                                                                        |
| Angebot  | kurstyp         | Arten von Kursen, z. B. Mat oder Reformer                                                                                  |
| Angebot  | produkt         | Was man kaufen kann: First Timer, Single Class, Block 3, Block 10. Mit Preis, Einheiten und Gültigkeit in Tagen.           |
| Angebot  | produkt_kurstyp | Zwischentabelle: Für welche Kurstypen gilt welches Produkt?                                                                |
| Kurse    | kurs_serie      | Vorlage für einen wöchentlichen Kurs, z. B. „jeden Dienstag 18:00 Reformer“, mit Zeitraum                                  |
| Kurse    | kurstermin      | Ein konkreter Termin an einem Datum. Das ist das, was man bucht. Er kann aus einer Serie stammen oder ein Einzelkurs sein. |
| Geld     | kauf            | Ein Kauf über Stripe: wer, welches Produkt, zu welchem Preis, Zahlungsstatus                                               |
| Geld     | guthaben        | Die Einheiten aus einem bezahlten Kauf: wie viele noch übrig sind und bis wann sie gelten                                  |
| Buchen   | buchung         | Wer welchen Kurstermin gebucht hat und von welchem Guthaben die Einheit kam                                                |

**Merksatz für den Kern:** **Kauf → Guthaben → Buchung → Kurstermin.** Wer kauft, bekommt Guthaben. Mit dem Guthaben bucht man einen Kurstermin.

## Ein Beispiel durchgespielt

So sieht man am besten, wie die Tabellen zusammenarbeiten. Kundin **Kim** kauft einen **Block 10** und bucht damit den Reformer-Kurs am Dienstag.

1.  **Trainerin Tina legt eine Serie an:** „Reformer, jeden Dienstag 18:00, 5.1. bis 30.3.2027“.

    - 1 neue Zeile in kurs_serie

    - 13 neue Zeilen in kurstermin, eine pro Dienstag, alle mit kurs_serie_id = diese Serie

2.  **Kim registriert sich.**

    - 1 neue Zeile in benutzer mit rolle = KUNDE

3.  **Kim kauft Block 10** und wird zu Stripe weitergeleitet.

    - 1 neue Zeile in kauf mit status = OFFEN. Preis, Einheiten und Gültigkeit werden aus dem Produkt **kopiert**.

4.  **Stripe bestätigt die Zahlung.**

    - kauf.status wird BEZAHLT

    - 1 neue Zeile in guthaben: einheiten_gesamt = 10, einheiten_rest = 10, gueltig_bis = Zahlungstag + Gültigkeit

5.  **Kim bucht den Termin am 5.1.**

    - 1 neue Zeile in buchung mit Verweis auf Kim, den Termin und ihr Guthaben

    - guthaben.einheiten_rest wird 9

**Was die Datenbank dabei verhindert:**

- Klickt Kim zweimal auf „Buchen“, entsteht **keine** zweite Buchung. Pro Person und Termin gibt es nur eine aktive Buchung.

- Meldet Stripe dieselbe Zahlung zweimal, gibt es **kein** zweites Guthaben, weil die Stripe-ID eindeutig ist.

- einheiten_rest kann **nie** unter 0 fallen.

## Unsere Entscheidungen erklärt

**Kurse: Serie + Termin statt nur „Kurs“** Trainer:innen bieten Kurse wöchentlich an. Würden wir nur die Regel „jeden Dienstag“ speichern, hätte eine Buchung kein konkretes Ziel. Deshalb erzeugt das Backend aus jeder Serie echte Termine. Ein einzelner Termin kann dann abweichen, z. B. eine Vertretung. Im MVP ist eine Serie danach nicht mehr bearbeitbar, das spart viel Aufwand.

**Guthaben pro Kauf statt einer Zahl beim Benutzer** Jeder Block hat ein eigenes Ablaufdatum und gilt nur für bestimmte Kurstypen. Mit einer einzigen Zahl „Kim hat 12 Einheiten“ wüssten wir nicht mehr, welche wann verfällt.

**Preis wird beim Kauf kopiert** Ändert das Studio später den Preis, sollen alte Käufe trotzdem den damals bezahlten Preis zeigen. Darum stehen Preis, Einheiten und Gültigkeit zusätzlich im kauf.

**Eine Tabelle für alle Rollen** Kund:innen, Trainer:innen und Admin haben dieselben Felder und melden sich gleich an. Drei Tabellen wären mehr Code ohne Nutzen.

**Konto löschen = anonymisieren** Rechnungsdaten muss das Studio aufbewahren. Darum löschen wir die Zeile nicht, sondern entfernen Name, Telefon und Passwort und ersetzen die E-Mail. Das Admin-Konto kann nie gelöscht werden.

**Ein Platz pro Person und Termin** „Mehrere Plätze buchen“ kommt erst in einem späteren Release. Bis dahin verhindert die Datenbank Doppelbuchungen.

**Geld in Cent, Zeiten in UTC** Kommazahlen rechnen ungenau (0,1 + 0,2 ergibt nicht exakt 0,3). Darum speichern wir 25,00 € als 2500. Zeiten in UTC vermeiden Fehler bei der Sommerzeit-Umstellung.

**Keine Kartendaten** Kartendaten gibt man nur bei Stripe ein. Wir speichern nur die Stripe-IDs. Das ist sicherer und erspart uns strenge Vorschriften (PCI).

## Was das für Frontend und Backend bedeutet

**Frontend:** Ihr greift **nie direkt** auf die Datenbank zu, nur über die API. Das Modell hilft euch aber zu verstehen, woher die Daten kommen:

- Feldnamen sind in der Datenbank snake_case (max_teilnehmer), in der API camelCase (maxTeilnehmer). Gemeint ist dasselbe.

- Freie Plätze sind keine Spalte. Das Backend zählt sie aus den aktiven Buchungen und schickt sie als freiePlaetze mit.

- „Mein Guthaben“ zeigt mehrere Blöcke, nicht eine Summe, weil jeder Block ein eigenes Ablaufdatum hat.

**Backend:** Aus jeder Tabelle wird eine **C#-Entity-Klasse** für Entity Framework Core, z. B. Kurstermin. EF Core legt die Tabellen später selbst an (Migrationen). Das SQL-Skript ist bis dahin unsere Diskussionsgrundlage. Regeln, die die Datenbank nicht prüfen kann (z. B. „Kurs nicht überbuchen“), prüft das Backend in einer Transaktion.

## Selbst anschauen

Drei Wege, vom einfachsten zum ausführlichsten:

1.  **Nur ansehen (0 Min.):** das Bild oben in diesem Doc bzw. die Datei TrainTone_Datenmodell.png.

2.  **MySQL Workbench (5 Min.):** *File → Import → Reverse Engineer MySQL Create Script…* → TrainTone_Datenmodell_MySQL.sql wählen → Häkchen „Place imported objects on a diagram“ → *Execute → Next → Finish*. Dann *Arrange → Autolayout*. Dafür braucht ihr keine laufende Datenbank.

3.  **Echte Datenbank mit Docker und DataGrip (15 Min.):** Ordner mit docker-compose.yml, .env und db/init/01_schema.sql anlegen, docker compose up -d ausführen, in DataGrip als PostgreSQL verbinden (localhost:5432). Dann Rechtsklick auf public → *Diagrams → Show Diagram*. Die Details stehen im Doc „Train and Tone – Datenmodell“.

**Fragen oder Einwände?** Schreibt einen Kommentar direkt an die Stelle in diesem Doc. Ari sammelt sie fürs nächste Team-Meeting.
