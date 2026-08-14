---
name: animationen
description: Hochwertige Web-Animationen und Mikro-Interaktionen ohne Framework (Vanilla JS/CSS, PWA). Verwende diesen Skill bei Animation, Übergang, Transition, Bewegung, Micro-Interaction, Hover-Effekt, Ladeanimation, Skeleton, Seitenwechsel/Seitenübergang, Scroll-Effekt, Konfetti- oder Feier-Effekt — und immer dann, wenn es heißt "fühlt sich träge an", "soll lebendiger wirken", "wirkt billig", "da fehlt Feedback" oder eine Oberfläche teurer und durchdachter wirken soll.
---

# Bewegung, die teuer wirkt

Teure Oberflächen bewegen sich **wenig, schnell und in Richtung der Aufmerksamkeit**. Verspielte
bewegen alles, lange, mit Bounce. Alle Rezepte hier laufen ohne Build-Schritt und ohne Bibliothek.

## 1. Die vier Grundregeln

**Dauer.** Je größer die bewegte Fläche, desto länger — aber nie lang:

| Fall | Dauer | Beispiel |
|---|---|---|
| Mikro | 120–200 ms | Hover, Fokus, Checkbox, Button-Druck |
| Standard | 200–300 ms | Dropdown, Tooltip, Karte einblenden, Tab-Wechsel |
| Groß | 300–500 ms | Modal, Bottom-Sheet, Seitenwechsel, Vollbild |

Über 500 ms wirkt es träge, unter 100 ms nimmt man die Bewegung nicht mehr wahr (dann lieber keine).
Auf dem Handy an das untere Ende gehen.

**Easing.** `linear` sieht fast immer falsch aus: In der physischen Welt startet nichts ohne
Beschleunigung und stoppt nichts ohne Abbremsen — das Auge liest es als "billig" oder "kaputt".
Ausnahmen: Endlosrotation (Spinner), Farbverläufe, scroll-getriebene Animationen.

```css
:root {
  --e-out:   cubic-bezier(0.16, 1, 0.3, 1);      /* Erscheinen: schnell rein, weich aus */
  --e-in:    cubic-bezier(0.7, 0, 0.84, 0);      /* Verschwinden: weich an, schnell weg */
  --e-inout: cubic-bezier(0.65, 0, 0.35, 1);     /* Bewegung von A nach B, beides sichtbar */
  --e-snap:  cubic-bezier(0.34, 1.2, 0.64, 1);   /* minimaler Überschwinger, sparsam! */
}
```

Beim **Erscheinen** soll das Element sofort da sein und sich dann setzen → `ease-out`. Beim
**Verschwinden** interessiert der Anfang niemanden mehr → `ease-in`. Ein Modal, das mit `ease-in-out`
schließt, fühlt sich zäh an; mit `ease-in` entschieden. Die CSS-Keywords (`ease`, `ease-out`) sind zu
flach — die `cubic-bezier`-Werte oben sind der eigentliche Unterschied.

**Versatz (Stagger).** Listen nie gleichzeitig einblenden: 30–60 ms pro Element, ab ~8 Elementen
deckeln, sonst wartet das letzte eine Sekunde. **Verlauf statt Sprung:** jede vom Nutzer ausgelöste
Zustandsänderung bekommt einen Übergang — und jede nicht ausgelöste (eintreffende Daten) erst recht.

## 2. Was animiert wird — und was nie

Nur **`transform`** und **`opacity`** — beide laufen im Compositor, ohne Layout und ohne Neuzeichnen,
und halten 60 fps auch auf schwachen Android-Geräten. Nie in Schleifen animieren: `width`, `height`,
`top`, `left`, `margin`, `padding` — jeder Frame löst Layout aus. Statt `left: 0 → 40px` also
`transform: translateX(40px)`; statt `height: 0 → auto` entweder `transform: scaleY()` auf einem
Wrapper oder `grid-template-rows: 0fr → 1fr` (kostet Layout, für Akkordeons aber vertretbar).
`box-shadow` ist teuer — stattdessen ein Pseudo-Element mit fertigem Schatten per `opacity`
einblenden; `filter: blur()` nur auf kleinen Flächen.

