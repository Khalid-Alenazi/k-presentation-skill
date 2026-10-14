---
name: k-presentation-skill
description: Build a self-contained, video-style narrated HTML film (player controls, synced captions, ElevenLabs v4 script with voice picks and audio tags, optional background music, animated SVG scenes, Elm identity).
---

# K Presentation Skill

Turns source material (HTML deck, PDF, notes, screenshots) into ONE self-contained HTML file that plays like a video: timeline engine, animated SVG/canvas scenes, captions, ElevenLabs narration, and optional background music.

## 0. FIRST STEP, ALWAYS: three questions
Before reading files, storyboarding or building anything, the very first action is to ask these three questions together (use AskUserQuestion when available, otherwise plain text) and wait for the answers. Never skip this step, even if the request seems to imply the answers.

1. **Language of narration and captions**
   - Arabic (Modern Standard / فصحى)
   - Arabic (Saudi dialect / لهجة سعودية)
   - English
   - Bilingual (Arabic voice + English captions, or the reverse)
2. **Expected duration**
   - ~2-3 min (quick pitch)
   - ~5 min (standard)
   - ~8-10 min (detailed walkthrough)
   - Other (user types it)
3. **Background music**
   - Yes, add it for me (Claude builds it, see section 8)
   - Yes, I will provide a track
   - No music

The language sets `dir`/`lang` of the content, fonts, caption direction, diagram flow and narration style. The duration sets number of scenes and depth per scene. The music answer sets section 8.

## 1. Collect remaining inputs (one message, only what is missing)
- Source material + topic + audience (e.g. managers, PMs, leadership)
- Speaker model: single narrator OR personas (each product/feature speaks in first person)
- Brand assets: logo (SVG/PNG), colors, department name. If not provided, see section 6. NEVER ask about fonts; fonts are fixed by section 6a.
- Must-keep items: exact numbers, names, wording, real screenshots/recordings
- Existing audio? If yes, sync visuals to that audio instead of writing a new script.

## 2. Understand, then storyboard BEFORE building
Read the whole source. Then send a storyboard table for approval:

| # | Scene | On screen (visual, minimal text) | Narration | Speaker | Suggested voice | ~Duration |

Rules:
- Separate what is SAID from what is SHOWN. The screen is visual; the voice carries the explanation. Never put the narration sentence on screen.
- Max ~12 words of on-screen text per scene (labels, numbers, short titles).
- Typical arc: hook -> problem -> solution(s) -> real example/demo -> impact numbers -> enablers/integration -> closing.
- Pace: Arabic ~130-150 words/min, English ~140-160 words/min. Fit total words to the chosen duration. When cutting runtime, trim silent padding and transitions first, content last.

## 3. Engine (single HTML file)
- No runtime dependencies. Embed fonts, logo and music as base64. Inline all CSS/JS.
- Content direction: Arabic `<html dir="rtl" lang="ar">`; English `dir="ltr" lang="en"`; bilingual follows the voice language, captions use their own direction.
- 16:9 stage scaled to fit viewport (letterbox), dark cinematic background.
- Expose a global controller for control and QA:
  `window.VIDEO = { play(), pause(), seek(sec), sceneAt(sec), T /*total*/, SCENES }`
- Scene object: `{ id, start, dur, speaker, lines:[{t, text}], enter(el), update(el, localT, amp), exit(el) }`. Animations are driven by `localT` (time-based, seekable), never by setTimeout chains or CSS animations that cannot seek.
- One requestAnimationFrame loop computes current time, active scene, active caption line.
- Captions: bottom-center above the control bar, synced to `lines[].t`, fade between lines.
- Audio slots: one `<audio>` per scene file (or one master track + timestamps). If a scene's audio exists, its duration overrides `dur` and all later starts re-flow. If missing, run silently on the timer so the film always plays.
- Voice reactivity: Web Audio AnalyserNode on the narration -> amplitude `amp` (0..1, smoothed) passed to `update()`.

