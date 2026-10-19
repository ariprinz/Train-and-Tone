# Claude-Skills für Train and Tone – Anleitung

Für unser Projekt gibt es zwei **Claude-Skills**. Ein Skill ist eine Anleitung, die Claude automatisch befolgt, wenn es um unser Projekt geht.

| Skill | Wofür | Wo verwenden |
|---|---|---|
| `train-and-tone-planung` | Projektplanung lernen: Lastenheft, Pflichtenheft, User Stories, MoSCoW, Story Map, Domain Storytelling, Kanban, Zeitplan | **claude.ai** (Chat) |
| `train-and-tone-dev` | Code schreiben: sicher, sauber, mit Tests und Git-Workflow – Frontend und Backend | **Claude Code** |

**Beide erklären Schritt für Schritt**, was passiert und warum. Ihr bekommt also nicht nur Ergebnisse, sondern lernt die Methoden und Konzepte dahinter. Das hilft auch bei der Diplomarbeit und der Präsentation.

Ein paar Dinge, die die Skills machen:
- 📍 zeigen, wo ihr gerade steht, und 👉 enden mit einem konkreten nächsten Schritt
- 🔒 prüfen und erklären Sicherheit (XSS, SQL-Injection, Passwörter, Bezahlung, DSGVO …)
- ✍️ bieten an, dass ihr Code oder Dokumente **zuerst selbst** macht und Hinweise bzw. ehrliches Feedback bekommt

**Die Nutzung ist freiwillig.** Wenn euch ein Skill nicht passt, könnt ihr ihn ausschalten oder anpassen (siehe unten).

---

## 1. `train-and-tone-planung` in claude.ai (jetzt starten)

Planung macht man meistens im Chat. Dort werden Skills **nicht** aus dem Repo geladen, deshalb ladet ihr diesen Skill einmal selbst hoch.

### Hochladen (einmalig)
1. Die Datei `.claude/skills/train-and-tone-planung/SKILL.md` aus dem Repo herunterladen.
2. Daraus eine **ZIP-Datei** machen. Wichtig: Die `SKILL.md` muss **direkt** in der ZIP liegen, nicht in einem Unterordner. Wenn ihr die ZIP öffnet, solltet ihr sofort `SKILL.md` sehen.
3. In claude.ai in die **Einstellungen** gehen, den Bereich für **Skills** öffnen und die ZIP hochladen.

### Verwenden
Einfach im Chat nach der Planung fragen, z. B. „Erklär mir, wie man ein Lastenheft für Train and Tone schreibt“. Gezielt starten:
```
/train-and-tone-planung Erklär mir, wie man User Stories schreibt.
```

> Falls der Planungs-Skill in Claude Code doppelt auftaucht (einmal aus dem Repo, einmal von eurem Konto): dort per `/skills` eine Version ausschalten.

---

## 2. `train-and-tone-dev` in Claude Code (später, wenn wir programmieren)

Dieser Skill liegt im Repo unter `.claude/skills/train-and-tone-dev/SKILL.md`. **Ihr müsst nichts installieren:** Sobald Claude Code im Projektordner arbeitet, ist der Skill automatisch da. Neue Versionen bekommt ihr mit `git pull`.

> **Bitte nicht zusätzlich in claude.ai hochladen.** Sonst kann er in Claude Code doppelt auftauchen.

### Schritt 1: Repo auf den Computer holen (einmalig)
```
git clone <link-zu-unserem-repo>
```
Merkt euch, in welchem Ordner das Repo jetzt liegt.

### Schritt 2: Projektordner in der Claude Desktop-App öffnen
1. Die **Claude Desktop-App** öffnen und zum Bereich **Code** (Claude Code) wechseln.
2. **Unter dem Eingabefeld** gibt es eine Ordnerauswahl. Draufklicken.
3. Den Ordner auswählen, in den ihr das Repo geklont habt (den Hauptordner des Projekts, nicht einen Unterordner).
4. Fertig – Claude Code arbeitet jetzt in unserem Projekt und lädt den Skill automatisch.

<details>
<summary>Alternativ: VS Code oder Terminal</summary>

- **VS Code:** Projektordner öffnen (Datei → Ordner öffnen), Claude Code-Erweiterung installieren und das Claude Code-Fenster öffnen.
- **Terminal:**
  ```
  cd Pfad/zum/Projektordner
  claude
  ```
</details>

### Schritt 3: Prüfen
`/skills` eingeben – `train-and-tone-dev` sollte in der Liste stehen.

### Verwenden
Claude nutzt den Skill automatisch, wenn ihr am Projekt-Code arbeitet. Gezielt starten:
```
/train-and-tone-dev Ich will die Kursplan-Ansicht bauen.
```

Vor dem Arbeiten immer `git pull`, damit ihr den neuesten Stand (auch vom Skill) habt.

---

## Ausschalten oder anpassen

**Nur für dich ausschalten**
- Claude Code: `/skills` eingeben und den Skill ausschalten. Betrifft nur dich.
- claude.ai: in den Einstellungen bei den Skills deaktivieren oder löschen.

**Nur für dich anpassen** (z. B. weniger oder mehr Erklärungen)
- Claude Code: den Skill mit **gleichem Namen** in deinen persönlichen Ordner kopieren und dort ändern:
  ```
  ~/.claude/skills/train-and-tone-dev/SKILL.md
  ```
  Deine Version ersetzt dann die Repo-Version nur für dich. Zurück zur Team-Version: den Ordner wieder löschen.
- claude.ai: `SKILL.md` bearbeiten, neu zippen, erneut hochladen.

> Bitte keine zusätzlichen Skills mit **anderem Namen** anlegen, die dasselbe tun – dann widersprechen sich die Anweisungen. Und große Änderungen lieber für alle machen (siehe unten), damit Claude bei uns allen nach denselben Regeln arbeitet, z. B. bei Sicherheit und Git.

**Für alle verbessern**
Die Datei im Repo auf einem eigenen Branch ändern und einen **Pull Request** erstellen. Nach dem Merge haben alle die neue Version.

Fragen oder Ideen: einfach in der Gruppe melden.
