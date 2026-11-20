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

## 6. Mathematics and theory roadmap

Suggested learning order:

1. **Discrete mathematics:** functions, relations, modular arithmetic, recurrence relations, finite-state machines.
2. **Linear algebra:** vectors, matrices, linear transformations, eigenvalues/eigenvectors, state-space models.
3. **Number theory:** congruences, primes, modular inverses, polynomial arithmetic over finite fields.
4. **Probability and statistics:** random variables, distributions, expectation, variance, correlation, sequence tests.
5. **Signals and systems:** discrete-time systems, LTI systems, convolution, Fourier analysis, sampling, quantization.
6. **Information theory:** entropy, source coding, rate-distortion, redundancy, limits of lossless compression.
7. **Dynamical systems:** fixed points, stability, periodic orbits, nonlinear maps, bifurcations, chaos.
8. **Digital arithmetic and hardware:** fixed-point formats, quantization, overflow, modular arithmetic, resource sharing, pipelining, timing closure.

## 7. Proposed FPGA investigation

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

## 8. Evaluation methodology

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

## 9. Suggested repository structure

```text
.
├── README.md
├── theory/
│   ├── recurrence-relations.md
│   ├── lfsr-and-pn-sequences.md
│   ├── cellular-automata.md
│   ├── fractals-and-iterated-maps.md
│   └── information-theory.md
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

## 10. First milestones

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

## 11. Initial research questions

1. Which classes of images can be represented accurately by a compact mathematical model?
2. How should model parameters be distinguished from dynamic state?
3. What is the minimum state size for a given deterministic sequence?
4. Which generators can produce one output sample per clock?
5. How does finite word length change nonlinear system behavior?
6. When does computation-on-demand cost less than storing and reading data?
7. Can generators be designed around desired autocorrelation, spectrum, or spatial-frequency properties?
8. What practical limits are imposed by information theory?

## 12. Starting references and keywords

- Alan V. Oppenheim and Ronald W. Schafer — *Discrete-Time Signal Processing*.
- Simon Haykin — *Communication Systems*.
- Thomas M. Cover and Joy A. Thomas — *Elements of Information Theory*.
- Steven H. Strogatz — *Nonlinear Dynamics and Chaos*.
- Kenneth H. Rosen — *Discrete Mathematics and Its Applications*.

Keywords: waveform design, pseudorandom sequences, LFSR, finite fields, recurrence relations, cellular automata, procedural generation, fractal geometry, fixed-point DSP, rate-distortion theory, algorithmic complexity.

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

The goal is not merely to generate an interesting picture. It is to understand the relationship between mathematical description, information content, computational complexity, memory requirements, and efficient digital hardware.
