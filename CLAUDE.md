# GeburtstagJenny

## Worum es geht

Ein Partyspiel für Jennys Geburtstagsfeier, das die Gäste ins Gespräch bringt. Wer die
Seite auf dem Handy öffnet, zieht per Knopfdruck eine zufällige Mission — anstoßen,
jemanden kennenlernen, gemeinsam etwas erfinden — und tippt danach auf „Nächste Mission".
Jede Mission trägt in Klammern eine Bedingung, wann sie als erfüllt gilt.

Technisch ist das eine einzige statische HTML-Seite ohne Framework, ohne Build und ohne
Abhängigkeiten: CSS und JavaScript stehen inline in `index.html`, die Missionstexte in
`questions.json`. Ausgeliefert wird über GitHub Pages unter
https://dragoninvensions.github.io/GeburtstagJenny/ — ein Push auf `main` genügt.

## Wo was liegt

| Ort | Inhalt |
|---|---|
| `index.html` | Die komplette Anwendung: Markup, CSS (Zeile 14–390) und Spiel-Logik (Zeile 435–591). Konfetti, Ballons, Missionswechsel. |
| `questions.json` | Alle Missionstexte als flaches Array von Strings. Die einzige Datei, die man für neue Missionen anfasst. |
| `logo.svg`, `logo.png` | Party-Logo. Das SVG zeigt die Seite an, das PNG dient als Favicon und Vorschaubild beim Teilen. |
| `smarttap_logo.png` | Logo des Sponsors im Fußbereich, verlinkt auf smarttap-offical.de. |
| `.github/workflows/static.yml` | Deploy nach GitHub Pages, ausgelöst durch Push auf `main`. |
| `.github/ISSUE_TEMPLATE/` | Issue-Vorlagen (Funktion, Fehler, Aussehen) für Jakob und den Autopiloten. |
| `.claude/skills/` | Die fünf Repo-Skills, siehe unten. Nicht verändern. |

Es gibt keine `package.json`, kein `node_modules`, keinen Bundler und keine Tests.
Das ist Absicht — die Seite soll ohne Werkzeugkette funktionieren.

## Womit man anfängt

1. **`questions.json`** — zeigt in 30 Sekunden, worum das Spiel inhaltlich geht und in
   welchem Ton die Texte geschrieben sind. Die meisten Wünsche betreffen genau diese Datei.
2. **`index.html`, ab Zeile 513 (`// ===== Spiel-Logik =====`)** — der gesamte Ablauf auf
   knapp 80 Zeilen: Laden, Mischen, Anzeigen, Überspringen. Wer den Block gelesen hat,
   kennt das Programm.
3. **`.claude/skills/issue-umsetzen/SKILL.md`** — nur nötig, wenn die Aufgabe aus einem
   Issue kommt: erklärt, wie hier ohne Rückfragen aus einem Issue ein fertiger PR wird.

## Starten, bauen, prüfen

Es gibt nichts zu bauen. Zum Ansehen braucht es einen lokalen Webserver — ein Doppelklick
auf `index.html` reicht **nicht** (siehe Fallstricke).

```bash
# Dev-Server, nur auf localhost
python3 -m http.server 8080 --bind 127.0.0.1
# danach: http://127.0.0.1:8080/

# Missionsdatei auf gültiges JSON prüfen (der einzige "Test", den es gibt)
python3 -m json.tool questions.json > /dev/null && echo "questions.json ist gültig"
```

Nach jeder sichtbaren Änderung eine Sichtprüfung machen: Skill `screenshot-verify`
(serverweit vorhanden, Chromium headless), Handy-Viewport nicht vergessen — die Seite wird
fast ausschließlich auf dem Telefon benutzt.

Deploy: passiert von allein durch den Workflow `static.yml`, sobald etwas auf `main`
liegt. Von hier aus wird nichts deployed.

## Skills in diesem Repo

