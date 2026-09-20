---
trigger: always_on
description: generates a frame on damage and the grab waits for it, DRI3 and XShm because the X server's
---

# Working on this repository

pixelflux is the screen-capture and video-encode library behind
[selkies](https://github.com/selkies-project/selkies), and is developed together with it and with
[pcmflux](https://github.com/selkies-project/pcmflux) (audio capture and encode). A change in one often belongs in
another; coordinate across all three.

Use web search, web fetch, and other available tools as necessary. Make sure that the comments or documentation are
not too verbose (do not add comments more fit for a PR summary than a comment). Do not leave arbitrary numbers (such
as issue or task numbers) in the code or documentation. Do not use inline comments. Do not use comments or
documentation that describe arbitrary code changes of previous states compared to the current code that do not need
explanation. The code commenting should reflect the current state of the codebase and be used to convey information
to an LLM bot or developer. Write American English -- color, behavior, center, initialize, canceled -- except
where a name belongs to something upstream, such as a Wayland `Cancelled` event or an NVENC `colourMatrix` field.

Empirical testing is possible for everything here, including implementation, auditing, validation and verification,
and every change is validated before it is reported. `cargo test --lib` in both feature configurations is the floor;
the `#[ignore]`d `gpu_` tests need an NVIDIA GPU (`cargo test gpu_ -- --ignored --nocapture --test-threads=1`,
serially, since concurrent session builds fault in the driver), the `gpu_dmabuf_` ones a render node as well, and the
`gpu_bench_` ones print measurements to quote rather than assert. The VA-API surface probe (`vpp_sw_formats`)
runs only where a VA-API device opens, so the CI runners and NVIDIA hosts never test it; it is proven on Intel
hardware in the selkies sandbox. End to end, a change is a wheel
(`pip wheel . --no-deps`) installed into a selkies sandbox as the Agentic Development section of that repository's
`docs/development.md` describes, driven by its suites over both transports on X11 and Wayland with the installed
Firefox and Chrome and Playwright/Selenium/Puppeteer/Cypress WebKit in place of Safari; the `.devcontainer` here
builds the extension with its native dependencies. Ask before building an environment on a machine that was not set
up for one (Miniforge serves a host with a closed package manager; keep the system `libgbm.so` for GBM on NVIDIA and
other GPUs) and take the operator's directives on how it is constructed and constrained. Say which checks could not
run where the hardware for them was not available.

Note that parity between X11 and Wayland, as well as between WebSockets and WebRTC, or between the default dashboard
and the wish dashboard, is considered a key focus (things that were not wired up correctly on either side, and similar
discrepancies, are subject to fixes or deduplication). I prefer deduplicating code that performs similar purposes
across different modes over keeping duplicate code for no reason and more fragility. Refactor through deduplication if
you are confident there will be no regressions (or able to validate regressions). Screen coroutine usage in both
Python and JavaScript, as well as thread usage in all languages, so that everything is performant and does not lead to
hanging or lagging. Performance preservation or improvements such as zero-copy and latency-reducing measures are
always important, and the GIL is held no longer than the work needs. End-to-end latency and an unrestricted frame
rate are separate goals rather than two ends of one dial: neither is spent to buy the other. A change never drops a
capability or falls back to an older implementation to make itself simpler; where one seems to be in the way, say
what it is rather than removing it. Note that compatibility should be ensured for Python 3.9 to 3.14 or even higher, and CUDA/NVENC 11
to 13 or higher. Protocol clients form fallback ladders that bind the newest architecture first (ext- before
zwlr-data-control in dcclient) and exist to keep selkies' Wayland path subprocess-free — they replace wtype/wl-copy
style forks, so extend them in-process rather than shelling out. A nested KWin session forwards no delta from its host
seat, so relative pointer motion reaches it through `org_kde_kwin_fake_input` on the app compositor socket
(`wayland/ficlient.rs`); wlroots sessions take the seat's `zwp_relative_pointer_v1` as before. Update the translations as well (and write/update additional entries if necessary) as necessary.
A defect that predates the change you are making is still in scope: finding it does not make it someone else's,
and "pre-existing" is not a reason to leave it. Fix it, or say precisely what is broken, what you ruled out, and
what you would do next. The same applies to a failure you cannot reproduce yet -- narrow it until it is either
fixed or precisely described, and never let a test that fails for an unknown reason pass unremarked.

A change is ready when four questions have answers, and the commit or pull request gives them to the reviewer:
was the defect, or the missing behavior, reproduced on the code before the change (a failing check or a measurement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selkies-project/pixelflux](https://github.com/selkies-project/pixelflux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
