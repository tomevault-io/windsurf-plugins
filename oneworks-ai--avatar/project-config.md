---
trigger: always_on
description: This project is the preview/export surface for the OneWorks avatar visual system.
---

# Avatar Assets Guide

This project is the preview/export surface for the OneWorks avatar visual system.

- `src/App.tsx`: standalone preview/export surface.
- `src/App.scss`: page layout and interaction styling.
- `.github/workflows/deploy-avatar.yml`: GitHub Pages deployment owned by the `oneworks-ai/avatar` repository.
- `.github/workflows/npm-publish.yml`: npm publishing for the four public SDK packages; OIDC Trusted Publishing remains the default, while an explicit token mode supports protected recovery without changing package bytes or provenance. Real publishing is restricted to `main`.
- `scripts/publish-sdk-packages.mjs`: pack, immutable-integrity reconciliation, ordered publish, and npm registry postflight for one shared SDK version.

The npm publisher binding must use repository `oneworks-ai/avatar`, workflow `npm-publish.yml`, environment `npm-publish`, and the `npm publish` allowed action. The protected environment allows only `main` and does not allow administrator bypass. OIDC mode exposes `NPM_TRUSTED_PUBLISHERS` only after all four bindings have been verified externally. Token mode is an explicit recovery path backed only by the `npm-publish` environment's `NPM_TOKEN` secret; it still publishes from this repository with npm provenance and must never weaken the exact-source, immutable-tarball, full-preflight, or postflight checks. Keep validation/packing, npm publishing, and GitHub release creation in separate least-privilege jobs.
- `packages/avatar/`: versioned framework-neutral 3D scene definitions and animation runtime.
- `packages/react/`: React renderer and the embeddable full editor; it is the single framework adapter allowed to import editor internals.
- `packages/web/`: Vanilla JS mounts and opt-in custom elements built on the React adapter.
- `packages/vue/`: Vue components built on the Vanilla JS mounts so rendering behavior remains identical across frameworks.
- `skills/oneworks-avatar/`: installable Agent Skill for 3D authoring, debugging, export verification, and current developer-integration boundaries.

The interactive preview uses geometric SVG face primitives. Its baseline face is a pair of rounded-rectangle eyes; keep new face work vector-native and do not replace it with web fonts, canvas text, or raster images. `packages/avatar` owns the public framework-neutral 3D definition and animation API, while this editor's Copy SVG and Download SVG actions serialize the current interactive SVG so body, pose, face, lighting, shadow, and outline appearance are preserved.

The interactive preview uses a sphere as the default avatar head surface. Keep geometric face primitives attached to the selected body's curved surface; any alternate head surface must be an explicit opt-in rather than replacing the spherical default. Tessellate every face boundary, including the nominally straight sides of rounded rectangles, before projecting it onto the body surface; projecting only edge endpoints leaves straight screen-space chords that visibly detach from the curved body. Face marks read as flush surface decals rather than recessed holes: render the foreground at full opacity and attenuate optional screen-space face shadows by the projected surface normal so they disappear near tangent angles instead of separating from the mark. The Build panel owns face geometry: eyes support rounded-rectangle and ellipse types plus width, height, gap, a shared rotation, independent additive left / right tilts, and rounded-rectangle corner ratio; the optional nose supports inverted-triangle, ellipse, and rounded types plus size, vertical position, and rotation. Build the inverted triangle as a continuous curved-sided Bézier silhouette with a domed top and rounded lower tip, not as straight edges with rounded vertices. The optional mouth is a curved geometric band controlled primarily by smile / flat / frown curvature plus width, thickness, and vertical position. Parameters persist in the URL and transition through stable interpolated geometry instead of jumping between values. Independent eye tilts are part of complete face state and must be captured and interpolated by animation keyframes; older saved animations that predate them resolve both offsets to zero. The Default face action is a compact 48px icon button that restores the complete baseline configuration rather than acting as a decorative selected card. Pointer drag updates the pose and leaves it at the released angle; do not automatically spring or reset the avatar back to center. Rotation is continuous on both axes: never clamp the pose, and occlude the geometric face while it travels behind the body.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oneworks-ai/avatar](https://github.com/oneworks-ai/avatar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