| Skill | Wann |
|---|---|
| `arbeitsablauf` | Immer, sobald committet, gebrancht oder ein PR geöffnet wird. Auch am Anfang lesen, damit der Schnitt passt. |
| `issue-umsetzen` | Wenn die Aufgabe aus einem GitHub-Issue kommt und autonom bis zum PR durchlaufen soll. |
| `frontend-design` | Wenn Aussehen, Typografie oder Farbwelt geändert werden — damit es nicht nach Standardvorlage aussieht. |
| `animationen` | Bei Konfetti, Ballons, Übergängen, Hover-Effekten oder wenn etwas „träge" bzw. „billig" wirkt. |
| `higgsfield-medien` | Wenn Bilder, Logos, Videos oder Ton erzeugt oder bearbeitet werden sollen (z. B. neues Motiv, Freistellen, Hochskalieren). |

## So entsteht hier Code

Issue → Branch → PR. Nie direkt auf `main` committen, nie selbst mergen.

```
Issue auf GitHub  →  Branch feature/<name>  →  Commits (deutsch)  →  Pull Request nach main
```

Details stehen im Skill `arbeitsablauf`; wie ein Issue ohne Rückfragemöglichkeit bis zum
fertigen PR durchgearbeitet wird, im Skill `issue-umsetzen`.

Auf Jakobs Sekretär-Server läuft ein **Autopilot**: Er sieht neue Issues dieses Kontos
innerhalb weniger Minuten, lässt Claude Code daran arbeiten und reicht das Ergebnis als
Pull Request ein. Gemergt wird nie automatisch — das macht Jakob von Hand. Issues mit dem
Label **`kein-autopilot`** bleiben unangetastet. Braucht ein Issue gar keinen Code, wird
das als Kommentar am Issue vermerkt statt ein leerer PR geöffnet.

## Fallstricke

- **`file://` funktioniert nicht.** `index.html` holt die Missionen per
  `fetch('questions.json')` (Zeile 529). Öffnet man die Datei per Doppelklick, blockiert
  der Browser den Abruf und die Seite zeigt nur „Fehler beim Laden der Missionen." Immer
  über einen lokalen Server testen.
- **Klammern in Missionstexten sind Syntax.** `parseQuestion()` (Zeile 538) schneidet den
  **ersten** Ausdruck in runden Klammern heraus und zeigt ihn getrennt als Erfüllungs-
  bedingung an. Eine Klammer mitten im Satz landet also unabsichtlich als Bedingung unten,
  und eine zweite Klammer wird stillschweigend ignoriert. Pro Mission genau eine Klammer,
  und die gehört ans Ende.
- **Der Pages-Workflow lädt das gesamte Verzeichnis hoch** (`path: '.'` in `static.yml`).
  Alles, was im Repo liegt, ist nach dem Deploy öffentlich abrufbar — keine Entwürfe,
  Notizen oder privaten Bilder hier ablegen.
- **Deploy nur von `main`.** Ein Branch oder offener PR ändert an der Live-Seite nichts;
  sichtbar wird eine Änderung erst nach dem Merge.
- **Kein Spielstand.** Die Missionen werden beim Laden einmal gemischt und danach zyklisch
  durchlaufen (`currentIndex` in `updateDisplay`). Ein Neuladen der Seite fängt von vorn
  an — es gibt bewusst kein `localStorage`. Wer daran etwas ändert, ändert das Spielgefühl.
- **Alles hängt an einer Datei.** CSS, JS und Markup stehen zusammen in `index.html`. Das
  Aufteilen in mehrere Dateien oder das Einführen eines Build-Schritts ist kein
  Aufräumen, sondern eine Produktentscheidung — nur auf ausdrücklichen Wunsch.
- **Externe Schrift.** „Space Grotesk" kommt von Google Fonts. Bei schlechtem Partynetz
  fällt die Seite auf eine Systemschrift zurück; das Layout muss das aushalten.
- **UTF-8 beibehalten.** Missionen enthalten Umlaute und typografische Anführungszeichen.
  Werkzeuge, die Dateien in einer anderen Kodierung zurückschreiben, zerlegen die Texte.
- **`prefers-reduced-motion` respektieren.** Zeile 387 schaltet alle Animationen für
  Nutzer ab, die das eingestellt haben. Neue Animationen dürfen diese Regel nicht umgehen
  (z. B. per JS-getriebener Bewegung, die keine CSS-Animation ist).
