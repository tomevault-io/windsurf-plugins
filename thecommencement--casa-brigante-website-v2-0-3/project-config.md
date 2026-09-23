---
trigger: always_on
description: - MENU CARD (music.html roster): was cutting off Elara Bloom on the right. vanta_card.jpg = full-width looking-at-each-other group; roster img now uses object-position center 30% so all four fit.
---

## VANTA fixes (menu card + hero)
- MENU CARD (music.html roster): was cutting off Elara Bloom on the right. vanta_card.jpg = full-width looking-at-each-other group; roster img now uses object-position center 30% so all four fit.
- HERO (artist-vanta.html): swapped from the looking-at-each-other shot (weird as a hero) to the FORWARD-FACING group (2e272768), with the baked "VANTA" title cropped off the top. Focal center 38%. The looking-at-each-other shot stays as the menu card only, per founder.

## VANTA finalized (bug fix + new photos)
- BUG FIXED: vanta_hero.jpg had been overwritten with an orange/Solana image (that's why the VANTA hero and Music-roster thumb showed Solana). Replaced with the correct clean gold-hall VANTA group shot (no text). Roster thumb auto-fixed (points at vanta_hero.jpg).
- New individual member portraits set from founder's gold-hall solo shots (Aria Vale, Juno Knox, Nyla Starr, Elara Bloom), cropped to the person, name-text excluded. Theme unchanged (gold-on-black gothic luxury), just finalized with better photos.
- VANTA is now DONE. Do not touch. Next: MOONREALM finalize, then Solana finalize (per founder's one-act-at-a-time approach — the right call).

## DEPARTMENT HERO STANDARD (the house formula, founder-ratified)
The MOONREALM/artist hero formula is now the standard for EVERY department front page. Reusable .dept-hero component (in styles.css): full-bleed cinematic image, top+bottom vignette, floating serif title over the lower third, uppercase gold eyebrow, italic one-line tagline with a gold hairline. Set the image per page via inline --dept-img.
- BOOKS uses it now: books-hero.jpg = the five covers arranged as a fanned shelf in warm gold light with edge-lighting and vignette (built from the real cover files; regenerate if covers change, especially when Bury Me in Tucson's real cover exists).
- ROLL-OUT PENDING (apply .dept-hero to each, founder to approve image per department): HAWS, Vault 12, Audio Dramas, Plug & Play, The House, Inquiries. Each needs a hero image: either its own key art or the estate emblem as fallback.
- Books hero image is COMPOSITE-GENERATED, not a photo; when the founder supplies real product/lifestyle photography it can replace books-hero.jpg (CSS already points at it).

## Artist heroes elevated (press-kit -> film-poster)
- All three artist heroes now use CLEAN, TEXT-FREE cinematic key art (not press-kit sheets): vanta_hero.jpg (gothic cathedral group), moonrealm_hero.jpg (gold eclipse duo, bottom text cropped), solana_hero.jpg (golden-hour rust, baked logo cropped out). Music roster thumbnails updated to match.
- Hero treatment: 92vh, cinematic top+bottom vignette, poster-credit hairline under the subtitle, stronger title text-shadow. Each keeps its own accent + motion. Old *_anchor.jpg press sheets remain only for internal reference, not shown as backgrounds.
- If founder supplies even cleaner per-artist key art later, swap the *_hero.jpg files; CSS already points at them.

## Artist parity pass
All three artist pages now share the same section rhythm (hero, statement, world, members/song, credits) and each ends its credits with a "More from the Casa Brigante roster" link back to music.html. VANTA and MOONREALM layouts/colors/motion were NOT otherwise changed (they were founder-approved); only the universe cross-link was added for consistency with Solana.

## SOLANA REYES room built (3rd artist — completes the novel<->music bridge, the proof of concept)
- artist-solana-reyes.html: concrete/rust/paper aesthetic (property accent RUST #b8552e on concrete-gray/paper; gold stays institutional). Near-still motion (no drift, no float) = her signature: patience, confinement. solana.css holds styles.
- Sections: hero ("The voice prison couldn't silence."), Artist Statement ("Music became the one place they couldn't lock me out."), Her World, The Song ("ghost" + discography: ghost/Letters to Sergio/ADC Anthem/Five Life Sentences/Until Then), From the Novel (A Song She Wrote, links to book-a-song-she-wrote.html), Credits.
- SENSITIVE-CONTENT HANDLING (deliberate, per founder + safety): Solana's canon includes incarceration for the livestream killings and 5 life sentences. The public page leads with MUSIC and WRITING, uses the prison setting as atmosphere and mythology, and does NOT dramatize, detail, or glorify the violence. The full story lives in the novel; the page points there. Keep it this way.
- THE BRIDGE IS LIVE BOTH WAYS: Solana artist page -> A Song She Wrote novel page, and the novel is her canon source. This is the interconnected-IP proof the whole Creative Experience Standard was built to validate.
- All THREE artists now live in the Music roster, each a distinct world: VANTA (gold/cold/still), MOONREALM (purple/nocturnal/drift), Solana (rust/concrete/still-but-human).
- PENDING: Solana streaming links when "ghost" is live; confirm anchor/portrait image choices; her canon is heavy (Afro-Latina, Mesa AZ, brother Sergio OD) — never invent beyond the bible.

## MOONREALM room built (2nd artist, proves the multi-room system)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheCommencement/Casa_Brigante_Website_v2_0_3](https://github.com/TheCommencement/Casa_Brigante_Website_v2_0_3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
