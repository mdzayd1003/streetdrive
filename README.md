# StreetDrive

Drive down a real street, anywhere Google Street View has been. Type a place, pick a road,
and go — the view is the actual Street View imagery for wherever you are.

**[▶ Live demo](https://mdzayd1003.github.io/streetdrive/)** — bring your own Google Maps API
key; the page asks for it on the first screen.

---

## Why it asks you for a key

There is no key in this repository, and there is not going to be one. A Maps API key in a
public repo is scraped within hours and billed to whoever owns it. So the demo asks each
visitor for their own, stores it in that browser's `localStorage`, and sends it nowhere except
to Google.

Getting one takes about three minutes:

1. **[Get an API key](https://developers.google.com/maps/documentation/javascript/get-api-key)** —
   the official walkthrough.
2. Enable **Maps JavaScript API** on the project
   ([console](https://console.cloud.google.com/google/maps-apis/api-list)).
3. Billing has to be enabled on the Cloud project, even to use the free monthly allowance.
4. Restrict the key to your own sites
   ([credentials](https://console.cloud.google.com/google/maps-apis/credentials)) — under
   *Application restrictions* → *Websites*, add `https://mdzayd1003.github.io/*`. An
   unrestricted key is a key somebody else will spend.
5. **Set a daily quota cap**
   ([quotas](https://console.cloud.google.com/google/maps-apis/quotas)) before you drive.

That last one matters more here than in most Maps projects. Driving loads roughly one panorama
per second, so a ten-minute session is several hundred billed requests. Light personal use
normally lands inside the free monthly allowance and costs nothing, but leave it running and it
will not. Check the current Maps Platform pricing yourself — Google changed the model in 2025
and any figure quoted in a README ages badly.

## How it actually works

Street View is not video. It is a set of discrete panoramas — on roads, typically one every
10–15 metres — and each one knows the compass headings of its neighbours. So "driving" splits
into two halves that behave very differently:

| | continuous? | why |
|---|---|---|
| heading, pitch | **yes** | rotating a textured sphere; genuinely smooth |
| position | **no** | you hop from one photograph to the next |

The loop keeps a normal car model — throttle, brake, drag, speed-dependent steering authority —
and integrates a distance travelled. Each time that distance passes the local panorama spacing,
it picks the neighbouring panorama whose heading is closest to where the car is pointing and
jumps there.

Three details make the difference between that feeling like a drive and feeling like a
slideshow:

**The hop is disguised, not hidden.** Between panoramas the view scales up very slightly and
snaps back on arrival. That reads as forward motion. It is a trick, and at walking pace you can
see straight through it — the imagery simply does not exist at a finer spacing.

**The spacing is measured, not assumed.** Panorama density varies between a dense city street
and a highway. After each jump the code measures how far it actually moved and eases its
estimate toward that, so the car's speed stays honest on both.

**Heading is re-asserted after every jump.** Moving to a new panorama resets the point of view
to whatever Google stored with that photograph. Without setting it back, the car spins to a
random bearing every ten metres.

Steering is where the illusion holds up best: because rotation is continuous, turning feels
right even though the ground beneath you is moving in steps.

## What it does at the edges

- **No coverage nearby** — a city centroid often lands on a rooftop or a river. The search tries
  progressively wider rings out to 3 km before giving up, and says so plainly when it does.
- **Dead end, or facing a wall** — no neighbouring panorama within 62° of where you are pointing.
  The car stops and tells you to steer or press `R`.
- **Indoor panoramas** — filtered out via `StreetViewSource.OUTDOOR`. Shop interiors and user
  photo spheres have no road links, so you would arrive and be stuck.
- **A rejected key** — caught through Google's `gm_authFailure` hook and shown on the page,
  rather than leaving a grey box and an explanation buried in the console.

## Controls

```
W  ↑   accelerate          Q E   look around (springs back)
S  ↓   brake               R     turn around
A D ← → steer              M     minimap
                           Esc   back to the start screen
```

Touch controls appear automatically on a touchscreen.

## Coverage

Street View reached India in 2022 and coverage is uneven between cities — some roads have it,
neighbouring ones do not. The preset list is a starting point, not a promise; if a road has
never been photographed, no amount of code will drive down it.

## Running it locally

It is one file with no build step and no dependencies.

```
git clone https://github.com/mdzayd1003/streetdrive
cd streetdrive/docs
python3 -m http.server 8000     # or just open index.html
```

## Scope, honestly

This is a toy built on somebody else's imagery, and it inherits that imagery's limits: discrete
positions, uneven coverage, no collision, no traffic, no other cars. It is not a driving
simulator. What it is is a demonstration that the Street View panorama graph is navigable in
real time, and that with continuous rotation and a disguised hop, a stack of still photographs
reads convincingly as motion.