**`will-change` sparsam:** nur auf Elemente, die gleich wirklich animieren, danach wieder entfernen —
dauerhaft gesetzt reserviert es GPU-Speicher und macht die Seite langsamer, nicht schneller.
**Layout-Thrashing vermeiden:** nicht abwechselnd lesen und schreiben, sondern erst alle Messwerte
(`getBoundingClientRect`, `offsetWidth`) einsammeln, dann alle Änderungen schreiben.

## 3. Rezepte

### Einblenden mit Versatz
```css
.reveal { opacity: 0; transform: translateY(12px);
  transition: opacity .3s var(--e-out), transform .3s var(--e-out); }
.reveal.is-in { opacity: 1; transform: none; }
```
```js
document.querySelectorAll('.reveal').forEach((el, i) => {
  el.style.transitionDelay = Math.min(i, 8) * 45 + 'ms';
  requestAnimationFrame(() => el.classList.add('is-in'));
});
```

### Button-Druckgefühl
```css
button { transition: transform .12s var(--e-out), background-color .15s var(--e-out); }
button:active { transform: scale(.97); }
@media (hover: hover) { button:hover { transform: translateY(-1px); } }
```
`(hover: hover)` verhindert, dass Touch-Geräte den Hover-Zustand kleben lassen.

### Karten-Hover mit Tiefe
```css
.karte { position: relative; transition: transform .25s var(--e-out); }
.karte::after { content:''; position:absolute; inset:0; border-radius:inherit;
  box-shadow: 0 12px 32px rgba(0,0,0,.18); opacity:0;
  transition: opacity .25s var(--e-out); pointer-events:none; }
@media (hover: hover) {
  .karte:hover { transform: translateY(-4px); }
  .karte:hover::after { opacity: 1; }
}
```

### Skeleton-Ladezustand
```css
.skeleton { background: #e9e9ee; border-radius: 6px; position: relative; overflow: hidden; }
.skeleton::after { content:''; position:absolute; inset:0; transform: translateX(-100%);
  background: linear-gradient(90deg, transparent, rgba(255,255,255,.6), transparent);
  animation: sk 1.4s linear infinite; }
@keyframes sk { to { transform: translateX(100%); } }
```
Skeleton nur ab ~400 ms Ladezeit einblenden — sonst blitzt es und wirkt hektisch.

### Zahlen-Hochzählen
```js
function zaehle(el, ziel, dauer = 900) {
  const start = performance.now(), von = Number(el.textContent) || 0;
  const tick = t => {
    const p = Math.min((t - start) / dauer, 1);
    const e = 1 - Math.pow(1 - p, 3);           // ease-out cubic
    el.textContent = Math.round(von + (ziel - von) * e);
    if (p < 1) requestAnimationFrame(tick);
  };
  requestAnimationFrame(tick);
}
```
Ziffernsprünge vermeiden: `font-variant-numeric: tabular-nums` auf das Element setzen.

### Seitenwechsel per View Transition API
```js
function wechsle(render) {
  if (!document.startViewTransition) return render();   // Fallback: einfach umbauen
  document.startViewTransition(render);
}
```
```css
@keyframes vt-raus { to   { opacity: 0; transform: translateY(-8px); } }
@keyframes vt-rein { from { opacity: 0; transform: translateY(8px);  } }
::view-transition-old(root) { animation: .18s var(--e-in)  both vt-raus; }
::view-transition-new(root) { animation: .28s var(--e-out) both vt-rein; }
```
Für ein Element, das über Seiten hinweg dasselbe bleibt (z. B. das Getränkebild im ProstOmat):
`view-transition-name: getraenk;` auf Quelle und Ziel — der Browser morpht es selbst. Unterstützt in
Chromium/Safari, in Firefox teils noch nicht; der Fallback oben ist Pflicht.

