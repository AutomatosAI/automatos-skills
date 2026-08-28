# SKILL — Automatos Editorial Plates

A reusable visual vocabulary for blog covers, social cards, doc figures, and
in-page illustrations. Every plate is a self-contained HTML snippet that:

- Uses **CSS custom properties** so it inverts automatically when the host
  page toggles between bone (light) and pitch (night) modes.
- Reads as a **printed artifact** — a watch bill, a cutaway, a sea chart, a
  field note — not a generic UI card.
- Carries the brand without needing a logo.

---

## When to reach for a plate

| Need                                  | Plate                |
|---------------------------------------|----------------------|
| Roster / list of agents / "the crew"  | **Watch bill**       |
| System architecture / "what's inside" | **Cutaway**          |
| Routing / decisions / journey         | **Sea chart**        |
| Single insight / metric / quote       | **Field note**       |
| Hero illustration / story opener      | **Pharos** (light + ship — commission art) |

If a plate doesn't fit — don't force it. Use a clean type-only block instead.
The plates are punctuation, not paragraphs.

### Decision tree

Run the post through this in order. Stop at the first match.

```
1. Is there ONE number, quote, or insight that carries the post?
     → field-note. Done. (one plate is plenty.)

2. Does the post enumerate 4–10 named things (agents, tools, people, steps)?
     → watch-bill. Use the names verbatim; do not invent rows to fill.
        Fewer than 4? Inline list. More than 10? Split, or pick a different plate.

3. Does the post describe a SYSTEM with 2–5 named parts and arrows between them?
     → cutaway. The SVG is the spec — every labelled circle must be a real
        component named in the post.

4. None of the above?
     → no plate. A serif pull-quote on a rule beats a forced figure.
```

**Plates per post:** 1 is great. 2 is the cap. 3+ turns the post into a catalogue.

---

## Tokens

Pull these from the host page (already defined on `body[data-mood]`):

```
--bg            page background
--bg-2          card background
--fg            ink
--fg-2          muted ink
--muted         label/caption ink
--rule-c        light rule
--rule-strong   strong rule
--accent        sage (bone mode) / warm gold (pitch mode)
--accent-warm   stamp/beam highlight
--serif         "Instrument Serif"
--sans          "Geist"
--mono          "Geist Mono"
```

Never hardcode a hex. If you need a colour the tokens don't cover, propose a
new token in a PR — don't sneak it into a plate.

---

## Rules of the form

1. **Title** in serif italic, bottom-aligned baseline, with a small mono
   `<small>` tag at the right (volume / plate number / date).
2. **Subtitle row** in 9.5px mono uppercase, letter-spacing .16em, sage muted.
3. **Body** — table, SVG diagram, or grid — with dashed dividers and the body
   rule strong only on the title underline.
4. **Stamp** — circular, rotated -8°, sage outline, ~55% opacity. Always
   off-axis (top-right or bottom-left).
5. **Foot** — mono uppercase, justified `space-between`, top-rule .12 alpha.
6. **Caption** under the plate — two-column mono uppercase: short label /
   `Plate · NNN`.

The plate IS the figure. Don't add a heading above it; the figure caption
below is the heading.

---

## Usage

```html
<link rel="stylesheet" href="/automatos/plates.css" />

<aside class="chapter-aside">
  <div class="plate plate-watchbill">
    <div class="wb-title">Watch bill <small>Vol. 19 · Wk 04</small></div>
    <div class="wb-sub"><span>Crew · 100+ rated</span><span>Drawn · K. Halford</span></div>
    <table>
      <thead><tr><th>No.</th><th>Name</th><th>Rating</th><th>Wtch</th></tr></thead>
      <tbody>
        <tr><td class="num">01</td><td class="name">Sentinel</td><td>Code Reviewer</td><td class="watch">A</td></tr>
        <!-- 6–8 rows total; more than 10 is a different plate -->
      </tbody>
    </table>
    <div class="wb-stamp">Mustered<b>04 · 26</b></div>
    <div class="wb-foot"><span>Marketplace · 100+</span><span>Bring your own</span></div>
  </div>
  <div class="photo-caption">
    <span>Fig. 01 — Watch bill</span><span>Plate&nbsp;/&nbsp;001</span>
  </div>
</aside>
```

See `examples/` for one fully-worked plate per type. Copy, retype, ship.

---

## When generating a plate from a blog post

The blog-creator agent should:

1. **Pick one plate** that maps to the post's spine (architecture? watch bill.
   how-to chain? sea chart. one metric? field note).
2. **Lift exact strings** from the post — never invent numbers. If the post
   doesn't have a roster, *don't generate a watch bill*.
3. **Caption it** with `Fig. NN — <noun>` and `Plate · NNN`. Number plates
   per-post starting at 001.
4. **Hand back HTML** — not Markdown, not screenshots. The host inlines it.

Refuse cheerfully if no plate fits. A plain pull-quote in serif italic, set
on a single rule, beats a forced cutaway every time.

---

## Don't do this

These are the four failure modes we keep seeing. If you catch yourself doing
one of them, back out and re-run the decision tree.

### 1. Forcing a plate when the post doesn't have one

❌ Post is a 400-word essay on "why we like sea metaphors" — agent generates
   a watch bill of fake agents because *watch bill* sounds nautical.

✅ No plate. A serif pull-quote. The aesthetic is in the type, not the
   figure.

### 2. Inventing numbers / names to fill a plate

❌ Post mentions "a few agents" — agent fabricates `Sentinel, Halberd, Cooper,
   Pemberton, Marlow` to fill a 6-row watch bill.

✅ Use only the names the post names. If the post has 2 names, the plate has
   2 rows. If it has 0, pick a different plate.

### 3. Using a plate as a section header

❌ Plate sits flush above an `<h2>` reading "The Crew" — same words twice,
   plate is decorative.

✅ The plate **is** the section break. The caption underneath (`Fig. 01 —
   Watch bill / Plate · 001`) is the header. Don't double it up.

### 4. Hardcoding colours

❌ `style="background: #F2EDE4; color: #0F1411"` baked into the plate HTML.

✅ Let the tokens do it. The plate inherits `--bg` / `--fg` from the host
   page and inverts for free under pitch mode. If you need a colour the
   tokens don't cover, that's a token PR — not an inline override.
