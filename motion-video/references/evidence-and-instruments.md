# Evidence tiers and instruments

The rules come from the author's private lab (a case library of source films, technique
cards and isolated imitation rounds). Provenance labels in the text: `Cnn` = a source
film, `Tnn` = a technique card, `rNN` = an imitation round (arms named by letter, e.g.
`r10 A1`). The lab's records are not shipped; its general instruments are, in `scripts/`.

Run them from the skill folder with `python -X utf8 scripts/<name>.py`. Requirements:
Python 3.10+, `ffmpeg` / `ffprobe` on PATH, and the packages in `requirements.txt`.
Every instrument except `make_beat_track.py` has `--selftest` (synthetic known-true and
known-false inputs); run it before trusting a number, and again after any edit. Each
script's docstring holds its flags, thresholds, calibration and limits; the calibration
notes cite lab cases (`Cnn`) that are not shipped.

## Tiers (how sure each rule is)

| Tier | Meaning | Where it comes from |
|---|---|---|
| author | the skill's author judged it after watching films | round reviews |
| measured | a calibrated instrument read it on source films | the technique cards' measurements |
| verified | a blinded A/B round moved it, ranges disjoint (usually n=3 per arm) | round reports |
| unvalidated | measured on sources, never tested by imitation | cards T08, T09, T10 (2026-09-30) |

A rule rises a tier only through the next row's evidence. "Looks good" is never a tier:
it is the viewer's own watching. An author-tier rule records one person's taste; the
current user's ruling overrides it.

## Which instrument answers which question

| Question | Instrument (`scripts/`) | Main limit |
|---|---|---|
| Real BPM, beats, cuts | `beatgrid.py` | bar downbeat unknown (grid x4 only) |
| Are events phase-locked to the beat | `phaselock.py` | small n is weak (random R ~ 1/sqrt(n)) |
| Section boundaries | `segments.py` (`--crop` for headline-only changes) | cannot tell a section change from an in-section push-in or a dialog |
| Section changes vs narration lines | `narrlock.py` (needs an `.srt` from any ASR tool) | dense lines weaken the test; machine transcript timing ±0.1-0.2 s |
| Moving share, still runs, frame fill | `density.py` | pixels changing, not meaning; noise counts as motion |
| Product-intro checks (bar lock, end hold) | `measure_product.py` | boundary = midpoint of the change run |
| Narrated explainer bundle | `measure_narrated.py` (`--srt <file>`, or set `MV_ASR_EXE` to an ASR command) | as its parts |
| Data showreel bundle | `measure_showreel.py` | as its parts |
| On-screen text and numbers vs the product | `fidelity.py` (`--viewport WxH` for vertical); needs a corpus spec pinned to the product commit the film saw — see the example below | review list, not a verdict |
| Camera moves (zoom/pan/rotate per section) | `camflow.py` | screen space only; transitions read as moves (`at_boundary` = look) |
| Shape-to-shape at a transition | `shapematch.py` | mask overlap, not identity; blind to a one-frame ground swap |
| Ground colour switches | `bgswitch.py` | full-bleed illustration unmeasurable |
| Spring fits | `springfit.py` | only `spring_consistent: true` fits are cited |
| Contact sheets | `sheets.py` | — |
| A known-beat test track (+ beat-time JSON) | `make_beat_track.py --bpm 120 --bars 24 --out <wav>` | synthetic clicks, not music |

Not shipped: the lab's sound-effect lock instrument was rejected (it missed its real
positive control); no rule here cites it.

## `fidelity.py` corpus spec (example)

`root` is a snapshot of the product's source at the commit the film shows (for example a
`git archive` of the README and UI sources, unpacked). Demo-data files go in `globs` only:
their numbers are listed for a reader, never credited as product facts.

```json
{
  "root": "path/to/product-snapshot",
  "root_note": "Pinned to product commit <sha>. Regenerate with git archive <sha> README.md src/ .",
  "globs": ["README.md", "src/**/*.tsx", "src/**/*.ts", "demo_server.py"],
  "number_globs": ["README.md", "src/**/*.tsx", "src/**/*.ts"],
  "optional_features": [
    {"name": "captions / ASR", "patterns": ["字幕", "語音辨識", "ASR"]}
  ]
}
```

## Reuse before rebuilding

Keep your past production prompts and the test videos they made; reuse a prompt before
writing a new one. Studying a NEW stream of reference videos (collecting, tagging,
measuring cases) is a separate job from making a film.