### FLIP für Listenumsortierung
Umsortierte Listen dürfen nicht springen — First, Last, Invert, Play:
```js
function flip(elemente, umbauen) {
  const vorher = new Map([...elemente].map(el => [el, el.getBoundingClientRect()]));
  umbauen();                                             // DOM neu sortieren
  elemente.forEach(el => {
    const a = vorher.get(el), b = el.getBoundingClientRect();
    const dx = a.left - b.left, dy = a.top - b.top;
    if (!dx && !dy) return;
    el.animate(
      [{ transform: `translate(${dx}px, ${dy}px)` }, { transform: 'none' }],
      { duration: 280, easing: 'cubic-bezier(0.65,0,0.35,1)' }
    );
  });
}
```

### Scroll-getriebene Animation
Robuste Variante, überall lauffähig:
```js
const io = new IntersectionObserver((eintraege) => {
  eintraege.forEach(e => {
    if (!e.isIntersecting) return;
    e.target.classList.add('is-in');
    io.unobserve(e.target);            // einmal einblenden, nicht bei jedem Scroll
  });
}, { threshold: 0.15, rootMargin: '0px 0px -10% 0px' });
document.querySelectorAll('.reveal').forEach(el => io.observe(el));
```

Moderne CSS-Variante ohne JS (Chromium 115+, Safari/Firefox teilweise — nur als Aufwertung, nie
als einzige Lösung):
```css
@supports (animation-timeline: view()) {
  .reveal { animation: einblenden linear both;
            animation-timeline: view(); animation-range: entry 10% cover 35%; }
  @keyframes einblenden { from { opacity:0; transform: translateY(16px); } to { opacity:1; } }
}
```
Hier ist `linear` korrekt: Die "Zeit" ist die Scrollposition, das Easing macht der Finger.

### Dezenter Feier-Effekt (Canvas, ohne Bibliothek)
```js
function konfetti(dauer = 1200, anzahl = 90) {
  if (matchMedia('(prefers-reduced-motion: reduce)').matches) return;
  const c = Object.assign(document.createElement('canvas'), { width: innerWidth, height: innerHeight });
  c.style.cssText = 'position:fixed;inset:0;pointer-events:none;z-index:9999';
  document.body.appendChild(c);
  const ctx = c.getContext('2d'), farben = ['#e8b923', '#d94f4f', '#3f8cff', '#f2f2f2'];
  const p = Array.from({ length: anzahl }, () => ({
    x: c.width / 2, y: c.height * .45, r: 3 + Math.random() * 4, rot: Math.random() * 6,
    vx: (Math.random() - .5) * 11, vy: -6 - Math.random() * 8, vr: (Math.random() - .5) * .3,
    f: farben[(Math.random() * farben.length) | 0] }));
  const start = performance.now();
  (function frame(t) {
    const q = (t - start) / dauer;
    ctx.clearRect(0, 0, c.width, c.height);
    ctx.globalAlpha = Math.max(0, 1 - q);
    p.forEach(o => {
      o.vy += .28; o.x += o.vx; o.y += o.vy; o.rot += o.vr;
      ctx.save(); ctx.translate(o.x, o.y); ctx.rotate(o.rot);
      ctx.fillStyle = o.f; ctx.fillRect(-o.r, -o.r * .6, o.r * 2, o.r * 1.2); ctx.restore();
    });
    q < 1 ? requestAnimationFrame(frame) : c.remove();
  })(start);
}
```
Ein Ausbruch, kein Dauerregen — nach spätestens 1,5 s ist das Canvas wieder aus dem DOM.

## 4. Web Animations API

