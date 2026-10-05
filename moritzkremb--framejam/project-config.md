---
trigger: always_on
description: Make and review videos and storyboards with the user in the loop, with any video tool (Hyperframes, Remotion, Motion Canvas, ffmpeg, screen recordings). Use when the user wants to make or edit a video, motion graphic or animation, wants a storyboard or shot list reviewed before animating, wants to pick a visual style, or wants to review a render and send timestamped feedback. Requires the framejam MCP server.
---


# FrameJam: style presets + one-click video and storyboard review

FrameJam runs next to your chat at http://localhost:2400. It gives you two things the chat is bad at:
**choosing a style** (a structured `style.json`, plus a working Hyperframes template) and **precise feedback**
(timestamped, pinned comments with a frame image of the render).

FrameJam works with whatever makes the video. **The user reviews the rendered video**, so what they see is exactly
what you rendered. Hyperframes is the default way to build, but you always review the render (`videoPath` only).
A live player that lets the user click a DOM element is a **beta** feature: use it only if the user asks for it (see
"Live player (beta)" below).

**Words to use with the user:** FrameJam calls each review a **project** (the start page is "Projects"). Say
"I opened the project in FrameJam", not "I opened a review". Tool and field names (`open_review`, `reviewId`) stay as
they are.

## 0. Open FrameJam in the built-in browser (always do this first)

As soon as this skill is used, open FrameJam where the user can see it, without being asked:

- **Cursor:** use the built-in browser tool (`cursor-ide-browser` → `browser_navigate`) with
  `position: "side"` so it opens beside the chat.
- **Other harnesses:** use their browser/preview tool if there is one. Otherwise give the user the link.
- **Which page:** the project's `url` when step 1 finds one, `http://localhost:2400/styles` for a new video, and
  `http://localhost:2400` (Projects) until you know.
- If the page doesn't load, nothing is hosting the UI. Run `npx -y framejam start` (it starts the UI in the background,
  or reuses a running one, and prints the URL), then open that URL. Never ask the user to start it.
- If the framejam tools themselves aren't available, run `npx -y framejam install <app>` with the app you're running
  in (`cursor`, `claude` or `codex`). It adds the MCP server and the skill (or confirms them) and starts the UI. Open
  the link it prints, then tell the user the last step it prints (restarting the app once so the tools load).

## 1. Work out what the user is doing (before asking anything)

The user may type nothing but `/framejam`. Check these in order and act on the first that applies. Don't ask "what do
you want to make?" if any of them answers it.

1. **What they typed.** "/framejam make a 10s teaser for X" → do that. A style check is only needed for a new video.
2. **This conversation.** If you've been making or editing a video in this chat, keep going with it: render the current
   state to the next `renders/vN.mp4`, then add it to the same project (`add_version` with its `reviewId`, or
   `open_review` with the same `title`), and wait for feedback.
3. **Feedback waiting.** Call `list_reviews()`. If a project is `sent_not_delivered` or `user_commenting`, call
   `get_feedback({ reviewId })` and apply it.
4. **A FrameJam project for this folder:** one whose `compositionDir`, `videoPath` or `panelsDir` is inside the
   current workspace.
   - `awaiting_user`: open its `url` and call `wait_for_feedback`.
   - `delivered_to_agent`: an earlier chat got the comments but never shipped the next version. Call `get_feedback` to
     get them again, apply them, and ship the next version.
   - A storyboard the user approved, and no video yet: build the video from the panels.
5. **A video project here, but no FrameJam project.** A Hyperframes `index.html` with `data-composition-id`, a Remotion
   or Motion Canvas project, a `renders/` folder or a loose video file: use the newest render if it's newer than the
   source, otherwise render. Open it as v1 and wait for feedback. Use the project's own tool; for a plain video file,
   pass `videoPath` only.
6. **Nothing relevant** (empty folder, unrelated code repo): open the Styles page and ask what video to make: what it's
   about, roughly how long, the format (16:9, 9:16, 1:1), and any copy, footage or brand assets. Mention they can pick a
   look on the Styles page. Don't start building from a style alone. New videos default to Hyperframes.

If several projects could match, ask one short question that lists them.

## The loop

1. **Style.** Call `get_selected_preset`. If it returns `selected: null`, ask the user to pick one on the Styles page you
   just opened (then call it again), or pick one yourself with `list_presets({ mood, pacing, format })` and `get_preset(id)`.
   - Follow `style.guide` literally, whatever the tool. Use the exact `palette` hex values, `fonts`, `easing` names,
     `transitions`, `textAnimations` and `rhythm.averageShotSeconds`.
   - `templateFiles` is a working Hyperframes composition. In a Hyperframes project, copy it in and replace the copy
     rather than starting from a blank file. With any other tool, don't copy it; read it as a reference for the look.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moritzkremb/framejam](https://github.com/moritzkremb/framejam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
