# Skylight — atmospheric optics lab

A real-time Monte Carlo simulator of how particles in the sky disperse light.
Open `index.html` in any modern browser — no build step, no dependencies, one file.

Every frame the GPU fires up to a few million photons from the light source. Each
photon gets a random wavelength, hits a randomly oriented particle at a random
point, and is traced with geometric optics — Snell refraction with per-material
Cauchy dispersion, Fresnel reflection roulette, total internal reflection — plus
Airy-theory wave-optics hooks for small droplets. Exit directions accumulate into
a sky radiance map viewed by a free camera. Nothing is painted on.

## Phenomena (all emergent)

Rainbows (primary/secondary, Alexander's band), supernumerary fringes, fogbows,
coronae, a stylized glory, 22°/46° halos, sundogs and the parhelic circle, upper
tangent arcs, Parry arcs, Lowitz arcs, circumzenithal arcs, light pillars,
odd-radius halos (9°/18°/20°/23°/24°/35°) from pyramidal crystals built from the
real ice lattice geometry, moon halos and moonbows under mesopic (rod) vision
with a procedural star field, and a near-field streetlamp-in-snow mode.

## Controls

- **Drag the sky** to look around; **scroll** zooms; projections: normal, fisheye, all-sky.
- **Drag the ☀/☾/⛯ handle** to move the light source (shift-drag works anywhere).
- **Particle panels**: shape cards (droplet, hex plate/column/mix, pyramid, prism, cube),
  orientation (random / plates / columns / Parry / Lowitz) with wobble, aspect ratio,
  material (water, ice, crown glass, fused silica, diamond, sapphire), droplet size
  (5 µm–1.5 mm — small drops fade in the wave optics), pyramid fraction.
  A second population can be mixed in for compound displays.
- **Light**: sun / moon / streetlamp, full-spectrum or single-wavelength laser,
  dispersion exaggeration, double-scatter probability, exposure, bloom.
- **▶** sweeps the sun across the sky; **📷** saves a PNG; the full state lives in
  the URL hash for sharing. Angle guides + hover readout for measuring.
- 15 scene chips set up everything, from Rainbow to Streetlamp snow.

URL parameters: `?scene=<chip-id>` (rainbow, super, fogbow, corona, glory, halo,
sundogs, pillar, cza, oddradius, parry, diamonddust, moonhalo, moonbow, streetlamp),
`&burst=N` extra first-frame passes, `&guides=1`, plus `#s=…` full state.

## Implementation notes

- WebGL2, single file. Monte Carlo runs in vertex shaders (one point = one photon)
  splatting into an equirect RGBA32F radiance map with additive blending; ACES display
  pass composites everything on one radiometric scale (L/E_sun; zenith Rayleigh at
  sun 40° anchored to mid-gray): an 8-node spectral single-scattering sky
  (Hansen-Travis Rayleigh, Chappuis ozone, Henyey-Greenstein aerosol, Kasten-Young
  airmass, empirical twilight), a cloud slab model for the phenomenon layer
  (transmission/reflection/immersed geometries + delta-Eddington multiple-scatter
  veil — Gedzelman's optical-depth visibility bands emerge from it), per-photon
  sunbeam depletion (halos redden at sunset), a true-radiance sun disk that ACES
  swallows into overcast near τ≈8, night sky (Larson-1997 scotopic blend, star
  field), and map-space bloom. Programs are split per particle family and linked
  with `KHR_parallel_shader_compile` to keep D3D shader-compile stalls off the UI
  thread.
- The sky map is stored sun-azimuth-relative: azimuth drags are free; other parameter
  drags decay the accumulation (`blendColor` trick) into a live motion trail.
- The streetlamp mode is a separate reverse single-scatter path tracer: photons pick a
  screen pixel, sample a scattering distance (uniform ⊕ equiangular mixture), trace the
  reversed ray through a crystal, and accept iff the exit hits the cone subtended by the
  frosted globe — unbiased for an extended Lambertian emitter, weights hard-bounded, with
  a blur-floor cone for random orientations. It converges progressively while you watch.
- Physics constants (ice lattice pyramid angles, Cauchy fits, Airy fringe positions and
  lobe energies, corona/glory scales) were derived and source-verified in
  design specs; see the About dialog for what is approximated (the glory, notably).
- ANGLE/FXC constraint: shader loops keep constant bounds with flag guards, crystal
  planes arrive as a 20-entry uniform array. Do not reintroduce dynamic breaks — a
  nested-dynamic-loop shader once hard-wedged a GPU process for a whole session.
- Headless verification: `chrome.exe --headless=new --screenshot=… "…index.html?scene=X&burst=200"`.

## Known limits

Single/double scattering only (no multiple-scatter fog), unpolarized Fresnel,
glory is stylized, Lowitz arcs use the classic uniform-spin model (over-extended
vs. most real displays), lamp mode is single-scatter with an analytic ground.
