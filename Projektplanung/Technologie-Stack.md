# T&T – Technologie-Stack

## Frontend

- **React** als Framework

- **TypeScript**

- **Vite** weil Single Page Application besser passt

- **UI-Bibliothek und Kalender** mein Vorschlag: Mantine mit Mantine Schedule, wir können auch die anderen Komponenten aus dieser Libary verwenden also die Formulare usw. aber Mantine Schedule ist recht neu also es kann zu Problemen kommen ODER shadcn/ui mit FullCalendar – shadcn für die andren Elemente und FullCalender für den Kalender: bisschen aufwendiger weil man den Kalender dann extra stylen muss damit es zusammen passt.

## Backend

- **C# mit ASP.NET Core Web API auf .NET 10**: Version mit Langzeit-Support (LTS), also stabil und lange mit Sicherheitsupdates.

- **Entity Framework Core**: erzeugt die Datenbank per Code-First mit Migrations aus den C#-Klassen. Ich denke das ist was Sie am Anfang gemeint haben?

- **ASP.NET Core Identity mit Session-Cookie**: fertiges Benutzer-System für Registrierung, Login, Rollen (Kund:in, Trainer:in, Admin), Passwort-Hashing und Kontosperre nach Fehlversuchen. Das Cookie ist HttpOnly, Secure und SameSite und damit vor Diebstahl per XSS geschützt und sofort sperrbar.

- **xUnit** : Tests für die wichtigsten Geschäftsregeln, etwa keine Überbuchung und korrektes Abziehen des Guthabens.

## Datenbank und externe Dienste

- **PostgreSQL** oder SQL?

- **Stripe Checkout mit Webhooks**: gehostete Zahlungsseite, das Studio-System speichert nie Kartendaten. Das Backend legt beim Checkout selbst fest, welchem Konto das Guthaben gutgeschrieben wird, und prüft die Signatur jedes Webhooks.

- **E-Mail-Dienst** (ist erst bei Release 2 relevant): für Buchungsbestätigungen und Passwort-Reset, bevorzugt mit Hosting in der EU wegen DSGVO.
