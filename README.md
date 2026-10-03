# Twisted

*A neighborhood romance — dual POV, branching, zero dependencies.*

He's the noise moving in next door. She's the reason he doesn't mind.

**Twisted** is a browser-based choice-driven romance game. Chapter 1 — *9:00 PM Sharp* — follows **Adeline** (the neighbor with the sleep mask) and **Zade** (the one with the power drill) from a 10 AM doorstep war to a candlelit dinner where one choice decides everything: kiss her, say the true thing, or ruin it with a joke.

## Play

No build, no install:

- **Double-click `index.html`**, or
- serve the folder (`python -m http.server` / `npx serve`) and open the URL.

Progress autosaves in your browser (localStorage). **Continue** resumes where you left off.

## What's inside

| Path | What it is |
|---|---|
| `index.html` | App shell — title screen, cinematic prologue, game screen, end screen |
| `js/intro.js` | Five-card cinematic opening with Adeline and Zade’s Chapter 1 backstory |
| `css/style.css` | Bright visual-novel UI (warm paper panels, berry/steel speaker tags, full-bleed character framing) |
| `js/engine.js` | Story engine — scenes, flags, stats, conditionals, autosave |
| `js/ui.js` | Presentation — typewriter text, character staging, choices, and optional native speech voices |
| `js/visuals.js` | Procedurally drawn SVG backdrops and foreground furniture, plus the wardrobe system that cuts each portrait into its scene outfit |
| `js/story/chapter1.js` | The whole of Chapter 1 as a scene graph |
| `js/story/visual-map.js` | Scene-by-scene background, outfit, pose, expression, props, and blocking cues |
| `assets/` | Supplied character portraits (`.jpg`), background-cleaned at runtime for compositing |
| `assets/outfits/` | Optional real outfit photos, listed in `manifest.json`; falls back to the cut silhouette when empty |
| `tools/` | Dev scripts (asset generator, story auditor) |

## Chapter 1 — structure

Dual POV in three acts:

- **Act I — Hibernation** *(her side)*: the 10 AM doorstep war, played from Adeline's door.
- **Act II — The Transformation** *(her side)*: the hallway negotiation, dinner terms, "I steal hearts."
- **Act III — 9:00 PM Sharp** *(his side)*: cooking, the knock — early, sharp, or late (she locks the door) — dinner, the fire escape, and the kitchen moment where it all hinges on you.

54 scenes · 19,683 complete choice paths in the current graph · 3 chapter closes (Spark / Smolder / Cold Truce) · the note under the door — and whose handwriting is on it — depends on how you played.

Mature themes, sensual tension, nothing explicit. Everyone in this story is fictional; behave accordingly out here.

## Editing the story

Scenes live in `js/story/chapter1.js`. A scene:

```js
my_scene: {
  pov: "Adeline",              // whose head we're in (drives HUD + portraits)
  mood: "dinner",              // color grade (see MOODS in style.css)
  beats: { left: "Adeline" },  // optional portrait override (left/right/null)
  lines: [
    "*Her thought, italicized.*",
    "~A softer-spoken line.",
    "Zade: \\\"A spoken line by Zade.\\\"",
    "{if:kissed_dinner}Shown only if the flag is set.{else}Otherwise this.{endif}",
  ],
  choices: [
    { label: "Close the distance. Kiss her.", hint: "the risky one",
      goto: "next_scene",
      effects: [{ set: "kissed_dinner", value: true }, { stat: "attraction_ad", add: 2 }] },
  ],
  next: "some_scene",          // or null to end the chapter
  effects: [{ stat: "trust", add: 1 }],
}
```

Rules of the house: flag conditions are `{if:flag}…{else}…{endif}` or `{if:trust>=2}…{endif}`; every choice needs a `goto`; endings use `next: null`.

Scene staging is kept separately in `js/story/visual-map.js`. Each of the 54 story scenes maps to an illustrated backdrop and character look (outfit, pose, expression), plus transition/exit cues and set dressing. The supplied portraits remain the base models; clothing is re-cut over the photograph at runtime (night-suit, blazer and camisole, sweater and jeans, denim and tee, charcoal suit, joggers, and the intimate robe / leather-jacket looks) while the flat studio background is removed locally.

### How the wardrobe works

Recolouring alone can only ever produce the one garment the model happened to be wearing, so each look is **cut, not tinted**. A look is a list of parts — bodice, sleeve, skirt/trousers, wedge, band — each written as a signed-distance silhouette and filled from a donor patch of the same photograph. The donor supplies the light and folds; the silhouette supplies the shape. That is what lets the night-suit, tailored looks, casual layers and intimate catalog-inspired outfits share one set of models.

Looks are chosen off the supplied outfit catalog and pinned to the beat they
belong to — the story specifies the garment, not the wardrobe:

