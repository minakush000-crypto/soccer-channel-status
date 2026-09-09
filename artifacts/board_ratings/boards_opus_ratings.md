# Board appearance ratings (Opus 5, authoritative judge)

Rated against the RESUME_RESEARCH.md / Coaches' Voice benchmark (7 criteria):
(1) layered depth; (2) desaturated pitch + high-contrast accents;
(3) selective visibility; (4) functional arrow language;
(5) contextual cropping; (6) condensed athletic fonts; (7) dark muted palette.

Source frames (640px, full-state) in frames/2026-09-06_arsenal-chelsea/:
board_formation_full.png, board_possession_full.png, board_stat_card_full.png

Per the user's instruction (Part 4b), appearance was RATED ONLY, not fixed.

## Scores

| Board | Opus score |
|---|---|
| formation | 3/10 |
| possession | 4/10 |
| stat_card | 5/10 |

---

## FORMATION board — OVERALL: 3/10

**1. Layered depth — 2/10**
Completely flat. The tokens are single-fill circles with a hairline stroke; no drop shadow, no inner gradient, no rim light, no separation between marker layer and pitch layer. Worse, z-ordering is unmanaged — tokens sit *on top of* the penalty-arc and penalty-box lines, and at least two pairs (left of the Arsenal box, and the pair on the right touchline-side of the Chelsea box) visibly collide/overlap with no halo or offset to resolve them. Coaches' Voice boards always float the player chip above the turf with a soft shadow so it reads as a physical counter.

**2. Desaturated pitch + high-contrast accents — 6/10**
The best-performing criterion. The near-black forest turf is genuinely desaturated and the red/blue chips pop. But the accents are raw primary red and primary blue rather than a curated brand pair, so they read as "default plot colours" instead of a designed palette. The blue also loses contrast against the dark green more than the red does, so the two teams are not equally legible.

**3. Selective visibility — 3/10**
Everything is shown and nothing is said. All 22 players, both full halves, the halfway circle, both penalty areas, both six-yard boxes — with zero hierarchy. No shape banding, no unit shading, no dimming of the opposition, no highlighted focal player. A formation board's whole job is to make a *shape* visible; here the shape is left for the viewer to reverse-engineer from scattered dots, several of which are positionally implausible.

**4. Functional arrow language — N/A**
No arrows on the board.

**5. Contextual cropping — 3/10**
Full-pitch, dead-centre, symmetrical framing with a wide dark gutter on all four sides. No zoom into the zone under discussion, no asymmetric crop, no tilt or perspective. It's a wallpaper, not a broadcast graphic.

**6. Condensed athletic fonts — 2/10**
The most immediately amateurish element. The title is a default bold grotesque (DejaVu Sans / Arial-class). Nothing condensed, no uppercase tracking, no Bebas/Barlow verticality. The shirt numbers inside the chips are rendered at ~5px and are functionally unreadable.

**7. Dark muted palette — 6/10**
Base is correct: deep near-black green, off-white lines at reduced weight. Loses points: uniform-width slightly-too-bright line work, no vignette/turf texture, fully saturated chip colours break the muted discipline.

What reads as amateurish: matplotlib fingerprints everywhere (default font, hairline uniform strokes, centred title, tiny grey caption where a plot x-label goes) — this is a scatter plot with a pitch behind it, not a tactics board; chips colliding and clipping pitch lines (no z-order/collision pass); unreadable numbers in 8px circles; ~20% dead black margin; no narrative furniture (no scoreline lockup, crest, formation caption, lower-third).

## POSSESSION board — OVERALL: 4/10

**1. Layered depth — 2/10**
Single flat plane. No drop shadow, no inner bevel, no gradient, no separating stroke where red meets blue. Zero z-ordering: title, labels and bar all read as stickers on the same layer.

**2. Desaturated base + high-contrast accents — 5/10**
Background (deep blackened green) is good. But the accents are fully saturated red and royal blue at full chroma, both equally loud — no hierarchy of which team dominates.

**3. Selective visibility — 6/10**
No gridlines, axis, legend clutter, logos — right instinct. But over-corrects into emptiness: the entire bottom 40% is dead space with no supporting metric.

**4. Functional arrow language — N/A**

**5. Contextual cropping — N/A / weak**
The bar is stranded upper-middle with unbalanced margins; the composition drifts.

**6. Condensed athletic fonts — 3/10**
Default system bold (Arial/Helvetica) with tracking. No Bebas/Barlow. Percentage figures (the hero numbers) are the same weight/size as the title — flattened hierarchy. Team names are tiny. Type scale is inverted.

**7. Dark muted palette — 7/10**
Strongest criterion. Near-black green base is restrained. Loses points only for unmodulated primary accents.

Amateurish tells: hard 90° butt-join between red and blue (biggest PowerPoint stacked-bar tell); square corners (no rounded caps/pill); misaligned labels (team names inset, not flush to bar); undersized team names floating with no connecting rule; no 50% reference marker (can't read how far past parity Arsenal is); flat fill, no texture/gradient (looks like default Excel series colour); uniform bold weight = no typographic voice.

## STAT CARD — OVERALL: 5/10

**1. Layered depth — 2/10**
Completely flat. Bars are solid single-fill rectangles with hard 90° corners, no drop shadow, no inner glow, no gradient falloff, no ghost/track bar behind each value. One z-plane.

**2. Desaturated base + high-contrast accents — 7/10**
Strongest element. Near-black forest-green is desaturated; crimson/steel-blue read as team identity. Loses points: Chelsea blue is lower-contrast against dark green than Arsenal red, so the right side recedes — card feels lopsided.

**3. Selective visibility — 6/10**
Ten rows is broadcast-standard but not editorial. Passes/Accurate Passes and Shots/Shots on Goal are near-duplicates. Everything weighted identically — states data rather than making a point.

**4. Functional arrow language — N/A**

**5. Contextual cropping — N/A**, but composition: large dead centre channel + wide empty left margin, bars compressed into two narrow gutters. Frame not used.

**6. Condensed athletic fonts — 3/10**
Default UI sans (Roboto/DejaVu from a plotting library), not Bebas/Barlow. Value numerals are wide, round-shouldered, soft. Centre labels small, mid-grey, low-contrast — the vertical spine is the weakest-reading part.

**7. Dark muted palette — 7/10**
Three colours plus white, no rogue hues, no neon. Loses points for flat green (no vignette) and pure-white title slightly hot against the muted body.

Amateurish tells: BOTH bar sets grow left-to-right — Arsenal bars should mirror (anchor at centre / right-align toward the label column) so the teams oppose; the home side's bars run AWAY from the comparison axis, destroying the head-to-head read; numerals colliding with bar ends (Tackles, Interceptions, Corners, Shots on Goal) — inconsistent, some float, some overlap (most PowerPoint-looking flaw); bar scaling reads inconsistent (Yellow Cards 2v4 bar ~55-60% not 50%; Shots 16v13 barely differentiate) so bars are decoration not info; hard square caps; no context furniture (no crests, scoreline, competition mark, minute stamp); uniform row weight (no banding, separators, hierarchy).

Skeleton is right (restrained colour, dark base, team-name headers, vertical rhythm); execution is a first draft.