---
name: higgsfield-medien
description: Erzeugt und bearbeitet Medien für Web- und App-Projekte über die Higgsfield-MCP-Anbindung. Verwende diesen Skill, wenn es um Bild erzeugen, Grafik, Hero-Bild, Header-Bild, Produktbild, Illustration, Icon, Logo, Brandkit, Corporate Design, Mockup, Favicon-Motiv, Placeholder-Bilder, Video, Clip, Animation rendern, Produktvideo, UGC-Ad, Explainer, YouTube-Thumbnail, Sprachausgabe, Voiceover, Text-to-Speech, Stimme klonen, 3D-Modell, GLB, Sprite, Bild hochskalieren, Upscale, 4K, Hintergrund entfernen, freistellen, Bild beschneiden/erweitern, Seitenverhältnis ändern, Reframe oder um Higgsfield selbst geht. Auch bei "mach mir ein Bild für die Landingpage", "wir brauchen noch Assets", "das Bild ist zu klein/zu unscharf" oder "Logo ohne Hintergrund".
---

# Higgsfield-Medien für Web- und App-Projekte

Ziel: brauchbare Assets im Repo, ohne Credits zu verbrennen. Reihenfolge immer:
**Guthaben prüfen → Modell wählen → Prompt schreiben → erzeugen → herunterladen → in `assets/` + `assets/MEDIEN.md` dokumentieren.**

## 1. Vor dem ersten teuren Lauf: Guthaben

- `balance` liefert genau zwei Felder, z. B. `{"credits": 358.5, "subscription_plan_type": "plus"}`.
  Das ist die einzige Abfrage, die du routinemäßig machst.
- `get_cost: true` in den `params` von `generate_image` / `generate_video` / `generate_3d` /
  `upscale_image` / `outpaint_image` / `reframe` gibt die Kosten zurück, **ohne** einen Job zu starten.
  Bei Video und bei größeren Serien immer erst so preflighten.
  Nicht verfügbar: innerhalb von `*_batch` und bei `upscale_video`.
- `show_plans_and_credits` **nicht** aus Neugier aufrufen — es öffnet ein Kauf-Widget mit Verkaufstext
  und Checkout-Links. Nur wenn Jakob ausdrücklich Credits kaufen oder das Abo ändern will.
- `use_unlim` weglassen. Lässt du es weg und es gibt ein Freikontingent, antwortet der Server mit
  `unlim_choice` statt zu generieren — dann nachfragen. `use_unlim: true` deckelt `count` auf 1.
- Plan Plus erlaubt parallel ca. 6 Videos / 8 Bilder — mehr gleichzeitig bringt nichts.

## 2. Entscheidungsbaum

| Ziel | Werkzeug |
|---|---|
| Einzelbild, Hero, Illustration, Produktshot | `generate_image` |
| 2–12 unabhängige Bilder (verschiedene Prompts) | `generate_image_batch` |
| 2–4 Varianten *desselben* Prompts | `generate_image` mit `count: 2..4` |
| Kurzes Video / Animation | `generate_video`, mehrere: `generate_video_batch` |
| Sprachausgabe, Voiceover | `generate_audio` (+ `list_voices`), mehrere Zeilen: `generate_audio_batch` |
| Bild → 3D-Mesh (GLB) | `generate_3d` |
| Bild schärfer / größer (2K/4K) | `upscale_image` |
| Video schärfer / größer | `upscale_video` |
| Bildrand erweitern, Uncrop, anderes Bildformat | `outpaint_image` |
| **Video** auf anderes Seitenverhältnis bringen | `reframe` (nur Video!) |
| Freisteller / transparenter Hintergrund | `remove_background` (Bild *oder* Video) |

### Wann zuerst `get_workflow_instructions`

Bei mehrstufigen, gebrieften Produktionen **musst** du erst den Workflow laden
(`get_workflow_instructions` ohne Argument = Katalog, dann mit `workflow: "<name>"`),
statt direkt `generate_*` zu rufen. Echte Namen im Katalog:

| Workflow | Wofür |
|---|---|
| `brandkit` | Logo, Visual Identity, Brandbook, Mockups, Merch, Verpackung, Social-Grafiken, Banner |
| `character-sheet` | Character-/Model-Sheet, Turnaround, konsistente Figur über mehrere Ansichten |
| `faceless-channel-video` | Erzähltes Video ohne Gesicht: Explainer, History, Kids, Märchen (30 s bis 10 min) |
| `ugc-flow` | UGC-Standard: Creator spricht in die Kamera, Produkt-Review |
| `ugc-product-flow` | Nur Produkt, Voiceover, niemand vor der Kamera |
| `ugc-saas-flow` | Video für eine Website/Web-App/Shop-URL, echte Screenshots als Overlay |
| `ugc-try-on-flow` | Anprobe / Fit-Check / OOTD |
| `ugc-tutorial-flow` | Schritt-für-Schritt-Anleitung mit "Step N"-Einblendungen |
| `ugc-unboxing-flow` | Unboxing / First Reaction / Haul |
| `youtube-thumbnail-generator` | YouTube-/Instagram-Thumbnail, Video-Cover (auch statt direktem `generate_image`) |

