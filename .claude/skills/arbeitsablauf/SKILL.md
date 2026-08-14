---
name: arbeitsablauf
description: Verbindlicher Arbeitsablauf fuer Jakobs Repos (GitHub-Konto DragonInvensions) - Branch, Commit, Pull Request, Selbstpruefung. Verwende diesen Skill, sobald eine Aenderung committet werden soll, ein Branch angelegt wird, ein Pull Request geoeffnet wird, eine Commit-Nachricht formuliert wird, jemand fragt "wie arbeite ich hier", Code eingecheckt wird, ein Review ansteht oder eine Aufgabe abgeschlossen wird. Auch am Anfang einer Aufgabe lesen, damit der Schnitt von vornherein passt.
---

# Arbeitsablauf in diesem Repo

Kurzfassung: **eigener Branch → kleine Aenderung → selbst pruefen → deutscher Commit → Pull Request → Jakob mergt.**
Du mergst nie selbst. Du pusht nie auf den Standardbranch.

## 1. Ablauf in Schritten

**a) Verstehen.** Erst das Repo lesen (README, CLAUDE.md, bestehende Struktur), dann anfangen.
Vorhandene Muster uebernehmen, statt eigene einzufuehren.

**b) Kleinsten sinnvollen Schnitt waehlen.** Ein Thema pro Branch, ein Branch pro PR.
Grosse Auftraege in mehrere PRs zerlegen — ein PR, den Jakob auf dem Handy in zwei Minuten
versteht, ist mehr wert als ein vollstaendiger, den niemand prueft.

**c) Branch anlegen.** Immer vom aktuellen Standardbranch, nie auf ihm arbeiten.

```bash
gh auth status                                    # muss DragonInvensions zeigen
BASIS=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)
git fetch origin
git switch -c feature/login-und-testknopf origin/$BASIS
```

Namen deutsch, kebab-case, sprechend. Praefix nach Art der Aenderung:
`feature/…` neue Funktion · `fix/…` Fehlerbehebung · `docs/…` Doku und Vorlagen.
Gelebte Beispiele: `feature/deploy-zwei-umgebungen`, `fix/sw-netz-zuerst`, `docs/caddy-unterseiten`.
Schlecht: `patch-1`, `claude/update`, `wip`.

**d) Umsetzen.** Nur das, was zum Branchthema gehoert. Keine nebenbei-Aufraeumaktionen im selben PR.

**e) Selbst pruefen.** Das Projekt starten und die Aenderung wirklich benutzen, nicht nur bauen.
Typische Befehle (im Repo pruefen, was existiert):

```bash
npm ci && npm run build        # oder: tsc -b && vite build
npm test && npm run lint
npm run dev                    # Dev-Server nur an 127.0.0.1 binden
```

Bei sichtbaren Aenderungen an einer Oberflaeche: Skill **`screenshot-verify`** verwenden und das Bild
selbst ansehen (Mobil- und Desktop-Viewport). Ein Bild sagt mehr als 1000 Log-Zeilen.

**f) Committen.** Siehe Abschnitt 2.

**g) Pushen und PR oeffnen.**

```bash
git push -u origin feature/login-und-testknopf
gh pr create --base "$BASIS" --title "Echte Konten mit Passwort, Testknopf unter /testButton" --body-file /tmp/pr.md
```

**h) Nicht mergen.** Der PR bleibt offen. Jakob mergt. Kein `gh pr merge`, kein `git merge`,
kein Push auf `main`/`master`/`release`.

## 2. Commit-Nachrichten

- Deutsch. Umlaute als `ae/oe/ue/ss` schreiben — so ist die Historie hier durchgaengig.
- **Erste Zeile: eine Aussage darueber, was sich fuer den Nutzer aendert.** Kein Dateiname, kein
  "fix stuff", kein `chore:`-Praefix. Form: `Bereich: was jetzt anders ist`. Bis ca. 100 Zeichen.
- Leerzeile, dann der Koerper: **warum** das noetig war, was vorher schiefging, was jetzt gilt.
  Aufzaehlungen mit `-`. Zwei bis fuenfzehn Zeilen sind normal.

Gut:

```
Service Worker: Netz zuerst fuer Seitenaufrufe, damit neue Versionen ankommen

Der bisherige Worker beantwortete JEDE GET-Anfrage zuerst aus dem Cache. Einmal
installiert, bekam ein Geraet dadurch nie wieder eine neue Version zu sehen — auf
dem Handy erschien weiter der alte Startbildschirm.

Jetzt:
- Seitenaufrufe/HTML: Netz zuerst, Cache nur als Notnagel ohne Verbindung
- Dateien mit Hash im Namen (/assets/) und Bilder: weiter Cache zuerst
- API, SSE und Audio bleiben unangetastet (immer live)
```