### Player controls (FIXED layout, never mirrored)
The control bar ALWAYS uses `dir="ltr"` regardless of the film's language, like a standard video player:
- **Play/Pause button: bottom-LEFT.** Then time display (current / total) next to it.
- **Fullscreen button: bottom-RIGHT.** Just left of it: **CC button** (captions on/off), then optional music/volume toggle.
- **CC button:** toggles captions (all on-screen spoken text). Shows active/inactive state clearly; default ON. Hiding captions does not affect scene visuals, only the caption layer.
- Progress bar spans between them and fills **left -> right**, with scene markers and hover tooltip of scene name; click/drag to seek.
- Keyboard: Space = play/pause, ArrowRight = +5s, ArrowLeft = -5s, F = fullscreen, C = toggle captions (same as CC button), M = mute music (if music exists).
- Controls auto-hide while playing, reappear on mouse move.
Do not let the RTL content direction flip the control bar. Verify this in QA.

## 4. Personas (when products/features speak)
Each persona gets: name, one accent color, an SVG emblem (motif tied to what it does), a one-line identity, and its own suggested voice (section 7).
- The OUTER RING pulses/scales with voice amplitude.
- The EMBLEM itself animates (rotate, draw, morph) but does NOT scale with the voice.
- Persona intro: emblem draws in (stroke-dashoffset), ring appears, name + one-line role.
- Personas speak in first person ("أنا مسار، أساعدك..." / "I'm Masar, I help you...").

