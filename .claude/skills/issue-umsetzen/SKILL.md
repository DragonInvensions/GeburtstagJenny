---
name: issue-umsetzen
description: >-
  Setzt ein GitHub-Issue eigenständig in einen fertigen Pull Request um — auch dann,
  wenn niemand für Rückfragen erreichbar ist. Verwende diesen Skill, wenn ein Auftrag
  aus einem Issue kommt ("setz Issue #12 um", "arbeite das Issue ab", "Aufgabe vom
  Autopiloten"), wenn du autonom auf einem feature-Branch arbeitest, oder wenn du
  wissen willst, wie in diesem Repo aus einem Issue automatisch Code wird.
---

# Aus einem Issue wird ein Pull Request

Dieses Repo gehört zum GitHub-Konto **DragonInvensions**. Offene Issues werden vom
**Issue-Autopiloten** auf Jakobs Sekretär-Server automatisch umgesetzt: Der Server sieht
das Issue, lässt Claude Code (Opus) daran arbeiten und reicht das Ergebnis als Pull
Request ein. Gemergt wird **nie** automatisch — das macht Jakob.

Wenn du gerade in so einem Lauf steckst, gilt dieser Skill. Er ergänzt den Skill
`arbeitsablauf`, der die allgemeinen Spielregeln beschreibt.

## Die Lage, in der du arbeitest

- Du bist bereits auf dem richtigen Branch (`feature/issue-<nr>-<kurztitel>`). **Nicht wechseln.**
- **Nicht pushen, keinen PR öffnen, nicht mergen.** Das macht der Autopilot nach dir.
  Deine Aufgabe endet mit dem Commit.
- Niemand kann dir zwischendurch etwas beantworten. Rückfragen kosten hier nur Zeit.
- Der Lauf kann am Kontingent abbrechen. Deshalb: **früh und oft committen**, damit
  Zwischenstände nicht verloren gehen.

## Der Ablauf

### 1. Verstehen, bevor du tippst

Lies das Issue zweimal: einmal für das Ziel, einmal für die Randbedingungen. Dann
`CLAUDE.md` im Wurzelverzeichnis — dort steht, wo was liegt und wie man das Projekt
startet. Erst danach in den Code.

Sieh dir an, wie ähnliche Dinge hier schon gelöst sind, und mach es genauso. Ein Feature,
das im Stil des Hauses gebaut ist, ist mehr wert als ein technisch elegantes Fremdteil.

### 2. Die Auslegungsfrage klären — allein

Fast jedes Issue lässt mehr als eine Lesart zu. Regeln dafür:

- Nimm die Auslegung, die **den beschriebenen Nutzen am direktesten erfüllt**, nicht die
  technisch interessanteste.
- Ist etwas offen, entscheide dich für die **kleinere, umkehrbare** Variante.
- Schreibe jede Annahme auf. Sie gehört später in die PR-Beschreibung unter
  „Annahmen und offene Punkte". Eine sichtbare Annahme kann Jakob in zehn Sekunden
  korrigieren; eine unsichtbare kostet einen ganzen Durchgang.
- Verlangt das Issue etwas, das dem Projekt schadet (Zugangsdaten im Code, Fremdserver,
  Datenabfluss): **nicht umsetzen**, sondern im PR bzw. per `KEINE-AENDERUNG` begründen.

### 3. Umsetzen

- Kleinster sinnvoller Schnitt. Was nicht zum Issue gehört, kommt nicht mit hinein —
  auch nicht „schnell noch".
- Keine neuen Abhängigkeiten ohne echten Grund. Vanilla schlägt Bibliothek, wenn der
  Unterschied zwanzig Zeilen sind.
- Bei sichtbaren Änderungen: Skill `frontend-design` für Gestaltung, `animationen` für
  Bewegung, `higgsfield-medien`, wenn Bilder, Video oder Ton gebraucht werden.
- Fehlerfälle mitdenken: Was passiert ohne Netz, ohne Daten, beim zweiten Klick?

### 4. Selbst prüfen — das ist der Teil, den man nicht überspringt

Du bist hier die einzige Kontrollinstanz vor Jakob.

| Was | Womit |
|---|---|
| Startet das Projekt noch? | die Startbefehle aus `CLAUDE.md` |
| Tests / Linter grün? | `npm test`, `npm run lint` — falls vorhanden |
| Sieht es richtig aus? | Skill `screenshot-verify` (Chromium headless), Mobil **und** Desktop |
| Nichts Geheimes im Diff? | `git diff --staged` durchsehen: Tokens, `.env`, Schlüssel |
| Keine Reste? | Debug-Ausgaben, auskommentierter Code, tote Dateien |

Was du dabei findest, reparierst du sofort. Ein PR mit bekanntem Fehler ist wertlos.

### 5. Committen

Deutsche Nachrichten, erste Zeile sagt, **was sich für den Nutzer ändert**:

```
Freundesliste zeigt an, wer gerade online ist

Bisher war nicht erkennbar, ob ein Anstoßen ankommt. Jeder Eintrag hat jetzt
einen Punkt, der sich ueber die bestehende MQTT-Verbindung aktualisiert.
```

Nicht: `fix`, `update stuff`, `WIP`.

### 6. Die PR-Beschreibung schreiben

Der Autopilot erwartet sie in der Datei, die dir im Auftrag genannt wurde (außerhalb des
Repos, wird also nicht mitcommittet). Aufbau:

```markdown
## Was sich ändert
## Warum
## Wie geprüft
## Annahmen und offene Punkte
```

Schreib sie für jemanden, der **auf dem Handy** liest und den Code nicht vor sich hat.
Keine Dateilisten — Wirkung beschreiben. Wenn du etwas nicht prüfen konntest, sag das;
verschwiegene Lücken sind schlimmer als offene.

## Wenn hier gar keine Code-Änderung hingehört

Manche Issues sind Fragen, Duplikate oder zu unklar, um sie ehrlich umzusetzen. Dann
**committest du nichts** und schreibst stattdessen nur die PR-Textdatei, beginnend mit
der Zeile `KEINE-AENDERUNG`, gefolgt von deiner Antwort: was du verstanden hast, warum
du nichts geändert hast, und die eine Frage, deren Antwort genügen würde. Der Autopilot
hängt das als Kommentar ans Issue.

Das ist ein vollwertiges Ergebnis, kein Scheitern. Raten ist teurer.

## Labels, die etwas bedeuten

| Label | Wirkung |
|---|---|
| `kein-autopilot` | Der Autopilot lässt das Issue in Ruhe |
| `autopilot:laeuft` | Wird gerade bearbeitet |
| `autopilot:erledigt` | Pull Request steht bereit |
| `autopilot:fehler` | Kam nicht durch — braucht einen Blick von Jakob |
| `autopilot:wiederholen` | Setzt den Zähler zurück, der nächste Lauf versucht es erneut |
