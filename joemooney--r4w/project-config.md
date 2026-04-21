---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**R4W - Rust for Waveforms** - A platform for developing, testing, and deploying SDR waveforms in Rust. Provides reusable DSP libraries, educational tools, and production-ready components for waveform development.

### Architecture

- **r4w-core**: Core DSP algorithms, timing, RT primitives, configuration
  - `waveform/`: 42+ waveform implementations (including GNSS: GPS L1 C/A, GPS L5, GLONASS L1OF, Galileo E1)
  - `waveform/gnss/environment/`: Keplerian orbits, Klobuchar ionosphere, Saastamoinen troposphere, multipath presets, antenna patterns
  - `waveform/gnss/scenario*.rs`: Multi-satellite GNSS IQ scenario generation with realistic channel effects
  - `coordinates.rs`: ECEF/LLA coordinate types, geodetic conversions, look angles, range rate, FSPL
  - `filters/`: Digital filters with trait-based architecture
    - `traits.rs`: Filter, RealFilter, FirFilterOps, FrequencyResponse traits
    - `fir.rs`: FirFilter with lowpass/highpass/bandpass/bandstop, Kaiser window design
    - `iir.rs`: IirFilter with Butterworth, Chebyshev I/II, Bessel via cascaded biquads
    - `polyphase.rs`: PolyphaseDecimator, PolyphaseInterpolator, Resampler, HalfbandFilter
    - `remez.rs`: Parks-McClellan equiripple FIR design (RemezSpec builder)
    - `pulse_shaping.rs`: RRC, RC, Gaussian filters (also implement Filter traits)
    - `windows.rs`: Hamming, Hann, Blackman, Kaiser window functions
  - `agc.rs`: Automatic Gain Control (Agc, Agc2, Agc3 - basic/dual-rate/fast-acquisition)
  - `carrier_recovery.rs`: Costas loop for BPSK/QPSK/8PSK carrier recovery
  - `clock_recovery.rs`: Mueller & Muller symbol timing recovery, FreqXlatingFirFilter
  - `crc.rs`: CRC engine (CRC-8, CRC-16 CCITT/IBM, CRC-32, CRC-32C) with table lookup
  - `equalizer.rs`: Adaptive equalizer (LMS, CMA, Decision-Directed) with Filter trait
  - `signal_source.rs`: Signal generator (Tone, TwoTone, Chirp, Noise, Square, DC, Impulse)
  - `squelch.rs`: Power squelch gate with ramp transitions (equiv. to GNU Radio pwr_squelch_cc)
  - `ofdm.rs`: OFDM modulator/demodulator (WiFi-like, DVB-T 2K, simple configs)
  - `pfb_channelizer.rs`: Polyphase filter bank channelizer with windowed-sinc prototype filter
  - `correlator.rs`: Cross-correlation sync word detector (Barker codes, dynamic threshold, holdoff)
  - `scrambler.rs`: LFSR scrambler/descrambler (additive/multiplicative, WiFi/DVB-S2/Bluetooth/V.34)
  - `differential.rs`: Differential encoder/decoder (DBPSK/DQPSK/D8PSK, complex-domain DPSK)
  - `packet_framing.rs`: Packet formatter/parser (sync word, headers, CRC, AX.25/ISM configs)
  - `nco.rs`: NCO/VCO + FM modulator/demodulator (PLL building block, frequency translation)
  - `snr_estimator.rs`: SNR estimation (M2M4, split-spectrum, signal+noise, EVM methods)
  - `symbol_mapping.rs`: Constellation mapper/demapper (BPSK/QPSK/8PSK/16QAM/64QAM, soft LLR)
  - `probe.rs`: Measurement probes (Power/PAPR, EVM, constellation scatter, freq offset, rate)
  - `pll.rs`: Second-order PLL, DC blocker (IIR highpass), sample delay (circular buffer)
  - `costas_loop.rs`: Costas loop carrier recovery (BPSK/QPSK/8PSK decision-directed)
  - `constellation_receiver.rs`: Combined AGC + Costas + symbol demapper receiver
  - `goertzel.rs`: Goertzel single-frequency DFT + MultiGoertzel + DTMF detector
  - `burst_detector.rs`: Power-based burst detector with hysteresis (SOB/EOB events)
  - `noise.rs`: Colored noise generator (white/pink/brown/blue/violet) + AWGN helpers
  - `stream_tags.rs`: Metadata propagation (TagStore, TagValue, range queries, well-known keys)
  - `filters/cic.rs`: CIC decimator/interpolator for high-ratio sample rate conversion
  - `filters/adaptive.rs`: Adaptive filters (LMS, NLMS, RLS) for equalization/cancellation
  - `filters/fractional_resampler.rs`: MMSE interpolating resampler for arbitrary rate conversion
  - `freq_xlating_fir.rs`: Frequency-translating FIR filter (mixing + FIR + decimation)
  - `fm_emphasis.rs`: FM pre-emphasis/de-emphasis (US 75us / EU 50us, 1-pole IIR)
  - `quadrature_demod.rs`: FM discriminator y[n] = gain * arg(x[n] * conj(x[n-1]))
  - `access_code_detector.rs`: Bit-level sync word detector with Hamming distance threshold
  - `fll_band_edge.rs`: FLL band-edge coarse frequency sync (band-edge FIR + 2nd-order loop)
  - `type_conversions.rs`: Complex/Real converters (Mag, MagSq, Arg, Real, Imag, MagPhase)
  - `stream_control.rs`: Head (first N), SkipHead (drop N), Throttle (rate-limit)
  - `log_power_fft.rs`: Windowed FFT to dB power with exponential averaging
  - `pdu.rs`: PDU↔tagged stream conversion, message debug (Hex/Text/Decimal/Summary)
  - `ofdm_channel_est.rs`: OFDM pilot-based channel estimation (LS/smoothed), ZF/MMSE equalization
  - `ssb_modem.rs`: SSB modulator/demodulator (Hilbert transform phasing method, BFO)
  - `wavelet.rs`: DWT analysis/synthesis (Haar/Db4/Sym4), soft/hard threshold denoising
  - `cpm.rs`: CPM/GMSK/GFSK/MSK modulation (constant-envelope, Gaussian/LRC pulse shapes)
  - `dynamic_channel.rs`: Composite time-varying channel (fading + CFO/SRO drift + AWGN)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joemooney) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
