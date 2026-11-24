# Stateful Mathematical Signal & Image Generation

> Research idea: generate complex signals, sequences, and visual patterns from compact mathematical descriptions, using minimal state, memory, and hardware resources.

## 1. Project vision
Investigate how signals, sequences, and images can be generated on demand from mathematical relationships instead of storing complete datasets.

**Central question:** How much state, memory, and computation are required to generate a desired signal or image?

The long-term goal is to understand the mathematics behind compact generators and translate suitable methods into efficient real-time FPGA architectures.

## 2. Core mathematical model
A coordinate-based (stateless) generator can be written as:

\[
RGB(x,y)=F(x,y;\theta)
\]

where \(x,y\) are pixel coordinates and \(\theta\) is a compact parameter set.

A stateful generator can be described by:

\[
S_{n+1}=F(S_n,U_n), \qquad Y_n=G(S_n,U_n)
\]

where \(S_n\) is internal state, \(U_n\) is an optional input, and \(Y_n\) is the generated output. If output is consumed as a stream, previously generated samples need not be retained.

For comparison, a 640 × 480 RGB888 frame requires:

\[
640\times480\times24=7,372,800\text{ bits}=900\text{ KiB}
\]

A compact generator may avoid this frame buffer when the image has exploitable structure.

## 3. Important distinction: generation, compression, reconstruction
These are related but different problems:

1. **Procedural generation:** create outputs that follow a mathematical rule.
2. **Compact representation / compression:** encode a particular existing signal or image efficiently, then reconstruct it.
3. **Stateful sequence generation:** produce samples one by one from a recurrence or finite-state machine.
4. **Exact arbitrary-image reproduction:** a tiny fixed state cannot reproduce an arbitrary independent image unless the missing information is encoded or supplied elsewhere.

The research must quantify the trade-off between fidelity, model size, state size, computation, and throughput. There is no information-theoretic shortcut around the information content of arbitrary data.

## 4. Candidate mathematical families

### 4.1 Recurrence relations and discrete-time systems
\[
S_{n+1}=F(S_n)
\]
Study linear and nonlinear recurrences, higher-order systems, fixed points, periodicity, and stability.

### 4.2 Linear Feedback Shift Registers (LFSRs)
Compact deterministic binary sequence generators based on shift registers and XOR feedback. Investigate feedback polynomials, maximal-length sequences, period, autocorrelation, cross-correlation, and hardware parallelization.

### 4.3 Linear Congruential Generators (LCGs)
\[
X_{n+1}=(aX_n+c)\bmod m
\]
Use as a case study in modular arithmetic, deterministic pseudorandom sequences, period conditions, and hardware implementation.

### 4.4 Cellular automata
Grid-based systems that evolve through local update rules. Begin with one-dimensional elementary cellular automata, then explore two-dimensional rules for image-like patterns.

### 4.5 Fractals and iterated maps
Mandelbrot and Julia sets generate rich images from compact equations. Study:
\[
z_{n+1}=z_n^2+c
\]
including iteration count, escape conditions, fixed-point precision, and computational cost per pixel.

### 4.6 Procedural noise and spatial functions
Explore coordinate-based deterministic functions, periodic functions, value noise, gradient noise, and multiscale constructions.

### 4.7 Dynamical systems and chaos
Study sensitivity to initial conditions, boundedness, periodicity, bifurcations, and the effect of finite numerical precision.

## 5. Applications

- **Digital communications:** PN sequences, spreading codes, synchronization sequences, and waveform design.
- **Radar:** waveform design and correlation properties.
- **DSP:** recursive filters, oscillators, and streaming signal generators.
- **FPGA graphics:** procedural textures, fractals, test patterns, and generated video.
- **Verification:** deterministic stimulus and compact test-pattern sources.
- **Data representation:** model-based coding and compression where the source has exploitable structure.
- **System diagnosis:** generate controlled test patterns, observe the system response, and associate recurring response/error patterns with candidate fault classes. This connects mathematical signal generation to fault-signature analysis and model-based diagnosis.

## 6. Pattern-to-fault mapping and fault-signature analysis

A generated signal or sequence can be more than an output to inspect: it can be a controlled stimulus for investigating a system. The key idea is to compare the observed response with an expected reference and characterize the *structure* of any deviation.

### 6.1 Reference behavior and observed behavior

Represent a test stimulus by \(u[n]\), the expected response by \(y_{ref}[n]\), and the measured response by \(y_{obs}[n]\). Define the residual:

\[
e[n]=y_{obs}[n]-y_{ref}[n]
\]

For digital systems, the residual may instead be a bitwise mismatch vector:

\[
e[n]=y_{obs}[n]\oplus y_{ref}[n]
\]

The residual is not merely a pass/fail result. Its temporal, spatial, bit-level, and statistical structure may carry diagnostic information.

### 6.2 Fault signatures

A fault signature is a set of observable features associated with a fault or fault class. Examples include:

