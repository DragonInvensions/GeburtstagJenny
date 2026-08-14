# Jennys Geburtstag – Das Partyspiel 🎉

Ein Kennenlern- und Missionsspiel für Jennys Geburtstagsfeier: Gäste ziehen zufällige
Missionen (anstoßen, kennenlernen, gemeinsam kreativ werden) und bringen so die Party
in Schwung.

**Live:** https://dragoninvensions.github.io/GeburtstagJenny/

## Anpassen

Alle Missionen stehen in [`questions.json`](questions.json) – einfach Einträge ändern,
hinzufügen oder löschen und pushen. Der Pages-Workflow deployt automatisch (nur von `main`).

Der Teil in runden Klammern am Satzende wird beim Anzeigen abgetrennt und als
Erfüllungsbedingung dargestellt – pro Mission also genau eine Klammer, und die ans Ende.

## Lokal ansehen

Ein Doppelklick auf `index.html` reicht nicht: die Missionen werden per `fetch` geladen,
was der Browser bei lokalen Dateien blockiert. Stattdessen:

```bash
python3 -m http.server 8080 --bind 127.0.0.1
# dann http://127.0.0.1:8080/ öffnen
```

Wer hier mitarbeitet (auch als KI), findet die Orientierung in [`CLAUDE.md`](CLAUDE.md).

Unterstützt von [SmartTap](https://smarttap-offical.de/).
