---
name: brand-image-red-team
description: Prompt engineering and red-team review for AI-generated brand imagery (social kits, banners, thumbnails, backgrounds, portraits). Use when asked to generate, improve, critique or quality-check AI images for a brand, to write image-generation prompts, or to compare generated brand work against award-level design.
---

# Brand image prompting and red team

Generated imagery fails brands in predictable ways. This skill gives a production pipeline, a prompt anatomy, a red-team rubric and integrity rules. Use it with `wolff-olins-brand-craft` for strategy and identity decisions.

## 1. Integrity rules (non-negotiable)

1. **No synthetic people presented as real.** Never generate a founder, client or customer face for public use. Edit a genuine photograph only, with identity locked, and keep the original.
2. **No fabricated social proof.** Mock-ups must use placeholder metrics ("0 followers", "Sample") or none. Invented follower counts, reviews or logos are misleading if they escape a presentation.
3. **No legible fake documents.** Generators invent citations, case names and statutes. For legal, medical or financial brands, any page text in an image must be illegible blur or real, verified, licensed text added later.
4. **Disclose material edits** where the platform or the brand promise requires it.

## 2. Plate-first pipeline

How professional studios use generators: AI makes parts, designers assemble them.

| Step | Tool type | Output |
|---|---|---|
| 1. Real photography | Photographer, phone with tripod and window light | Genuine portraits, workspace shots |
| 2. Plates | Image generator | Text-free backgrounds, objects, textures, marks |
| 3. Edits | Image editor or edit model with identity lock | Background swaps, relighting of real photos |
| 4. Typography and layout | Figma, Canva, Affinity, InDesign | All words set as live text in brand fonts |
| 5. Templates | Components with locked styles and safe-area guides | Reusable files per channel |
| 6. Red team | Section 4 rubric, vision-model check, five people | Pass or fix |

Never ask a generator to produce a finished branded asset with words in it. Type rendered by a generator is unreliable and cannot be edited.

## 3. Prompt anatomy (ten slots)

Write prompts as concrete instructions, not adjective lists. Quality words ("award-winning", "premium", "exceptional sharpness") have weak, unpredictable effects. Nouns, optics, light and exclusions have strong ones.

1. **Purpose and format:** what the plate is for, aspect ratio, where text will go.
2. **Subject:** one concrete subject, named precisely.
3. **Composition:** placement by thirds, empty areas, crop safety.
4. **Camera or render:** lens, distance, angle, depth of field. Say "photograph" or "render" deliberately.
5. **Light:** source, direction, hardness, colour temperature.
6. **Materials and surfaces:** physical and specific (uncoated paper, red enamel, brushed steel).
7. **Palette with roles:** hex values with a role and rough proportion. Ask for neutral reflections so accents keep their hue.
8. **Style anchors:** describe technique (editorial still life, Swiss poster), not living artists' names.
9. **Exclusions:** the category clichés, the house avoid list, text, people, glow, specific colours.
10. **Output check:** what must be true for the image to pass.

Contradictions to avoid: asking for metallic or chrome reflections and exact brand hues at the same time (metal reflects its surroundings and shifts hue); "cinematic" and "true colour" (cinematic grading pushes teal and orange); "minimal" with five effects.

## 4. Red-team rubric (score 0, 1 or 2 each)

| Test | Pass condition |
|---|---|
| Identity truth | Any person is genuine and unaltered in identity. |
| Integrity | No invented metrics, endorsements, documents or citations. |
| Palette fidelity | Sampled dominant colours match the brand hex values within a small difference; no foreign hues. |
| Contrast | Every text and background pair computed at 4.5:1 or above (3:1 for large text). |
| Platform blend | Brand accent is distinguishable from the platform's own button colours. |
| Cliché | Swap test: replace the brand name with a competitor's. If it still fits, fail. |
| Generator tells | No silk waves, bokeh orbs, glowing lines, chrome heads or garbled UI text unless deliberately art-directed. |
| Consistency | Same face, same palette and same type treatment across every asset. |
| Small-size test | Headline and hook read at 168 px wide (sidebar thumbnail) and in a 110 px circle. |
| Safe areas | Channel safe areas respected (recheck the platform's current help pages). |

Total 20. Ship at 17 or above with no zero in the first two rows.

## 5. Vision-model QA prompt

> You are a strict brand red-team reviewer. Assess the attached image against this brief: [paste brief and hex palette]. For each test (identity truth, integrity, palette fidelity with sampled hex values, forbidden elements, text artefacts, safe area, legibility at 168 px wide, cliché risk) give a score of 0 to 2, evidence and a fix. Then give a pass or fail. Do not soften findings.

## 6. Award-level benchmark questions

- What single idea does this image express, and could only this brand own it?
- Is "premium" coming from craft (type, spacing, photography, restraint) or from effects (gloss, glow, gradients)? Effects date fastest.
- Would a design-literate viewer recognise the image as a generator default?
- Does the image prove the brand promise or only claim it?
