---
trigger: always_on
description: This file tells you, the coding agent, how to make a video with explainroo.
---

# explainroo for coding agents

This file tells you, the coding agent, how to make a video with explainroo.
The user usually gives you a topic, and maybe a length or a style. You write
the words and the pictures. explainroo makes the video.

explainroo turns two files into an MP4 video with narration. `script.md`
holds what the voice says. `scenes.js` draws what the viewer sees. Every
drawing can appear on a word the voice says. A voice model (Kokoro) reads the
script on this computer. Whisper listens to the recording and writes down
when each word is spoken. Chrome draws the frames in the background.
explainroo adds music and sound effects, and ffmpeg makes the MP4. You need
no API key unless the user wants AI images.

The full documentation for people is at https://www.explainroo.com/docs/.

## Before you start

Maybe the user pasted the prompt from the README or from explainroo.com, and
you are not in an explainroo folder yet. Then clone it into the current
folder first. If the user named another place, use that.

```bash
git clone https://github.com/vincentsch/explainroo.git
cd explainroo
```

Run this once in the explainroo folder:

```bash
npm install
node bin/explainroo.js doctor --fetch
```

`doctor` checks Node, ffmpeg and Chrome. It also downloads the speech models,
about 400 MB, the first time only. If the user ran `npm link`, you can write
`explainroo` instead of `node bin/explainroo.js` in the commands below. Every
command takes the project folder first and accepts `--json`.

### When to ask the user

Decide the normal things yourself, like the look, the length, the voice, the
layout and the wording. Ask only when you cannot go on without an answer.
When you ask, put everything you need in one message. Typical reasons to ask:

- The topic is unclear, or you need a fact you cannot check.
- The user wants AI images and there is no OpenRouter key (see Images).
- The user wants something explainroo cannot do, like video shot with a
  camera.

## Making a video, step by step

1. **Learn the subject.** Read the code, docs or pages the user points to.
   Write down the facts you will use. Never make up numbers, quotes or claims.
   If a number matters and you cannot check it, leave it out.

2. **Create the project.**

   ```bash
   node bin/explainroo.js init videos/<name> --theme paper --title "..."
   ```

   Projects go in `videos/`. Git ignores that folder. Pick the look that fits
   the audience: `paper` (friendly, hand drawn, the default), `clean`
   (products and business), `chalk` (teaching), `blueprint` (engineering) or
   `midnight` (developer tools). Pick the size for the place the video goes,
   for example `--size tiktok` or `--size linkedin`. The sizes are listed in
   "Sizes for each platform" below. Add `--pace 1.2` when the user wants a
   quicker video.

3. **Write `script.md`.** The narration comes first. It sets the timing for
   everything else. See "How to write the narration" below.

4. **Make the voice.**

   ```bash
   node bin/explainroo.js voice videos/<name>
   ```

   The command prints how long each scene is and how many words the speech
   check confirmed. If a word is not confirmed, the voice probably said it
   wrong. Fix it with `{shown|spoken}` in the script and run the command
   again. Only the scenes you changed are made again.

5. **Plan the pictures.** For each scene, decide what is on screen when each
   marker or important word is spoken. Decide where each thing sits and what
   leaves the screen. Keep things in the same place from scene to scene.

6. **Make images, if the video needs them.** Icons and diagrams cover most
   technical topics. Everyday how-to topics like cooking, cars or gardening
   work better with pictures. See "Images" below.

7. **Write `scenes.js`.** Write one function per scene. Tie every picture to
   the narration with `at: 'word'` or `at: '#marker'`. The scene API is at
   the end of this file.

8. **Check.**

   ```bash
   node bin/explainroo.js check videos/<name>
   ```

   Fix every error and every warning. You decide what to do with the hints.

9. **Look at the frames.** You cannot watch the video, so this is how you see
   it.

   ```bash
   node bin/explainroo.js still videos/<name>                 # the end of each scene
   node bin/explainroo.js still videos/<name> intro@2.5 12.0  # any moment
   node bin/explainroo.js sheet videos/<name> --scene intro   # one scene over time
   node bin/explainroo.js sheet videos/<name>                 # the whole video
   ```

   Open the images and judge them like a viewer would. Is the text big
   enough? Is anything cut off or on top of something else? Is half the frame
   empty? Does the sheet show something new every few seconds? Fix what is
   wrong and look again.

10. **Make the video and check the file.**

    ```bash
    node bin/explainroo.js render videos/<name> --draft   # quick half-size version
    node bin/explainroo.js render videos/<name>           # final video
    node bin/explainroo.js verify videos/<name>
    ```

    `render` makes the video. `verify` measures the loudness and looks for
    black frames and silence. It also runs the speech check on the finished
    sound to make sure the voice is still clear over the music.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentsch/explainroo](https://github.com/vincentsch/explainroo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