| Look | Catalogue pick | Why this beat |
| --- | --- | --- |
| `Adeline:sleepwear` | **1. Satin black night-suit** — white piping on placket, cuffs and turn-up | She literally leans on the doorframe in her night-suit. Satin gets its fold contrast pushed to 1.34, because the sheen *is* the highlight along each crease |
| `Adeline:work` | **5. Black blazer + trousers** — worn open over an ivory camisole | Act II is titled *"armor is chosen carefully"*; the lapels are cut away so the light top shows down the whole front |
| `Adeline:evening` | **4. Cream sweater + jeans** — oversized knit, ribbed hem, straight denim | *"It's noodles. It's discipline, not a date."* She is deliberately **not** dressed up. The softest thing she owns on screen |
| `Zade:casual` | **7. Denim jacket + graphic tee** — light-wash denim open over a near-black tee | Moving day. The only layered look in the cast: two garments on the top of the body instead of one |
| `Zade:evening-casual` | **4. Charcoal suit, no tie** — white shirt open at the throat under the jacket | She takes his collar, then fists in his shirt. The scene needs a real shirt under a real jacket |
| `Zade:midnight` | **2. White tee + grey joggers** — gathered at the ankle | He is alone on a mattress on moving day, so the denim comes off |
| `Adeline:romantic` | **3. Burgundy satin robe + matching lace set** — the front stays open so the set reads through it | A deliberately intimate catalog change for the close / honest scene branch |
| `Zade:romantic` | **1. Open black leather jacket + bare chest + dark jeans** | Adds contrast and closeness to the romantic branch; his jacket panels remain open around the photographed chest |

Other catalog items (saree, gym set, winter coat, tuxedo, trench, kurta,
puffer, slip dress, floral sundress) stay unused for Chapter 1.

Details that make it hold up:

- **Anchored to the photo.** Every garment is clipped to the photographed figure, so a look can cover less of the model than the studio shot does but can never grow out over the room. `band` and `grow` in [js/visuals.js](js/visuals.js) set how far past the silhouette it may reach.
- **Skin stays visible where intended.** Warm-hued pixels are classified once per photo into a mask; cloth parts can leave hands and exposed skin alone inside a `keepSkin` row window, while garments that deliberately replace skin — trousers over bare legs — simply omit it.
- **Real tonal separation.** Source lightness is stretched across the donor's own dark range rather than mapped straight from 0–1, so near-black fabric keeps visible folds instead of washing out, then is lit as a body (rounded across, shadowed under the hem above, darker at the turning edge).
- **Antialiased edges.** Signed distances give free soft edges and cheap subtraction, which is how the blazer's open front, the shirt's open collar, and the romantic robe's split front are cut.
- **Branch-specific romance looks.** Dinner and the joke deflection keep the cream sweater and charcoal suit. The burgundy robe and open leather-jacket looks appear at the kiss choice and along the honest-answer branch.

Adding a look is a data change: append a part list to `GARMENTS` in [js/visuals.js](js/visuals.js) and name the new `outfit` in [js/story/visual-map.js](js/story/visual-map.js). The per-scene `cue` strings name the garment on screen, so the label and the pixels cannot drift apart. The renderer adapts backgrounds to portrait mobile screens without page overflow.

Any look can be swapped for a real photograph instead. Drop the image into `assets/outfits/` and add one line to `assets/outfits/manifest.json`; the imported shot replaces both the base portrait and the generated garment for that look, and every other look keeps its cut. A re-shot body rarely sits in the frame like the original, so an entry can carry `crop`, `offsetX` and `scale` to re-register it — applied to the portrait slot rather than the image, because the portrait, the garment layer and the import all resolve one shared geometry from there. See [assets/outfits/README.md](assets/outfits/README.md).

Presentation follows a visual-novel layout: characters are cut out at runtime and framed full-bleed, running from the top of the frame to roughly mid-thigh, with the dialogue as a warm paper panel carrying dark ink, a coloured name tag on its top edge, and a “Tap to continue” prompt. Backdrops are drawn as layered, furnished rooms in bright saturated palettes, and each room also gets a *near* layer — a shag rug, a bed footboard, a desk lip, a counter edge, a stack of moving boxes, a fire-escape rail — drawn in front of the figures so the room visibly continues past them instead of leaving them pasted onto an empty floor. That foreground is measured in CSS pixels rather than a fixed viewBox, because a `slice` crop would eat a rug whose near edge sits at the bottom of a 16:9 design space on a phone in portrait; measuring means it lands at the same fraction of the frame at every aspect ratio, and it is re-measured on resize. New Game opens a skippable, five-card cinematic prologue with the leads’ backstory. Browser-native voiceover reads spoken character dialogue (including Priya’s messages) and can be toggled, replayed, or assigned separate system voices in the menu; support and available voices depend on the browser/OS.

## Dev tools

```bash
node tools/story-audit.js       # walks every choice-path; proves all branches end & are reachable
node tools/generate-assets.js   # optional procedural PNG placeholders; game uses supplied JPG portraits
```

## Roadmap

- **Chapter 2 — The Letter**: consistency, virtue three, and what's in the box he didn't label.
- Chapter 3+: forced proximity, backstories, the escalation (with taste — see above).

---

*Built with nothing but HTML, CSS, and spite for build steps.*