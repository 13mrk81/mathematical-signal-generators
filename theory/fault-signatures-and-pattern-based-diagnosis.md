# Fault Signatures and Pattern-Based Diagnosis

## 1. Motivation

A deterministic signal or sequence generator can be used as a controlled stimulus for a system under test (SUT). Instead of reducing the result to pass/fail, compare the measured response with the expected response and study the structure of the deviation.

The central idea is:

**Stimulus pattern → system response → residual pattern → candidate fault class**

The pattern does not automatically identify a unique fault. It provides evidence that helps detect anomalies, narrow hypotheses, and choose the next experiment.

## 2. Mathematical formulation

Let:

- \(u[n]\): applied test stimulus
- \(y_{ref}[n]\): expected response from a reference model
- \(y_{obs}[n]\): measured response from the real system

For numerical signals, define the residual:

\[
e[n] = y_{obs}[n] - y_{ref}[n]
\]

For digital words, a bitwise mismatch vector is often more useful:

\[
e[n] = y_{obs}[n] \oplus y_{ref}[n]
\]

The residual can be analyzed in several dimensions:

- **Temporal:** when do errors occur? Are they periodic, bursty, or isolated?
- **Spatial/address:** where do errors occur? Are they tied to particular addresses, pixels, or channels?
- **Bit-level:** which bit positions fail, and do they fail together?
- **Statistical:** what are the error rate, distribution, autocorrelation, and run lengths?
- **Conditional:** how does the error depend on clock frequency, temperature, reset sequence, traffic, or burst length?

The diagnostic observation is therefore not just the number of errors, but a feature vector extracted from the residual:

\[
\phi = \Phi(e[n],\; \text{address},\; \text{time},\; \text{operating conditions})
\]

A fault-signature database associates such feature patterns with candidate fault classes.

## 3. Fault signature is evidence, not proof

A fault signature is an observable response pattern associated with a fault model. In a simplified classification view:

\[
\phi_{obs} \longrightarrow \{F_1,F_2,\ldots,F_k\}
\]

The result may be a ranked set of hypotheses rather than a single answer. Multiple faults can produce similar signatures, and the same fault can look different under different stimuli or operating conditions.

A useful diagnostic system must therefore track:
- the observed signature,
- candidate explanations,
- confidence or evidence strength,
- alternative explanations,
- and the additional experiment that could distinguish them.

## 4. Examples of stimulus patterns

| Stimulus | What it helps investigate |
|---|---|
| Constant all-zero / all-one words | Stuck-at behavior and basic data-path integrity |
| Walking one / walking zero | Individual data bits, lanes, and bit mapping |
| Alternating 1010… / 0101… | Neighboring-bit interaction and transition-sensitive behavior |
| Address-derived data | Address/data swaps, aliasing, and address-dependent corruption |
| Monotonic ramp or sample counter | Missing, duplicated, reordered, or truncated samples |
| PRBS / LFSR sequence | Broad transition activity, deterministic stress, and correlation-based checks |
| Boundary-focused bursts | Off-by-one errors, burst termination, and state transitions |
| Multiple clock rates | Timing-margin and frequency-dependent failures |

No stimulus is universally best. Pattern selection should follow the fault model and the system's interface behavior.

## 5. Example: memory write/read diagnosis

Suppose a memory test writes a known word to each address and reads it back.

For each address \(a\), define:

\[
e[a] = D_{read}[a] \oplus D_{expected}[a]
\]

Do not only count nonzero values of \(e[a]\). Record:
- failing addresses,
- failing bit positions,
- whether errors cluster at burst boundaries,
- whether the same pattern repeats across runs,
- dependence on address, data value, burst length, and clock rate.

Example interpretations:

| Observed signature | Candidate explanations |
|---|---|
| Same address fails on every run | Address decode, address generation, localized memory issue |
| Same bit fails across many addresses | Data lane, pin mapping, mask, or bit-slice issue |
| Errors appear at the first/last word of bursts | Burst counter, command timing, boundary handling |
| Errors appear only at higher clock rates | Timing closure or signal-integrity margin |
| Failures vary with clock-domain phase | CDC or handshake assumptions |

