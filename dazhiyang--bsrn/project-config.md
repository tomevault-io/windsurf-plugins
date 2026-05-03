---
trigger: always_on
description: Standards and governance for the bsrn project, including naming conventions and documentation rules.
---


# 🤖 Project Standards & Governance

This document defines the naming conventions, scientific symbols, and documentation standards used throughout the `bsrn` project.

**Contributors and automated agents** MUST read and **apply** the rules below before editing project code (especially §Documentation — function signatures and docstrings). Treat this file as binding, not background.

## 📏 Radiometric Parameters

Code variables and documentation **MUST** strictly adhere to the following table:

| Acronym | Code Name | Symbol | Full Name |
| :--- | :--- | :--- | :--- |
| **GHI** | `ghi` | $G_h$ | global horizontal irradiance |
| **BNI** | `bni` | $B_n$ | beam normal irradiance |
| **DHI** | `dhi` | $D_h$ | diffuse horizontal irradiance |
| **LWD** | `lwd` | $L_d$ | downward longwave radiation |
| **LWU** | `lwu` | $L_u$ | upward longwave radiation |
| **SWU** | `swu` | $S_u$ | upward shortwave radiation |
| **NET** | `net` | $R_n$ | net radiation |
| **SZA** | `zenith` | $Z$ | solar zenith angle |
| **cosSZA** | `mu0` | $\mu_0$ | cosine of SZA |
| **SAA** | `azimuth` | $\phi$ | solar azimuth angle |
| **GHIC** | `ghi_clear` | $G_{hc}$ | clear-sky GHI |
| **BNIC** | `bni_clear` | $B_{nc}$ | clear-sky BNI |
| **DHIC** | `dhi_clear` | $D_{hc}$ | clear-sky DHI |
| **LWDC** | `lwd_clear` | $L_{dc}$ | clear-sky LWD |
| **LWUC** | `lwu_clear` | $L_{uc}$ | clear-sky LWU |
| **BNIE** | `bni_extra` | $E_{0n}$ | extraterrestrial BNI |
| **GHIE** | `ghi_extra` | $E_{0}$ | extraterrestrial GHI |
| **SC** | `solar_constant` | $E_{\text{sc}}$ | solar constant |
| **CSI** | `kappa` | $\kappa$ | clear-sky index |
| **k_t** | `kt` | $k_t$ | clearness index |
| **K_t** (daily) | `Kt` | $K_t$ | daily clearness index |
| **k_b** | `kb` | $k_b$ | beam transmittance |
| **k_d** | `kd` | $k_d$ | diffuse transmittance |
| **k** | `k` | $k$ | diffuse fraction |
| **TMP** | `temp` | $T$ | air temperature |
| **RH** | `rh` | $RH$ | relative humidity |
| **SP** | `pressure` | $P$ | station pressure |

---

## 🚩 Quality Control Flags
- For the data points that do not pass **PPL test**, we **MUST** use the following flag names:
    - `flagPPLGHI`: Flag for GHI physically possible limit test
    - `flagPPLBNI`: Flag for BNI physically possible limit test
    - `flagPPLDHI`: Flag for DHI physically possible limit test
    - `flagPPLLWD`: Flag for LWD physically possible limit test
- For the data points that do not pass **ERL test**, we **MUST** use the following flag names:
    - `flagERLGHI`: Flag for GHI extremely rare limit test
    - `flagERLBNI`: Flag for BNI extremely rare limit test
    - `flagERLDHI`: Flag for DHI extremely rare limit test
    - `flagERLLWD`: Flag for LWD extremely rare limit test
- For the data points that do not pass **Closure test**, we **MUST** use the following flag names:
    - `flag3lowSZA`: Flag for closure test at low SZA ($Z \le 75^\circ$)
    - `flag3highSZA`: Flag for closure test at high SZA ($Z > 75^\circ$)
- For the data points that do not pass **Diffuse ratio (k) test**, we **MUST** use the following flag names:
    - `flagKKt`: Flag for combined $k$ and $k_t$ test ($k < 0.96$)
    - `flagKlowSZA`: Flag for diffuse ratio test at low SZA ($Z < 75^\circ$, $k < 1.05$)
    - `flagKhighSZA`: Flag for diffuse ratio test at high SZA ($Z \ge 75^\circ$, $k < 1.1$)
- For the data points that do not pass **k-index test**, we **MUST** use the following flag names:
    - `flagKbKt`: Flag for $k_b < k_t$ test
    - `flagKb`: Flag for $k_b$ physical limit test
    - `flagKt`: Flag for $k_t$ physical limit test
- For the data points that do not pass **Tracker-off test**, we **MUST** use the following flag name:
    - `flagTracker`: Flag for solar tracker failure detection

---

## 📝 Documentation & Coding Rules

> ### **REQUIRED — Public function docstrings (NumPy style)**
>
> Every **function** and **public method** (including module-level callables and class methods that form the API) **MUST** use NumPy-style docstrings in English (see **Docstring Structure** below).
>
> **Required sections (in order):**
>
> 1. **Summary** — concise English description.
> 2. **`Parameters`** — always present if the callable has parameters (use `None` / optional wording where needed).
> 3. **`Returns`** — always present unless the callable returns `None` *and* that is obvious; prefer an explicit ``None`` description when the function exists only for side effects.
> 4. **`Raises`** — **whenever** the body can raise a documented exception (typically `ValueError`, `TypeError`, `KeyError`, etc.). Omit the section only if truly no exception is part of the contract.
>
> **`References`** — still **MUST** appear when the implementation follows a paper or external spec (see §2).
>
> Data-only modules (e.g. large static dicts with no functions) are exempt from per-function docstrings.

> ### **REQUIRED — Function signature layout (no `->`, wrap at 80)**
>
> - **MUST NOT** use PEP 484 **return** annotations on definitions: do **not** write `-> SomeType` after the closing `)` of `def` (e.g. forbid `def f(x) -> str:`). Describe return types in the **`Returns`** section of the docstring instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dazhiyang/bsrn](https://github.com/dazhiyang/bsrn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
