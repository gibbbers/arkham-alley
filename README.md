
# arkham alley

goal is to make a homemade batman arkham knight style game with a unique plot/storyline. currently it is as follows: 
-A 1 file tech demo in three.js to deliver the first scene.
-which is a gotham-flavoured alley, three fights with crooks, and a two-hander with the joker at the far end. 
-everything is primitive geometry in three.js
-in the process of developing the map using primitive geometry and physics

**`index.html` is the whole project currently .** two ai generated audio recordings are embedded

## Running it

ES modules are blocked over `file://`, so it needs a local web server:

```bash
python3 -m http.server 8099 --directory .
```

Then open <http://localhost:8099> and click *begin* (browsers won't start audio
without a click).

##  sequence — 57.6s, loops

| time | beat |
|------|------|
| 0.6–7.0s | Batman walks in through the rooftop god ray |
| 7.0–11.3s | fight 1 — haymaker slipped, jab parried, counter cross |
| 12.4–16.7s | fight 2 — jab parried, haymaker ducked, counter cross |
| 17.8–22.1s | fight 3 — two swings evaded, counter hook |
| 23.2–28.6s | walks on past the bodies |
| 28.6–30.8s | camera whips around behind him; the Joker steps out |
| 32.2–40.5s | Batman's line + caption |
| 40.9–54.8s | the Joker's line + caption |

## rendering

PBR materials throughout, ACES Filmic tone mapping with sRGB output, and
physically-based light units. Post stack is `RenderPass → GTAO → Bokeh DOF →
UnrealBloom → OutputPass → custom grade` (vignette, film grain, chromatic
aberration, split-tone, impact flash and radial smear).

The wet ground is a `Reflector` mirror under a transparent PBR asphalt sheet
whose alpha map punches puddle holes; the mirror runs a custom shader with
scrolling ripple distortion. Ambient reflections come from a procedural
equirect environment through `PMREMGenerator`.

Textures are all synthesised at load from tileable value-noise fBm — brick,
concrete, wet asphalt with a puddle mask, worn metal, graffiti — each with
derived normal and roughness maps. Build takes ~3s.

## performance

Pixel ratio caps at 1.25, the puddle mirror runs at 640² on alternate frames,
shadow maps refresh every other frame, and there's a two-step automatic
fallback (drop to 1×, then disable AO and DOF) if the frame rate sags.

## poking at it

`ALLEY.seek(10.26)` in the console jumps to any moment — that one is the first
counter landing. `ALLEY.scene`, `ALLEY.camera`, `ALLEY.batman`, `ALLEY.crooks`
and `ALLEY.audio` are exposed too.

---

## `city/` — open-world Gotham (in progress)

`city/index.html` is a fork of the cutscene that grows a playable open world
onto the same rendering foundation. One renderer, one `EffectComposer`, one
post stack — the city scene is swapped onto the same `RenderPass` / `GTAOPass`
/ `BokehPass` objects, so it cannot visually diverge from the cutscene.

```bash
python3 -m http.server 8100 --directory city
```

`?act=city` boots straight into the world; **G** toggles between the cutscene
and the city. Fly with **WASD**, drag to look, **Shift** to boost.

**Built so far**

- Night sky with a real-radius moon, stars, camera-anchored dome, and haze-
  matched fog. Wet asphalt with a 70m puddle mirror that rides the camera.
- 20×20 blocks at 74m pitch — **1480m across**, roughly 50–60s to cross by
  chained swing. 469 rooftops, 304 of them above 90m.
- Windows are baked into procedural facade textures with emissive maps rather
  than built as geometry: ~12 draw calls instead of ~40,000.
- Ten facade variants (dark brick, soot concrete, pale limestone, ribbon-glazed
  glass) and archetypes for slabs, art-deco setbacks, twin-mass lots, tapered
  glass towers, round/octagonal towers and gothic twin-spire halls.
- Every other block is forced to carry a tall mass, so there is always another
  high roof within grapple range — verified: zero orphaned rooftops at 120m.

**Still to come:** ground-level life (traffic, stoplights, pedestrians, parking
lots, a basketball court), City Hall with a full interior, player traversal
(momentum swing, glide, dive, falling POV), and the cutscene→gameplay handoff.