## 5. Visual language
- Prefer animated visualizations over text: SVG line drawing, flowing connectors, counters that count up, nodes lighting up, typing animations for AI prompts, before/after splits, particle fields on canvas.
- Demo scenes: rebuild real examples as animated UI mockups (chat bubbles, cards, tables) using the real content provided; do not invent results.
- Transitions: camera-like (zoom-through, light sweep, morph from previous scene's shape), 0.6-1.2s, eased.
- Opening: strong visual metaphor (e.g. light pillars rising from a digital horizon, one per solution). Closing: everything converges (e.g. personas charging a central core) + logo.
- Numbers: big, animated, with short label; always exactly as the source says.
- In Arabic content, text alignment, diagrams and timelines flow right -> left (this applies to content only, not the control bar).

## 6. Brand identity (Elm)
- Top corner: Elm logo + department name beside it, on every scene except full-bleed opening/closing moments.
- Use the provided logo and palette exactly. If assets are inside an existing HTML, extract them (logo src, colors) instead of re-creating.
- If NO logo or colors are provided: ask for them. Do not invent the Elm logo or guess brand colors; use a neutral placeholder and say so.

### 6a. Fonts (fixed, NEVER ask the user)
Do not ask about fonts at any point. Always apply this order automatically:
1. **Elm font (default).** Look for it, in order: an `@font-face` embedded in any attached/source HTML (family names containing "Elm"), font files attached by the user (.ttf/.otf/.woff/.woff2 with "Elm" in the name), then a previous film in the conversation. If found, embed it as base64 and use it.
2. **IBM Plex (fallback).** If no Elm font is found, silently use **IBM Plex Sans Arabic** for Arabic and **IBM Plex Sans** for English/Latin. Get them from npm (`npm pack @fontsource/ibm-plex-sans-arabic @fontsource/ibm-plex-sans`, weights 300/400/500/700, woff2) and embed as base64 so the file stays offline and self-contained.
- CSS stack always: `font-family: "Elm", "IBM Plex Sans Arabic", "IBM Plex Sans", system-ui, sans-serif;`
- Mention which font was used in one line of the delivery note only (e.g. "Font: IBM Plex fallback"), never as a question.

## 7. Voice: ElevenLabs v4 script and voice picks

### 7a. Suggest a voice per speaker/scene
- Before writing the script, look up current ElevenLabs voices (web search the ElevenLabs Voice Library for the chosen language/accent, e.g. "Arabic Saudi male narrator").
- For each speaker give: voice name (+ voice ID if found), why it fits the persona (age, warmth, energy), and suggested settings: model = Eleven v4, Stability, Similarity, Style, Speed.
- Personas must sound clearly different from each other (mix gender, pitch, energy) but stay in one consistent quality level.
- If a voice could not be verified from a live source, say so and describe what to search for instead of inventing a name.

### 7b. Audio tags
- Eleven v4 reads natural-language direction in square brackets, e.g. `[warm]`, `[confident]`, `[excited]`, `[calm]`, `[whispers]`, `[pause]`, `[slowly]`, `[smiling]`. Before writing, check the official ElevenLabs Audio Tags list (elevenlabs.io/blog/elevenlabs-audio-tags-list) for current tags.
- Use tags sparingly: one at the start of a line or on a turn in emotion, not on every sentence. Write tags in English even when the text is Arabic.
- No sound-effect tags unless the user asks (music/SFX are handled in the HTML).

### 7c. Deliver ONE complete script file (narration.md)
All scenes, all speakers, in order, in a single file, ready to paste. Structure:

```
# Voice cast
| Speaker | Voice | Voice ID | Stability | Similarity | Style | Speed |

# Script
## s01 — <scene name> — <Speaker> — file: s01_<speaker>.mp3
[calm] النص هنا...

## s02 — ...
```
- Arabic: chosen variety (MSA or Saudi dialect), warm professional tone, short sentences; write numbers as words where pronunciation matters ("أربعة وعشرون نموذجًا"); add tashkeel only on words likely to be misread.
- English: conversational professional tone; spell out acronyms meant to be read letter by letter.
- Product terms: keep in original language only if they will be said that way; otherwise write how they should sound.
- End the script message with a clear handoff, in the user's language: "ولّد الصوت لكل مقطع باسم الملف المكتوب، وارفعها لي هنا، وأنا أدمجها وأرتبها مع الملف."
- Meanwhile, the film is built and works silently on estimated timings.

### 7d. When the user returns the audio
- Match files to scenes by name; measure each duration (ffprobe or Web Audio decode).
- Normalize loudness so all speakers match (~-16 LUFS; no speaker noticeably quieter). Use ffmpeg if available.
- Re-flow the timeline to real durations; re-time caption lines proportionally to word count within each clip (or use ElevenLabs timestamps if the user provides them).
- Embed audio as base64 and re-run QA.

## 8. Background music (only if the user said yes)
- "Add it for me": generate an original ambient bed with Web Audio inside the HTML (soft pads/chords + subtle pulse, key and mood matching the film: calm for problem, rising for solutions, uplifting for closing). No downloaded or copyrighted tracks.
- "I will provide a track": embed the user's file.
- Either way: music sits well under the voice (~-20 to -24 dB relative), auto-ducks a further ~6-8 dB while narration plays, fades in at the start (~2s) and out at the end (~3s), stays in sync with seek/pause, and has a mute toggle (M key + button).

## 9. QA before every delivery (Playwright)
- Load the file, use `VIDEO.seek(t)`, wait ~600ms, screenshot the middle of every scene plus transitions.
- Build a contact sheet (PIL grid of resized JPEGs) and review it in one Read.
- Check: font is Elm or IBM Plex (no system-font fallback visible); Play button bottom-left and Fullscreen bottom-right; CC button and C key hide/show captions; progress fills left->right; content direction correct; no text overflow/clipping; no overlap with logo, captions or controls; captions match narration; emblem not scaling with voice; music ducks under voice; runtime matches the chosen duration; keyboard controls work.
- Fix issues, then re-screenshot.

## 10. Delivery and iteration
- Phase 1 delivery: the HTML film (silent, estimated timings) + narration.md (voice cast + full tagged script) + scene list with timestamps.
- Phase 2 delivery (after audio arrives): final HTML with embedded narration, re-timed captions, music, final runtime.
- Iterate scene by scene. When the user gives notes, change ONLY what was asked and preserve all other scenes, audio, captions and controls.
- When merging a new design file from someone else into the film, keep the existing engine, audio and content notes, and port the new visuals into scenes.