| Observed pattern | Candidate explanations to investigate |
|---|---|
| Errors repeatedly occur at the same address | Address decoding, address generation, or a localized memory fault |
| The same data bit is wrong across many words | Data path, pin mapping, mask handling, or bit-lane issue |
| Errors cluster at burst boundaries | Burst control, counter transitions, or protocol timing |
| Errors increase with clock frequency | Timing margin, setup/hold, or signal integrity |
| Errors depend on clock-domain interaction | CDC, asynchronous reset release, or handshake design |
| Missing or duplicated samples | Buffer overflow/underflow, enable logic, or handshake |
| A consistent phase or sample offset | Latency accounting, framing, or alignment |

These are diagnostic clues, not one-to-one proofs. Different faults can produce similar signatures, and one fault can produce different signatures under different stimuli or operating conditions.

### 6.3 A disciplined diagnostic workflow

1. **Define the golden behavior:** specify expected outputs, timing, latency, valid windows, and permitted tolerances.
2. **Design a discriminating stimulus:** choose deterministic patterns that expose particular failure modes (for example, walking ones/zeros, address-dependent words, alternating bits, PRBS, ramps, and boundary-focused bursts).
3. **Capture the response:** preserve the stimulus, observed output, timestamps or sample indices, configuration, and operating conditions.
4. **Extract features:** measure mismatch locations, burst lengths, bit positions, periodicity, correlation, error rate, and dependence on address or clock.
5. **Build candidate signatures:** associate feature patterns with plausible fault classes, while recording confidence and alternative explanations.
6. **Choose the next experiment:** select a new stimulus or operating condition that best distinguishes the remaining hypotheses.
7. **Confirm and document:** reproduce the signature and verify the suspected cause with an independent test.

### 6.4 Why mathematical signal generation matters here

A compact deterministic generator can produce repeatable, parameterized stimuli without storing large test vectors. Different seeds, initial states, recurrence parameters, periods, and pattern families can be selected to probe different behaviors.

For example, an LFSR/PRBS can exercise many data transitions, while address-derived patterns can make address/data swaps or aliasing easier to detect. A ramp can expose ordering and dropped-sample problems; walking-bit patterns can isolate data lanes. No single pattern is sufficient for every fault class.

### 6.5 Limits and evaluation

A fault-signature mapping is an inference mechanism, not a guarantee of unique diagnosis. Evaluate it using:
- **Detection coverage:** which injected or known faults produce a detectable deviation?
- **Isolation resolution:** can the method distinguish fault classes, or only detect an anomaly?
- **False alarms and missed detections:** how often is normal variation misclassified or a fault overlooked?
- **Repeatability:** does the same condition produce a stable signature?
- **Diagnostic cost:** how many tests, samples, and resources are required?
- **Robustness:** does the signature remain useful across voltage, temperature, frequency, and workload changes?

This topic provides a bridge between deterministic signal generation, verification, test-pattern design, statistical feature extraction, and fault detection and isolation (FDI).

## 7. Mathematics and theory roadmap

Suggested learning order:

1. **Discrete mathematics:** functions, relations, modular arithmetic, recurrence relations, finite-state machines.
2. **Linear algebra:** vectors, matrices, linear transformations, eigenvalues/eigenvectors, state-space models.
3. **Number theory:** congruences, primes, modular inverses, polynomial arithmetic over finite fields.
4. **Probability and statistics:** random variables, distributions, expectation, variance, correlation, sequence tests.
5. **Signals and systems:** discrete-time systems, LTI systems, convolution, Fourier analysis, sampling, quantization.
6. **Information theory:** entropy, source coding, rate-distortion, redundancy, limits of lossless compression.
7. **Dynamical systems:** fixed points, stability, periodic orbits, nonlinear maps, bifurcations, chaos.
8. **Digital arithmetic and hardware:** fixed-point formats, quantization, overflow, modular arithmetic, resource sharing, pipelining, timing closure.
9. **System testing and diagnosis:** testability, fault models, residual analysis, fault signatures, detection and isolation, statistical decision methods.

## 8. Proposed FPGA investigation

### Initial demonstrator: streaming mathematical image generator

Generate pixels in raster order without a full-frame buffer.

**Provisional target:**
- Resolution: 640 × 480
- Pixel format: RGB565
- Inputs: pixel coordinates and a small parameter set
- Output: streaming pixel data
- Storage: algorithm state, pipeline registers, and small lookup tables only if needed
- No full-frame memory

**Candidate implementations:**
1. Coordinate-based mathematical pattern.
2. LFSR-based deterministic texture or test pattern.
3. Cellular automaton.
4. Mandelbrot or Julia renderer.
5. Procedural noise generator.

### Architecture questions

- Is the algorithm stateless, stateful, or hybrid?
- What is the minimum state and parameter size?
- Does output depend on coordinates, previous output, or both?
- Can it sustain the required pixel rate?
- How many multipliers, adders, comparators, LUTs, and DSP blocks are required?
- What fixed-point precision is necessary?
- How do rounding, truncation, and overflow affect output?
- Can it be pipelined without changing mathematical behavior?
- Is random pixel access required, or is raster streaming sufficient?