Schlecht:

```
fix sw.js
```

(Sagt nicht, was kaputt war, nicht was sich aendert, nicht warum — und ist englisch.)

## 3. Pull-Request-Beschreibung

Titel wie die erste Commit-Zeile: eine Aussage in Alltagssprache. Koerper nach dieser Vorlage:

```markdown
**Was sich aendert**
- Punkt fuer Punkt, aus Nutzersicht

**Warum**
Das Problem oder der Auftrag dahinter, mit Beleg (Fehlermeldung, Log, Symptom auf dem Handy).

**Wie geprueft**
Build/Tests/Lint gelaufen, Klickpfad im Browser, Screenshot, curl gegen den Endpunkt.

**Offene Punkte / Risiko**
Was bewusst nicht drin ist, was nach dem Merge noch getan werden muss, wo es weh tun koennte.
Wenn nichts offen ist: "keine".

Closes #<nr>
```

`Closes #<nr>` nur, wenn der PR aus einem Issue entstanden ist — dann schliesst der Merge es mit.
Tabellen sind erwuenscht, wenn sie etwas ordnen (Ports, Endpunkte, Vorher/Nachher).

## 4. Checkliste vor jedem PR

- [ ] Projekt startet und die Aenderung funktioniert im echten Betrieb, nicht nur im Build
- [ ] Tests und Lint gruen (`npm test`, `npm run lint`) — oder begruendet, warum es sie nicht gibt
- [ ] Nichts Geheimes im Diff: `git diff --stat origin/$BASIS...HEAD` sichten, keine Tokens,
      keine `.env`, keine Schluessel, keine echten Zugangsdaten
- [ ] Neue Dateien wirklich noetig? Bestehende Datei erweitern schlaegt neue Datei
- [ ] Doku nachgezogen: README, CLAUDE.md, Vorlagen unter `deploy/` — wenn sich Verhalten,
      Befehle oder Ports geaendert haben
- [ ] Bei UI-Aenderungen: Screenshot gemacht und angesehen (`screenshot-verify`)
- [ ] Branchname und PR-Titel sind ohne Kontext verstaendlich

```bash
git status --short          # nichts Ungewolltes dabei?
git diff origin/$BASIS...HEAD --stat
gh pr view --web            # so sieht Jakob es
```

## 5. Was hier nie passiert

- Push auf `main`, `master` oder `release` — auch nicht "nur schnell"
- `git merge` oder `gh pr merge` — das Mergen ist Jakobs Entscheidung
- `git push --force` auf einen geteilten Branch (auf dem eigenen Feature-Branch nur
  `--force-with-lease`, und nur solange niemand darauf aufgebaut hat)
- Zugangsdaten, Tokens, SSH-Keys oder `.env`-Inhalte in Commits, Logs oder PR-Texten
- Arbeiten als anderes GitHub-Konto als `DragonInvensions`
- Fremde Server anfassen oder deployen — Deploys laufen automatisch nach dem Merge
- `sudo` oder Systemaenderungen ausserhalb des Projektverzeichnisses
- Dev-Server oeffentlich binden; nur `127.0.0.1`

## 6. Unklarer Auftrag, niemand antwortet

Aufträge kommen oft per Sprache, und Jakob kann selten sofort nachliefern. Deshalb nicht warten:

1. Die **plausibelste Auslegung** waehlen — die, die zum bestehenden Code und zum erkennbaren
   Ziel passt — und sie umsetzen.
2. Die Annahme **sichtbar dokumentieren**, im PR unter "Offene Punkte / Risiko":
   `Angenommen, dass … . Falls stattdessen … gemeint war, ist das eine Aenderung an genau einer Stelle.`
3. Im Zweifel **kleiner schneiden statt raten**: den unstrittigen Teil als PR liefern, den
   strittigen Teil als Frage im PR-Text oder als neues Issue stehen lassen.
4. Nur wirklich blockierende Entscheidungen als Rueckfrage stellen — knapp, mit Vorschlag.

## 7. Issue-Autopilot

Laeufe werden hier automatisch aus GitHub-Issues gestartet: ein Issue wird zugewiesen, ein Agent
arbeitet es ab, ein PR entsteht. Daraus folgt:

- **Branch- und PR-Name muessen fuer sich sprechen.** Niemand liest daneben einen Chatverlauf.
- **Der PR muss allein verstaendlich sein.** Jakob liest ihn meist auf dem Handy: kurze Absaetze,
  Aufzaehlungen, das Wichtigste oben, keine Verweise auf "wie besprochen".
- Ein Issue = ein Branch = ein PR. `Closes #<nr>` nicht vergessen.
- Was im Issue nicht stand, gehoert auch nicht in den PR — Zusatzideen als neues Issue anlegen:

```bash
gh issue create --title "…" --body "…"
```
