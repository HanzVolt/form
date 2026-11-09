# FORM: a vocal synthesizer for the M-VAVE FM-1

**FORM** is free, open-source firmware that turns the M-VAVE FM-1 pocket synth into a **singing voice synth**. It sings vowels and consonants, spells out words you type, writes alien melodies with a Markov chain, and puts a clean mono sub under it all. A little cyborg face sings along on the screen.

> ⚠️ **Test build.** FORM 1.3 "MARKOV v2 + beginner" is a test build, **not a release**, and it **has not been tested on a real FM-1 yet**. Flash it only to a spare FM-1, at your own risk. Read [Before you flash](#before-you-flash) first.

---

## Contents

1. [Downloads](#downloads)
2. [What FORM is](#what-form-is)
3. [The voice engine](#the-voice-engine)
4. [Two voice tracks, and the vowel / consonant split](#two-voice-tracks-and-the-vowel--consonant-split)
5. [The 64-step vowel sequencer](#the-64-step-vowel-sequencer)
6. [Words: typed words, built-in words, ALIEN, SENTENCE](#words)
7. [MARKOV: melodies that write themselves](#markov-melodies-that-write-themselves)
8. [Shaping the vowels](#shaping-the-vowels)
9. [Shaping the consonants](#shaping-the-consonants)
10. [The SUB](#the-sub)
11. [Effects, EQ and the rest](#effects-eq-and-the-rest)
12. [BEGINNER / PRO and SMART](#beginner--pro-and-smart)
13. [Flashing](#flashing)
14. [Before you flash](#before-you-flash)
15. [Credits and licence](#credits-and-licence)
16. [Contact](#contact)

---

## Downloads

| File | What it is |
|---|---|
| [`FORM-1.3-MARKOV-public.fwsc`](FORM-1.3-MARKOV-public.fwsc) | The firmware package. 610,035 bytes. |
| [`FORM-1.3-MARKOV-public-Flash.html`](FORM-1.3-MARKOV-public-Flash.html) | A one-page flash helper with the same firmware built in. No server and no internet needed. |

**SHA-256 of `FORM-1.3-MARKOV-public.fwsc`:**

```
5310ea91004bef2547e1cf5ed78972bab4d7a98d84ed4730f383a9788ad1a6fb
```

**About the flash page:** GitHub shows HTML files as code, so clicking it here won't run it. Open the file, press **Download raw file** (or **Raw**, then save the page), and open the saved file in **desktop Chrome or Edge**. It flashes over **Web MIDI with SysEx**, so other browsers won't work.

The visualizer is a singing female cyborg face in two sizes (styles 17 and 18). It starts on style 18.

---

## What FORM is

- A **voice synth**: instead of a plain oscillator, you hear a mouth shaping a buzz into vowels.
- **9 vowels**: A, E, I, O, U, EE, OO, SCHWA (the soft "uh") and M.
- **23 consonants**: S, SH, F, H, V, Z, T, K, P, D, G, ZH, TH, B, M, L, R, W, Y, N, NG, CH, J.
- Hold a key and FORM **sings a word at that pitch**. The built-in demo word is `THIS IS HANS VOLT`.
- FORM is built on SLOOP / Felucca, the open FM-1 firmware family, and keeps their sequencer, effects, MIDI and USB features where they make sense for a voice synth.

The FORM sounds are **AH CHOIR**, **MURMUR PAD** and **EE LEAD**.

---

## The voice engine

FORM's voice is a **formant-FM engine** built on the firmware's 6-operator FM core (in the spirit of Yamaha's FS1R voiced and unvoiced operators). Each voice uses six operators:

| Operator | Job |
|---|---|
| OP1 to OP3 | The three **formants** (the mouth's resonances): each is a pair of harmonics of the note, phase-modulated, and crossfaded so the formant can sit between harmonics |
| OP4 | The **carrier**: the note itself, the shared modulator, and an audible "body" |
| OP5 | The **driver**: a slow sine or a chaotic (logistic map) wobble that moves the formants |
| OP6 | The **unvoiced** part: filtered noise for breath and frication |

**Consonants are synthesized too.** On a consonant track each consonant is band-passed noise plus a sine "voice bar". L, R, W, Y, N and NG also bend the vowel track's formants toward the consonant. On a plain vowel track, a step can add a noise burst, a filter snap, or both.

**No samples.** The voice, the consonants and the sub are all calculated in real time. The firmware ships no sample data. The only thing that can play audio files is TEXTURE's four user slots (USR1 to USR4), if you load your own sounds into them.

**About wavetables:** the voice is not wavetable synthesis. The EDIT > `WAVE` page can swap the carrier's sine for SAW, SQUARE, TRI, SFTSAW, PLS25, VOWEL or METAL. Those shapes are short single-cycle tables that the firmware calculates from harmonics at power-on. Nothing is stored in the image. The sub's SAW and SQR are band-limited (polyBLEP) oscillators.

---

## Two voice tracks, and the vowel / consonant split

- FORM has **two voice tracks**, 1 and 2. Each has its own FORM sound and its own 64-step sequencer, so you can play two different voices side by side. The TRACKS screen shows tracks 1 and 2. There is no drum track.
- **The split:** set track 2's `MODE` to `CONS` and it becomes a **consonant track** for track 1. Track 1 sings the vowels, track 2 plays the consonants in time with them, and together they form words. Type a word on track 1 and **SEND** writes its consonants onto track 2 for you.
- **SEQS ONE / TWO** (on a split):
  - `ONE`: the consonant track is locked to the vowel track, step for step.
  - `TWO`: two separate sequencers, each with its own length, rate and step timing. Each track's clock can follow the song (`SONG`) or run at its own BPM (`OWN`, 40 to 240).

---

## The 64-step vowel sequencer

Tap **SEQ** on track 1 or 2. Each of the 64 steps holds its own:

| Page | K1 | K2 | K3 | K4 |
|---|---|---|---|---|
| `VOWELS` | STEP (1 to 64) | VOWEL (A, E, I, O, U, EE, OO, SCHWA, M) | TIME (SNAP, 2 to 16) | GLIDE (OFF / ON: slide into the next step) |
| `VOWELS 2` | PITCH: the step's own note, relative to the key you hold (or HOLD) | CONS: NONE, NOISE, SNAP, N+S | FILT 0 to 7 (darker) | LEVEL: 0 dB down to -24 dB, or OFF |
| `VOWELS 3` | BRTH 0 to 7 | STRES / UNSTR | HARD OFF / ON | RDUCE (vowel reduction) |
| `VOWELS 4` | ON | LEN 1 to 64 | RATE 1/64 to 1/4 | MIX |
| `VOWELS 5` | RSTRT | COPY row | PASTE row | MUTAT (mutate a row) |
| `VOWELS 6` | WORD (built-in words) | LOAD | COART (how much neighbouring sounds blend) | TRANS (transition time) |
| `VOWELS 7` | VBRTH (breath on all vowels) | PHRAS: SHARED / PERKEY | SHIFT (the step's own formant shift) | |

- **GLIDE** slides the vowel, pitch, filter and level smoothly into the next step.
- **RSTRT ON** (the default): every new key press sings the word again from step 1. That's **word mode**: press a key, hear the word.
- **PERKEY**: each held key sings its own copy of the word (up to 3), so you can play a word as a chord.
- In PRO, the white keys pick steps on these pages, the first four black keys pick a row of 16, and the top black key (F#5) auditions the word. Hold SEQ and tap HOME to make the keys play notes instead.
- Risky actions (COPY, PASTE, MUTAT, LOAD) ask first. Turn once and the screen says `AGAIN: …`, then turn again to confirm.

---

## Words

**WORD TYPE**: spell a word letter by letter (A to Z and space, up to 64 steps). Hold a key while you add a letter to give it that pitch, then **SEND**. FORM turns the letters into vowel and consonant steps. On SEND it adds natural breath to H, S, V, M and N.

**Built-in words:** 15 ready-made words and phrases on `VOWELS 6`, from `THIS IS HANS VOLT` to `SYNTH SAMURAI` and `AUTOBOTS ROLL OUT`.

**ALIEN and GO** (on the `MODE` and `MARKOV` pages, or WORD TYPE's DO menu): one turn writes a new made-up phrase into the sequencer.

- `8`, `16`, `24`, `32` or `64`: a random alien phrase that many steps long.
- `WORD`: a phrase from the built-in alien dictionary (50 phrases).
- `SONG`: the next built-in **song line**, a short sung melody with its own suggested BPM (13 lines in this build).
- Turn GO left to **undo** the last roll.

**SENTENCE**: chain up to **32 slots** of built-in words, alien phrases or your own typed words. `TAP` plays the next word on each key press. `AUTO` steps through the words on a rhythmic grid while you hold a note. `LOOP` repeats the sentence.

---

## MARKOV: melodies that write themselves

The `MARKOV` page (SEQ, on a vowel track) sets the melody that ALIEN writes, and lets you change it **while it plays**.

| Knob | What it does |
|---|---|
| K1 **MELODY** | `WALK`: a random walk. `1` to `8`: a **first-order Markov chain** learned from FORM's transcribed song lines. Higher numbers wander from the table more and add more glides and breaths |
| K2 **SCALE** | `SEL` (follow the SEL page), CHR, MAJ, MIN, DOR, MIX, PEN, MPEN, HARM, PHRY, LYD, LOC, MEL, BLUES, WHOLE, DIMHW, DIMWH, plus four tunings off equal temperament: **PYTH** and **JUST** (major, Pythagorean and just intonation) and **PYMIN** and **JUMIN** (their minor versions) |
| K3 **STEPS** | How long the next roll is: 8, 16, 24, 32 or 64 steps |
| K4 **GO** | Turn right to roll a new line, left to undo |

- Turning MELODY or SCALE **re-shapes the playing line straight away**. Only the pitches, glides and breaths change; the vowels, consonants and timing stay. Turn back and you get exactly the same line again.
- Song lines keep their own melody (`SONG: OWN MELODY`).
- **CHAOS layer** (PRO only): tap **REC** on the MARKOV page and K3 / K4 become **CHAOS MELODY** (0 to 8: notes jump off the line, kept in the scale) and **CHAOS VOICE** (0 to 8: vowel swaps, stutters, timing jitter, pitch bends). Both at 0 means off. The CHAOS settings are saved with the project.

---

## Shaping the vowels

EDIT > `EDIT 1` (shown as `VOWEL`) and `EDIT 2` (shown as `DRIVE`):

| Knob | What it does |
|---|---|
| **VOWX** | Moves the vowel from front (i, e) to back (u, o) |
| **VOWY** | Moves the vowel from closed (i, u) to open (a) |
| **SHFT** | Shifts all formants up or down by up to an octave, independent of the pitch (bigger or smaller "mouth") |
| **SKRT** | The formants' width ("skirt"): narrow and pure, or wide and buzzy |
| **RATE** | The driver's speed |
| **DEPT** | How far the driver moves the formants |
| **CHAO** | Blends the driver from a smooth sine into chaos |
| **FRIC** | The breath and frication noise level |

EDIT > `WAVE`: **WAVE** (the carrier shape), **BODY** (the carrier's own level) and **BUZZ** (its self-modulation). The track's FLT and SHP modulation (from ENV DEST and LFO DEST) move SHFT and SKRT.

**Breath on vowels:** `VBRTH` (VOWELS 7) sets one breath level for every sung vowel, from OFF and -36 dB to 0 dB (default -24 dB). A step's own `BRTH` overrides it. This soft breath under the vowels is the "HANS" sound of `THIS IS HANS VOLT`.

---

## Shaping the consonants

On a consonant track (`MODE` = `CONS`):

| Page | K1 | K2 | K3 | K4 |
|---|---|---|---|---|
| `CONSONANT` | CONS (`--` or one of the 23) | LEN (AUTO, or 5 to 315 ms) | EDGE 0 to 7 | STEP |
| `CONSONANT 2` (SEQS TWO only) | TIME | ANCHR (the track's anchor note) | | STEP |
| `CONSONANT 3` | LEVEL (0 dB to -24 dB, or OFF) | BRTH 0 to 7 (a puff of air at the attack) | GLIDE: OFF, AUTO, 10 ms to 300 ms (morph into the next consonant) | HARD OFF / ON (harder, brighter) |
| `CONSONANT 4` | SHIFT (the step's frication band) | | | STEP |

When track 1 is split with track 2, track 1 also gets a `CONSONANT 3` page, so you can set the consonants from the vowel track.

---

## The SUB

EDIT > `SUB` adds **one clean, low note under the voice**.

| Page | K1 | K2 | K3 | K4 |
|---|---|---|---|---|
| `SUB` | **LEVL** (0 = off) | **INTV**: 5TH, -1, -2, -3 (octaves), B0, OWN | **MODE**: ROOT / LOW | **LFO** (OFF, 1 to 15: a level wobble) |
| `SUB 2` | **SWELL** OFF / ON | **COUPL** OFF / ON | **DEPTH** 0 to 15 | **LRAT** (the LFO rate, 0.1 to 20 Hz) |
| `SUB ENV` | ATK | DEC | SUS | **REL** (each `TRK` = follow the track, or its own time: release 40 ms to 10 s) |
| `SUB 3` | SRC: TRACK / MIDI | CH (MIDI channel) | LINK | WAVE: SINE / SAW / SQR |

- **Monophonic:** one sub per track, however many notes you hold.
- **Where it sits:** in `ROOT` mode it goes under the root of the chord you hold. In `LOW` mode it goes under your lowest note. It is never above the note it follows.
- **ROOT triad detection:** FORM looks for a third and a fifth among the held notes, so inversions keep the right root. A single note is its own root. With no triad, the last root stays.
- **INTV:** a fifth or 1 to 3 octaves below. The interval is raised by an octave rather than drop under 25 Hz. **B0** goes 3 octaves down but holds at B0 (30.87 Hz) instead of jumping when the note goes low.
- **SWELL** ON re-triggers the sub's envelope on each new root. OFF keeps a steady level across chord changes.
- **COUPL / DEPTH:** open vowels (VOWY up) make the sub louder.
- **Clean path:** the sub skips the 10-band EQ and every track effect (distortion, crusher, INSERT, slicer, track filter, delay and reverb sends). It still follows the track's level and mute, and it still goes through the master DJ filter and the limiter.
- `SRC` = `MIDI` lets the sub play from its own MIDI channel.

---

## Effects, EQ and the rest

- **Per track:** DIST, a **bit crusher** (AMT, BITS, RATE, FOLD; SRMD and ADPT on `CRUSH 2`), **INSERT** (SOFT, HARD, FOLD, FUZZ, CRUSH, PHASR, FLANG), **SLICER**, and the **track filter** (a DJ-style low-pass / high-pass).
- **Shared:** tempo delay and reverb.
- **10-band EQ** (FX > `EQ`): one global EQ at the end of the voice tracks, 31 Hz to 16 kHz, ±12 dB per band, with shelves at both ends. On the EQ page the black keys pick the band.
- **Master:** the **DJ filter** (GLO > `MASTER` FILT) and a **look-ahead peak limiter**.
- **TEXTURE** (GLO > `TEXTURE`): a looping bed under the mix: VINYL, TAPE, RAIN, WIND, RADIO, ROOM (all generated) or your own sample in USR1 to USR4.
- **Removed for FORM:** DUST, the chorus (the send and the INSERT type) and DUCK.
- **Mouth visualizer:** tap HOME for a cyborg face whose mouth sings the vowels, with an eye that flashes on each word. A small face also sings in the sequencer window. Styles: 17 (CYBORG F SMALL) and 18 (CYBORG F).
- **Saving:** autosave, user preset slots, four project sections A to D, and an overwrite warning before a save would replace your vowel steps (you can turn the warning off in the HOME menu).
- **Also kept:** the arpeggiator with KBD HOLD, ENV, two LFOs, chords and scales on the SEL pages, MIDI in and out, MIDI clock, and USB audio.

---

## BEGINNER / PRO and SMART

- **BEGINNER** (GLO > `MASTER`, K2 `LEVEL`) locks the few settings that can make FORM go quiet or act strangely. The keys always play notes, SENTENCE is off, MELODY stays `WALK`, STEPS offers only `16` or `WORD`, GO rolls on the first turn, CHAOS is hidden, and layers can't be locked. A small `B` shows in the top bar.
- A new FM-1 **starts in BEGINNER**. Turn K2 one notch right for **PRO** (everything unlocked). The level is saved with your project, and BEGINNER never erases anything you set.
- **SMART** (GLO > `MASTER`, K1) is **off at every power-on**, so you see the full raw parameter view. Turn it on to hide parameters that can't change the sound right now. It's a view switch only and isn't saved.

---

## Flashing

The flash page does everything in the browser:

1. Use **desktop Chrome or Edge** (the page's own steps are written for a Mac). Open the downloaded `FORM-1.3-MARKOV-public-Flash.html`.
2. Connect the FM-1 with a USB **data** cable, **directly** (no hub). Close your DAW, M-UPGRADE and every other app or tab that uses MIDI.
3. Press the big button and allow **MIDI SysEx** access. It's Web MIDI, not Web Serial.
4. Pick your FM-1 in the list, then press the button again to flash.
5. The page checks the built-in package (SHA-256, size and identity) before writing anything. It then switches the FM-1 into its **Felucca Update** loader, writes the firmware, and waits for the FM-1 to restart and report its identity. Keep the cable in and the page open until it says it's done.

**If it gets interrupted:** the FM-1 stays in update mode. Run the page again and it resumes. **USB rescue:** hold **OCT−** alone while switching the FM-1 on. (OCT− and OCT+ together is calibration, not rescue.)

After flashing, HOME (hold) > SYSTEM > ABOUT shows `FORM 1.3`.

---

## Before you flash

- **This is a test build.** MARKOV v2 + beginner hasn't been tested on a real FM-1 yet, and the flash helper isn't hardware-tested either. **Use a spare FM-1 and flash at your own risk.** There is no warranty.
- **Back up first.** Save your projects and presets with your current firmware's tools before flashing.
- **Know your way back to stock:** download the official FM-1 firmware (V15, `FM-1.fwsc`) and M-VAVE's updater, **M-UPGRADE**, from m-vave.com. Close every other MIDI app before using it. The firmware still answers M-UPGRADE's update handshake. This return path hasn't been tested with this build.
- **If the FM-1 won't start at all,** recovery needs extra hardware: [FM-1-transporter](https://github.com/kurogedelic/FM-1-transporter) by kurogedelic.

---

## Credits and licence

- **FORM** by Hans (HanzVolt).
- Built on **SLOOP** ([sloop-fm1](https://github.com/isod89/sloop-fm1)), the FM-1 groovebox firmware, which is based on **[Felucca](https://github.com/hugelton/Felucca)** by **Leo Kuroshita (@kurogedelic), Hügelton Instruments**. The SLOOP and Felucca credits stay on the device: SYSTEM > ABOUT reads "BASED ON FELUCCA".
- The flash page's package and update libraries are © Leo Kuroshita (@kurogedelic), Hügelton Instruments, GPL-3.0-only.
- FM core: **msfa** by Google Inc. and Pascal Gauthier, from Dexed (Apache-2.0).
- Screen font: **Terminus** (SIL OFL 1.1).
- **Female cyborg face** (visualizer styles 17 and 18): "Female cyborg" (File:Female_cyborg.png), made with Bing Image Creator, uploaded by Perquirius to Wikimedia Commons, licensed **[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)**. FORM cropped it, scaled it to 240 × 240, reduced it to 64 colours and tinted the android eye cyan. The adapted picture is CC BY-SA 4.0 as well. No endorsement by the creator or by Wikimedia is implied. On the device, ABOUT shows `FACE: BY-SA`.
- **The HV boot logo** is © Hans (HanzVolt), all rights reserved. It is **not** covered by the GPL.
- Recovery tool mentioned above: FM-1-transporter by kurogedelic (separate project).

**Licence:** the FORM firmware code is **GPL-3.0-only** (see [LICENSE](LICENSE)). The exceptions are the HV logo and the CC BY-SA 4.0 face picture described above.

**Source code:** the complete corresponding source for this build will be published here. Until then, it's available on request (see [Contact](#contact)).

"M-VAVE" and "FM-1" are trademarks of their owners. FORM is independent firmware. It isn't affiliated with, endorsed by or supported by M-VAVE. Like SLOOP and Felucca, it never turns on the FM-1's Bluetooth or Wi-Fi radio.

---

## Contact

- Website: **[hanzvolt.com](https://hanzvolt.com)**
- YouTube: **[@hanzvolt](https://www.youtube.com/@hanzvolt)**, for demos and walkthroughs
- Found a problem with the test build? Open an issue on this repo.