## 9. Evaluation methodology

| Metric | Measurement |
|---|---|
| State size | Number of state bits / registers |
| Parameter size | Bits required to configure the model |
| Auxiliary memory | LUT, ROM, BRAM, or external memory |
| Logic resources | LUTs, FFs, DSP blocks |
| Throughput | Samples or pixels per second |
| Latency | Cycles from input to valid output |
| Clock frequency | Achieved timing after implementation |
| Numerical quality | Error against a high-precision reference |
| Output quality | Signal- or image-specific quality metrics |
| Reproducibility | Same seed and parameters produce same output |

Compare every hardware implementation against a software reference. Separate algorithmic limitations from fixed-point and implementation errors.

## 10. Suggested repository structure

```text
.
├── README.md
├── theory/
│   ├── recurrence-relations.md
│   ├── lfsr-and-pn-sequences.md
│   ├── cellular-automata.md
│   ├── fractals-and-iterated-maps.md
│   ├── information-theory.md
│   └── fault-signatures-and-pattern-based-diagnosis.md
├── experiments/
│   ├── software-models/
│   └── fixed-point-models/
├── rtl/
│   ├── generators/
│   └── common/
├── verification/
│   ├── testbenches/
│   └── reference-models/
├── results/
│   ├── resource-reports/
│   └── images-and-plots/
└── docs/
    ├── architecture/
    └── references/
```

## 11. First milestones

### Milestone 1 — Understand the models
- Implement several recurrence relations in MATLAB or Python.
- Plot their output sequences.
- Examine periods, state size, and sensitivity to initial conditions.

### Milestone 2 — Software image generators
- Generate a coordinate-based pattern.
- Generate an LFSR-based pattern.
- Generate one fractal image.
- Save reference outputs and document each algorithm.

### Milestone 3 — Hardware cost analysis
- Select one simple generator.
- Derive fixed-point equations.
- Estimate registers, arithmetic units, and throughput.
- Determine whether pipelining is needed.

### Milestone 4 — VHDL implementation
- Separate datapath from control/interface logic.
- Add a valid/ready or equivalent streaming interface.
- Verify against the software reference.
- Synthesize and record resource and timing reports.

### Milestone 5 — Compare approaches
Compare memory, logic cost, throughput, latency, and output quality. Identify which mathematical models are especially suitable for FPGA implementation.

### Milestone 6 — Diagnostic pattern experiments
- Define a simple system under test and its expected response.
- Generate multiple deterministic stimuli (ramp, alternating bits, walking bits, address-derived patterns, and PRBS).
- Inject or emulate selected faults.
- Capture response residuals and extract fault signatures.
- Test whether additional stimuli can distinguish faults with similar signatures.

## 12. Initial research questions

1. Which classes of images can be represented accurately by a compact mathematical model?
2. How should model parameters be distinguished from dynamic state?
3. What is the minimum state size for a given deterministic sequence?
4. Which generators can produce one output sample per clock?
5. How does finite word length change nonlinear system behavior?
6. When does computation-on-demand cost less than storing and reading data?
7. Can generators be designed around desired autocorrelation, spectrum, or spatial-frequency properties?
8. What practical limits are imposed by information theory?
9. Which deterministic test patterns best expose specific fault classes?
10. Which observable features make two fault signatures distinguishable?
11. How can a test be selected to maximally reduce uncertainty among competing fault hypotheses?

## 13. Starting references and keywords

- Alan V. Oppenheim and Ronald W. Schafer — *Discrete-Time Signal Processing*.
- Simon Haykin — *Communication Systems*.
- Thomas M. Cover and Joy A. Thomas — *Elements of Information Theory*.
- Steven H. Strogatz — *Nonlinear Dynamics and Chaos*.
- Kenneth H. Rosen — *Discrete Mathematics and Its Applications*.

Keywords: waveform design, pseudorandom sequences, LFSR, finite fields, recurrence relations, cellular automata, procedural generation, fractal geometry, fixed-point DSP, rate-distortion theory, algorithmic complexity, fault signature analysis, model-based diagnosis, fault detection and isolation, test-pattern generation.

## Project principle

**Start with the mathematical model, not the RTL.**

For every experiment:

1. Define the desired output properties.
2. Derive or select the mathematical relationship.
3. Build a high-precision software reference.
4. Determine state and parameter requirements.
5. Choose fixed-point formats and analyze numerical error.
6. Design the streaming hardware architecture.
7. Verify equivalence and measure FPGA cost.

For diagnostic experiments, additionally define the fault model, design a discriminating stimulus, compare observed and expected responses, and test whether the resulting signature isolates the fault.

The goal is not merely to generate an interesting picture. It is to understand the relationship between mathematical description, information content, computational complexity, memory requirements, efficient digital hardware, and the use of controlled patterns to understand system behavior.