Der Katalog kann wachsen — vor dem Bauen immer erst ohne Argument listen, nie raten.
Braucht ein geladener Workflow eine Vorlagendatei, holst du sie mit `get_workflow_bundle_file`.

## 3. Modelle (Stand der Recherche, mit `models_explore` prüfbar)

`models_explore(action:"list")` ist sehr lang; gezielter ist
`models_explore(action:"recommend", query:"...", type:"image"|"video"|"audio"|"3d")`
oder `action:"get"` mit `model_id` für die exakten Parameter.

**Bild**

| Modell | Stärke |
|---|---|
| `nano_banana_pro` | höchste Qualität, gut bei Text und Diagrammen, 4K |
| `nano_banana_2` / `nano_banana` | schnell bzw. günstig, realistisch |
| `seedream_v4_5`, `seedream_v5_pro`, `seedream_v5_lite` | 4K, präzise Steuerung, instruktionsbasiertes Editieren |
| `flux_2` | sehr genaue Prompt-Befolgung (`variant`: pro/flex/max) |
| `gpt_image_2`, `openai_hazel` | Editing, beste Textwiedergabe im Bild |
| `recraft_v4_1` | Logos/Icons/Vektor (`model_type: vector`), Produkt-Mockups (`utility`), dazu `colors` (#RRGGBB) und `background_color` |
| `soul_2` / `soul_v2` | Porträts, Fashion, UGC-Look |
| `cinematic_studio_2_5`, `soul_cinematic` | kinoartige Stills, Concept Art |
| `marketing_studio_image` | Ein-Klick-Produktwerbung |

**Video**: `seedance_2_0` (Standard; `duration` 4–15 s, `resolution` 480p/720p/1080p/4k,
`mode: std|fast`, `genre`, `generate_audio` standardmäßig **true**), `seedance_2_0_mini` (billiger),
`flux_3_video` (5–20 s, 720p/1080p, synchroner Ton), `grok_video_v15` (2–15 s),
`higgsfield_preset` (braucht `preset_id` aus `presets_show` **und** ein Startbild),
`marketing_studio_video`, `cinematic_studio_3_0`, `topaz_video` (Upscale).

**Audio**: `seed_audio` (Standard, ByteDance; `voice_type: preset|element` + `voice_id`,
dazu `speech_rate`, `pitch_rate`, `loudness_rate`) oder `text2speech_v2` mit
`variant: elevenlabs|minimax|seed_speech|vibe_voice|cozy_voice`. Stimm-IDs kommen aus `list_voices`.
Musik und Soundeffekte gibt es hier **nicht** — `sonilo_music`, `mirelo_text_to_audio` und
`inworld_text_to_speech` sind ausschließlich für die Game-Pipeline und dürfen nicht ersatzweise
benutzt werden. Für Sprachausgabe in Jakobs Podcasts bleibt edge-tts zuständig.

**3D**: `image_to_3d` (Standard), `multi_image_to_3d` (1–4 Ansichten desselben Objekts, bessere
Geometrie), `sam_3_3d` (einzelnes Objekt freilegen), `3d_rigging` (erwartet `model_url`, keine Bilder).

## 4. Batch-Protokoll (Pflicht bei mehreren Assets)

Nicht zehnmal `generate_image` rufen. Stattdessen:

1. `generate_image_batch` mit 1–12 `requests`, jedes mit stabilem `index` (z. B. Szenen-/Slot-Nummer).
2. `jobs_wait` mit den zurückgegebenen `job_id`s, ebenfalls max. 12 pro Gruppe,
   `timeout_seconds` max. 15. Ist `all_terminal: false`, `poll_after_seconds` abwarten und erneut rufen.
3. Wenn **alle** Jobs terminal sind: **genau ein** `show_generation_by_ids` mit dem kompletten
   indizierten Satz (max. 60). Niemals `show_generations` oder `job_display` pro Job.

Analog `generate_video_batch` und `generate_audio_batch`. Die Ergebnis-URLs liefert schon `jobs_wait`.

## 5. Prompts für dieses System

Bau den Prompt aus festen Bausteinen, in dieser Reihenfolge:

1. **Motiv** — was genau zu sehen ist, konkret und zählbar ("ein Zapfhahn aus gebürstetem Edelstahl, frontal").
2. **Bildsprache/Stil-Anker** — "Produktfotografie", "flache Vektorillustration", "Cinema-Still",
   plus ein Anker wie "wie ein Katalogfoto von 2020", nicht "schön" oder "modern".
3. **Licht** — "weiches Softbox-Licht von links oben, weiche Schatten", "hartes Gegenlicht".
4. **Kamera** — Perspektive und Brennweite: "Augenhöhe, 50 mm, leichte Untersicht", "Top-Down".
5. **Hintergrund/Farbe** — konkret und zur Marke passend; bei `recraft_v4_1` besser über
   `colors` und `background_color` als über Worte.
6. **Negative Vorgaben** — was weg soll: "keine Menschen, keine Reflexionen, kein Logo,
   keine Beschriftung, kein Rahmen".

Regeln:
- **Kein Text ins Bild erwarten.** Beschriftungen, Preise, UI-Labels gehören in HTML/CSS über das Bild.
  Muss Text wirklich gerendert werden: `nano_banana_pro` oder `openai_hazel`, und Ergebnis prüfen.
- Seitenverhältnis explizit setzen. Bei Video ist der Default oft `16:9` — für Reels/Shorts
  `aspect_ratio: "9:16"` mitgeben, sonst kommt Querformat zurück.
- Für Videos zusätzlich die **Bewegung** beschreiben (Kamerafahrt, was sich im Bild bewegt),
  sonst wird es ein Standbild mit Rauschen.
- Konsistenz über mehrere Bilder: Referenzbild mitgeben (`medias` mit passender `role`) oder
  den `character-sheet`-Workflow nutzen — nicht durch Prompt-Wiederholung erzwingen wollen.

## 6. Ergebnisse ins Repo

1. Ergebnis-URL aus `jobs_wait` bzw. dem Generation-Widget nehmen und lokal herunterladen
   (`curl -L -o ...`). Zwischenstände und große Dateien nach `/data/builds`, **nicht** in `~/Nextcloud`.
2. Ablage im Projekt: `assets/` bzw. der projekteigene Medienordner (im ProstOmat z. B. neben den
   bestehenden Bildern). Dateiname sprechend und kleingeschrieben:
   `hero-zapfanlage-16x9.webp`, `icon-becher-vector.svg`, `clip-ui-tour-9x16.mp4`.
3. Für Web: Bilder nach dem Download in WebP/AVIF konvertieren und auf die tatsächlich benötigte
   Breite bringen. Ein 4K-PNG gehört nicht in ein Frontend.
4. `assets/MEDIEN.md` fortschreiben — eine Zeile pro Asset:

   | Datei | Tool + Modell | Prompt (gekürzt) | Seitenverhältnis / Dauer | Datum | Credits |
   |---|---|---|---|---|---|
   | `hero-zapfanlage-16x9.webp` | generate_image / nano_banana_pro | "Zapfhahn aus gebürstetem Edelstahl, …" | 16:9 | 2026-08-05 | 4 |

   Ohne diese Notiz ist ein Asset später nicht reproduzierbar.
5. Lizenz/Herkunft: erzeugte Medien sind Projektassets, aber Referenzbilder von Dritten nie
   ungeprüft hochladen.

## 7. Stolperfallen

- `medias[].value` erwartet eine **media_id oder job_id**, niemals eine `https://`-URL.
  Web-Bild zuerst durch `media_import_url` schicken (max. 50 MB), dann die zurückgegebene ID nutzen.
- Einige Modelle brauchen zwingend ein Bild: `higgsfield_preset`, `autosprite`, `sam_3_3d`,
  `image_to_3d`, `multi_image_to_3d`. Ohne Referenz schlägt der Aufruf fehl.
- `ms_image` verlangt eine `style_id`; vorher `show_marketing_studio(type:"image_style")` aufrufen
  und die Auswahl von Jakob bestätigen lassen — es gibt keinen Default.
- `upscale_image` und `upscale_video` (Provider `bytedance`) verlangen `width` und `height` der
  Quelle in Pixeln; der Server ermittelt sie nicht selbst. `upscale_video` mit `topaz` braucht sie nicht.
  Bei Video verdoppelt `fps > 30` die Kosten.
- `reframe` funktioniert **nur mit Video**. Für Bilder ist `outpaint_image` zuständig.
  Quellvideos über 15 s brauchen `duration_seconds` + `resolution`, Maximum 60 s.
- Seitenverhältnisse sind modellabhängig: `soul_cast` kann nur `16:9`, `wan2_6` nur `16:9/9:16/1:1`,
  `flux_2` nur `1:1/4:3/3:4/16:9/9:16`. Vor dem Lauf mit `models_explore(action:"get")` prüfen.
- `seedance_2_0`: `4k`/`1080p` gehen nur mit `mode: "std"`; `mode: "fast"` kann nur 480p/720p.
  `generate_audio` steht auf `true` — für stumme UI-Clips explizit auf `false` setzen.
- Jobs sind asynchron. Ohne `jobs_wait` (bzw. `job_status`) hast du keine Datei, nur eine Job-ID.
- Gibt der Server `adjustments` zurück, wurden Parameter stillschweigend korrigiert — vorlesen,
  bevor du das Ergebnis als "wie bestellt" verkaufst. Ein zurückgegebenes `recovery_tool`
  sofort aufrufen, nicht erst diskutieren.
- 3D-Meshes zeigen nur, was im Quellbild steckt. Fehlende Requisiten erst im Bild ergänzen
  (`generate_image`), dann konvertieren.