These interpretations guide the next experiment; they do not establish root cause by themselves.

## 6. Diagnostic workflow

### Step 1 — Define the golden behavior

Specify:
- expected values,
- ordering,
- latency,
- valid/ready or equivalent handshake behavior,
- reset and initialization behavior,
- permitted tolerance,
- and operating conditions.

A reference model must represent the intended behavior independently of the implementation being tested.

### Step 2 — Choose a discriminating stimulus

Choose patterns to expose specific fault classes. Use multiple complementary patterns rather than relying on a single pseudo-random sequence.

### Step 3 — Capture enough context

Log the stimulus, response, sample/address index, timestamps or cycle count, configuration, reset history, and clock settings. Without context, an error pattern can be misinterpreted.

### Step 4 — Extract residual features

Possible features include:
- error count and error rate,
- first and last failing index,
- run-length distribution,
- failing bit mask,
- periodicity,
- autocorrelation or cross-correlation,
- burst-position histogram,
- and dependence on operating conditions.

### Step 5 — Form and rank hypotheses

Map the observed features to plausible fault classes. Keep more than one hypothesis when signatures overlap.

### Step 6 — Design the next experiment

Choose a stimulus or operating condition that makes the remaining hypotheses predict different outcomes. This is often more useful than simply repeating the same test.

### Step 7 — Confirm the root cause

Reproduce the signature, change one relevant factor at a time, and verify the suspected cause with an independent test.

## 7. Connection to mathematical signal generation

Compact deterministic generators are valuable diagnostic stimulus sources because they can:
- generate repeatable test vectors without storing large datasets,
- vary seed, initial state, recurrence parameters, or sequence length,
- produce patterns aligned to addresses or sample indices,
- generate one sample per clock in streaming hardware,
- and reproduce the exact stimulus used during a previous failure.

Examples:
- LFSR/PRBS for transition-rich deterministic sequences,
- counters and ramps for ordering and continuity,
- address-derived patterns for memory mapping,
- periodic functions for frequency- and phase-sensitive tests,
- carefully selected impulse or step inputs for dynamic-system response analysis.

This creates a direct link between recurrence relations, discrete-time systems, sequence properties, verification, and system diagnosis.

## 8. Metrics for evaluating a diagnostic method

| Metric | Meaning |
|---|---|
| Fault coverage | Fraction of modeled faults that produce a detectable deviation |
| Detection rate | Probability of detecting a fault when present |
| False-alarm rate | Probability of flagging a healthy system |
| Isolation resolution | Ability to distinguish one fault class from another |
| Diagnostic latency | Time or number of tests required to identify a likely fault |
| Repeatability | Stability of the signature across repeated runs |
| Robustness | Sensitivity to normal operating variation |
| Test cost | Time, memory, hardware resources, and data volume needed |

Detection and isolation are different objectives: a method may reliably detect an anomaly while being unable to identify its exact cause.

## 9. Further concepts to study

- Fault Detection and Isolation (FDI)
- Model-Based Diagnosis
- Fault Signature Analysis
- Design for Testability (DFT)
- Built-In Self-Test (BIST)
- Residual generation and residual evaluation
- Statistical hypothesis testing and classification
- Information gain and experiment design
- Correlation and spectral analysis of error sequences

## 10. Suggested practical experiment

Build a small software model of a memory or streaming datapath:

1. Generate a known sequence of test words.
2. Pass it through a fault-free reference model.
3. Inject one fault at a time (for example, stuck-at bit, address alias, dropped sample, or one-cycle offset).
4. Compute the residual and extract simple features.
5. Compare signatures across fault types.
6. Find pairs of faults that look similar.
7. Design an additional test pattern that separates each ambiguous pair.

Only after this software experiment should the same methodology be transferred to an FPGA testbench or on-board diagnostic setup.

## References and search terms

- Fault Detection and Isolation (FDI)
- Model-Based Diagnosis
- Fault Signature Analysis
- Design for Testability
- Built-In Self-Test
- Test-pattern generation
- PRBS and LFSR sequence testing
- Residual analysis
- Statistical pattern recognition