Sobald CSS umständlich wird — dynamische Werte aus JS, Sequenzen, Abbrechen, auf das Ende warten —
ist `element.animate()` das Werkzeug.
```js
const a = kachel.animate(
  [{ transform: 'scale(1)' }, { transform: 'scale(1.06)', offset: .4 }, { transform: 'scale(1)' }],
  { duration: 240, easing: 'cubic-bezier(0.16,1,0.3,1)' }
);
a.finished.then(() => kachel.classList.add('fertig'));  // Promise, sauber verkettbar
// a.cancel()  bricht jederzeit ab, ohne Klassen aufräumen zu müssen
```
Gegenüber Klassen-Toggling: kein `transitionend`-Gefummel, keine hängenden Klassen bei schnellem
Klicken, `fill: 'both'` hält den Endzustand ohne zusätzliches CSS.

## 5. Barrierefreiheit und Anstand

Global abfangen, einmal ins Stylesheet, ganz nach unten:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: .01ms !important;
    scroll-behavior: auto !important;
  }
}
```
In JS vor jeder JS-Animation zusätzlich prüfen und sonst direkt den Endzustand setzen:
`if (matchMedia('(prefers-reduced-motion: reduce)').matches) { ... }`

- Keine Endlosanimation im Sichtfeld außer Ladeindikatoren — und die verschwinden, wenn geladen ist.
- Nichts blinken lassen, was schneller als 3 Hz ist (Anfallsrisiko).
- Fokus-Ringe nie wegkürzen. `:focus-visible` darf animiert werden, muss aber sichtbar bleiben.
- Animation darf keine Eingabe blockieren: `pointer-events: none` auf Overlays, und ein Klick
  während einer Animation wirkt sofort (deshalb `a.cancel()` statt Warteschlange).
- Bewegung ersetzt keine Statusmeldung — Screenreader brauchen `aria-live`.

## 6. Performance prüfen

1. **DevTools → Performance**: aufzeichnen, Interaktion auslösen, stoppen. Im Frames-Band keine roten
   Balken, Frame-Dauer unter 16,7 ms. Unter "Rendering → Paint flashing": blinkt es während der
   Animation grün, wird gemalt statt komponiert — dann läuft sie über die falsche Eigenschaft.
2. **CPU-Drosselung auf 4x** — entspricht etwa einem Mittelklasse-Android, dem realistischen Prüffall
   für eine PWA. Im Layers-Panel bekommt die animierte Ebene eine eigene Compositing-Ebene; zwanzig
   eigene Ebenen sind dagegen ein Fehler, kein Erfolg.
3. **Optische Kontrolle** über den Skill `screenshot-verify` (Chromium headless): Start-, Zwischen-
   und Endzustand, Mobil- und Desktop-Viewport. Zwischenzustand einfrieren mit
   `document.getAnimations().forEach(a => { a.currentTime = 120; a.pause(); })`.

## 7. Gegenprobe: so sieht Amateur-Bewegung aus

- **Alles wackelt.** Drei Elemente animieren gleichzeitig in verschiedene Richtungen.
- **Bounce überall.** Überschwinger gehören auf höchstens ein Element pro Ansicht, meist auf keins.
- **Zu lang.** 600 ms für ein Dropdown. Nach dem dritten Öffnen ist es nur noch nervig.
- **Animation blockiert Eingabe.** Der Button reagiert erst, wenn das Modal fertig eingeblendet ist.
- **`linear` als Easing** oder gar kein Easing — der Zustand springt hart um.
- **Endlos-Puls** auf einem Call-to-Action, damit er "auffällt".
- **Hover-Effekte auf Touch-Geräten**, die nach dem Tippen kleben bleiben.
- **Bewegung ohne Anlass**: etwas animiert, ohne dass der Nutzer gehandelt hat oder Neues ankam.

## 8. Randnotiz Frameworks

Nutzt ein Projekt doch React, bleiben die Regeln identisch; `framer-motion` ersetzt FLIP und
Ein-/Ausblendungen über `AnimatePresence` und liegt selbst auf der Web Animations API auf. Ohne
Build-Pipeline — dem Normalfall hier — ist jede Bibliothek Ballast.
