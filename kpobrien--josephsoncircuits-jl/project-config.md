---
trigger: always_on
description: | API or quantity | Unit |
---

# Units and conventions

## Frequencies

| API or quantity | Unit |
|---|---|
| HB `ws`, `wp`, and `w` | rad/s |
| `FrequencyDependent(w -> ...)` | rad/s, evaluated at the magnitude of the mode frequency |
| Scattering callables and tabulated scattering frequencies | rad/s |
| `transientdemodulate`, `transientiq`, `transientquantumplan` | Hz |
| `transientnoise` frequencies, cutoff, and quadrature weights | Hz |
| `RationalScattering` fitting `frequencies` and pumped fitting `band` | Hz |
| Time, line delays, transient step size | s |

Use `w = 2pi*f` to convert Hz to rad/s. For ordinary component values,
negative-frequency evaluation follows conjugate symmetry. A scattering
provider can instead supply its own signed-frequency data; see
[`ScatteringParameters`](@ref).

## Current amplitudes

HB sources use complex Fourier coefficients. A single nonzero coefficient
`Ip` at angular frequency `wp` represents

```math
I(t) = 2\operatorname{Re}\{I_p e^{i\omega_p t}\}.
```

Its peak current is `2abs(Ip)` and its RMS current is `sqrt(2)*abs(Ip)`.
The zero-frequency coefficient is the DC current itself, without doubling.
`TransientSource` takes the instantaneous real current. Thus the same
real pump is written as `current = Icoeff` in HB and as
`t -> 2Icoeff*cos(wp*t)` in a transient solve.

A port source is a Norton current injected into the port's positive
terminal. With a matched termination of resistance `R`, a sinusoid of
peak current `Ipeak` launches available power `Ipeak^2*R/8`.
A named `CurrentSource` follows its declared orientation: current flows
out of its first terminal and into its second.

### Check the convention on a resistor

The circuit below is just a 50 Ω port termination. Its voltage Fourier
coefficient is `50Icoeff`; its instantaneous voltage has twice that peak.
The transient initial voltage is supplied because a resistor cannot start
at zero voltage under a nonzero current.

```@example amplitude
using JosephsonCircuits
c = Circuit([(:p1, 1, 0, Port(1; Z0 = 50.0))])
f, Icoeff = 1e9, 1e-6
hb = hbnlsolve((2pi*f,), (1,), [(mode = (1,), port = 1, current = Icoeff)],
    c; keyedarrays = false)
Vcoeff = im*2pi*f*JosephsonCircuits.phi0*only(hb.nodeflux)
@assert isapprox(Vcoeff, 50Icoeff; rtol = 1e-10)
p = transientproblem(c; sources = [TransientSource(1, t -> 2Icoeff*cospi(2f*t))])
initial = transientstate(p; voltage = [2*50Icoeff])
td = transientsolve(p, (0.0, 1/f); dt = 1/(100f), initialstate = initial)
@assert isapprox(td.voltage[1, 1], 2real(Vcoeff); rtol = 1e-10)
(real(Vcoeff), td.voltage[1, 1])
```

## Modes and signed frequencies

A pump mode is an integer tuple `m`, with physical angular frequency
`sum(m .* wp)`. Independent pumps form a multidimensional Fourier grid.
The real-transform representation stores nonnegative indices along the
first grid axis and both signs along the others, with redundant conjugate
modes removed. This is an index-space convention: a stored mode can have
negative physical frequency when there is more than one pump.

A signal mode is an offset from the signal frequency:

```math
\omega_m = \omega_s + \boldsymbol m\cdot\boldsymbol\omega_p.
```

For a single pump and a positive signal below the pump:

| Mode | Signed frequency | Interpretation |
|---|---|---|
| `(0,)` | `ws` | Signal |
| `(-2,)` | `ws - 2wp[1]` | Conjugate idler coordinate in four-wave mixing |
| `(-1,)` | `ws - wp[1]` | Conjugate idler coordinate in three-wave mixing |

A negative idler coordinate represents the conjugate of the physical
positive-frequency tone. The sign matters for phases and commutators.
For commensurate pumps, specify one fundamental frequency and drive its
harmonics through different source mode indices. Independent incommensurate
pumps describe a quasiperiodic state rather than one finite-period orbit.

## Photon gain and power gain

`LinearizedHB.S` is normalized to photon flux. For nonzero frequencies,

```math
G_{\mathrm{photon}} = |S_{oi}|^2,\qquad
G_{\mathrm{power}} = \frac{|\omega_o|}{|\omega_i|}|S_{oi}|^2.
```

Multiply `S` by `sqrt(abs(wout)/abs(win))` for the corresponding power-wave
amplitude. The absolute values are required for signed idler coordinates.
For equal input and output frequencies, the two gains coincide.

Transient `incident` and `outgoing` arrays are instantaneous real power
waves in `sqrt(W)`. A demodulated peak wave amplitude `a` at one tone has
average power `abs2(a)/2`. Do not compare these arrays directly with
photon-normalized HB coefficients.

`NonlinearHB.S` contains output-to-drive ratios for the solved strong-drive
state. With several driven port modes, each column contains the response
to all sources divided by that column's incident wave. Use `LinearizedHB.S`
for the differential scattering response about the operating point.

## Flux, phase, and state arrays

Write physical node flux as `Φ`, in webers, and reduced flux as
`φ = Φ/φ₀`, where `φ₀ = ħ/(2e)` is the reduced flux quantum.

| Quantity | Meaning |
|---|---|
| HB `nodeflux` | Fourier coefficients of reduced node flux |
| HB voltage coefficient at nonzero `w` | `im*w*phi0*nodeflux` |
| HB `dcnodevoltage` | Average node voltage in volts; separate from the static flux |
| `transientstate` input `flux`, `voltage` | Physical node flux in Wb and node voltage in V |
| Transient `voltage` | Port voltages in V |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kpobrien/JosephsonCircuits.jl](https://github.com/kpobrien/JosephsonCircuits.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
