# DLE Compliance Dashboard — Color & Logo Update

## 1. Colorblind-safe color changes

**File: `ci-styles.css`**

The status color tokens previously relied on hue alone to separate green/red/blue,
which collapses for red-green color blindness (deuteranopia/protanopia — the most
common forms). There was also a bug: `--blue` was mistakenly set to an amber value.

Updated tokens (separated by both hue AND lightness now):

```css
--pos:   #2e9c85;  /* teal-green (was olive #7ba35e) */
--neg:   #d1553d;  /* vermillion (was terracotta #d97a4a, same as accent) */
--green: #2e9c85;  --green-dim: rgba(46,156,133,.16);
--amber: #d9a441;  --amber-dim: rgba(217,164,65,.16);
--red:   #d1553d;  --red-dim:   rgba(209,85,61,.14);
--blue:  #4a90d9;  --blue-dim:  rgba(74,144,217,.16);  /* was #c19a4b (amber) — bug fix */
```

Everywhere in the app that reads `var(--green)`, `var(--red)`, `var(--amber)`,
`var(--blue)` (badges, status dots, notices, charts) picks this up automatically —
no other files needed changes.

## 2. New "Blue" theme accent (replaces olive green)

The theme picker's third swatch was mislabeled "violet" but was actually an olive
green (`#7ba35e`). It's now a real blue:

```css
[data-accent="blue"] {
  --accent: #4a90d9; --accent-2: #7bb3e8;
  --accent-dim: rgba(74,144,217,.12); --accent-fg: #7bb3e8;
  --selected: rgba(74,144,217,.12);
}
```

Swatch + JS wiring updated in `CI Portal.html` (`setAccent('blue', 'sw-blue')`,
`swMap` in `applyTheme()`).

## 3. New logo — circle-badge rocket

A rocket-launching mark in a blue ring, with:
- an animated blue "comet" orbiting the ring (a fume trail tracing the circle)
- an animated orange/amber exhaust puffing at the rocket's base
- a few faint stars behind it
- colors driven entirely by CSS variables, so it recolors automatically between
  light/dark theme and any accent choice

### Markup (drop-in, place anywhere — currently used in `.nav-brand`)

```html
<svg width="40" height="40" viewBox="0 0 60 60">
  <circle cx="30" cy="30" r="27" fill="none" stroke="var(--blue)" stroke-width="1.8" opacity="0.4"/>
  <circle cx="12" cy="20" r="1" fill="var(--text)" opacity="0.4"/>
  <circle cx="17" cy="14" r="0.7" fill="var(--text)" opacity="0.3"/>
  <circle cx="10" cy="32" r="0.8" fill="var(--text)" opacity="0.35"/>
  <g class="orbit">
    <circle cx="30" cy="3" r="2.4" fill="var(--blue)"/>
    <circle cx="23.9" cy="4" r="1.9" fill="var(--blue)" opacity="0.6"/>
    <circle cx="18.5" cy="6.8" r="1.4" fill="var(--blue)" opacity="0.35"/>
    <circle cx="13.9" cy="11" r="1" fill="var(--blue)" opacity="0.18"/>
  </g>
  <path d="M30 8C27.5 13 26 19 26 26C26 32 26.8 37 28.3 41H31.7C33.2 37 34 32 34 26C34 19 32.5 13 30 8Z" fill="var(--text)"/>
  <path d="M26 29C22.5 30.5 19.5 34 19.5 40L27 36.5Z" fill="var(--muted)"/>
  <path d="M34 29C37.5 30.5 40.5 34 40.5 40L33 36.5Z" fill="var(--muted)"/>
  <circle cx="30" cy="19" r="2" fill="var(--bg)"/>
  <path d="M26.5 41.5L30 51L33.5 41.5Z" fill="var(--accent)"/>
  <path d="M28 41.7L30 47L32 41.7Z" fill="var(--accent-2)"/>
  <circle class="rfume" cx="30" cy="42" r="1.6" fill="var(--accent)"/>
  <circle class="rfume rfume2" cx="27.5" cy="42.5" r="1.1" fill="var(--accent-2)"/>
  <circle class="rfume rfume3" cx="32.5" cy="42.5" r="1.1" fill="var(--accent-2)"/>
</svg>
```

### Required CSS (already added to `ci-styles.css` / the inline `<style>` in `CI Portal.html`)

```css
.orbit { transform-origin: 30px 30px; animation: orbitspin 3s linear infinite; }
@keyframes orbitspin { 100% { transform: rotate(360deg); } }

.rfume { transform-origin: center; animation: rfumePuff 1s ease-out infinite; }
.rfume2 { animation-delay: .3s; }
.rfume3 { animation-delay: .6s; }
@keyframes rfumePuff {
  0%   { opacity: .9; transform: translateY(0) scale(0.6); }
  100% { opacity: 0;  transform: translateY(6px) scale(1.8); }
}
```

### Usage notes
- Size freely via the `width`/`height` on the outer `<svg>` — the `viewBox="0 0 60 60"`
  keeps proportions correct at any size.
- Needs `var(--text)`, `var(--muted)`, `var(--bg)`, `var(--accent)`, `var(--accent-2)`,
  `var(--blue)` to be defined in scope (already true anywhere inside this app's
  `[data-theme]` root).
- To use it standalone in another project, replace the `var(--...)` fills with fixed
  hex values, or define those custom properties on an ancestor element.
- Alternate concepts explored (initials integrated into rocket parts, rocket bursting
  through a "D") are kept in `Logo Concepts.html` for reference.
