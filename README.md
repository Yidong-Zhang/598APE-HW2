# 598APE-HW2

## Optimized Artifact Guide

This guide documents our six cumulative performance experiments and a later
memory-leak fix. The detailed commands, original results, profiling evidence,
failed attempts, and limitations are recorded in [EXPERIMENT_LOG.md](EXPERIMENT_LOG.md).
The inherited project description and examples follow this guide.

Repository: https://github.com/Yidong-Zhang/598APE-HW2

```bash
git clone https://github.com/Yidong-Zhang/598APE-HW2.git
cd 598APE-HW2
```

### Dependencies and Tested Environment

Required for building/running: GNU make, a C++17 compiler, and OpenMP support
for optimized parallel builds. GNU `time` provides external process statistics.
Racket with `racket/draw` renders the generated image programs. `perf` and
Valgrind are diagnostic tools, not required for an ordinary benchmark run.

All experiments were conducted on the course-provided VM. Tool versions and
detailed environment records are available in [EXPERIMENT_LOG.md](EXPERIMENT_LOG.md).

On an Ubuntu VM with sudo access, basic dependencies can be installed with:

```bash
sudo apt update
sudo apt install build-essential time racket valgrind
```

The course VM already supplied perf. For profiling elsewhere, install a perf
package compatible with that machine's kernel; first check `perf --version`.
The supplied Docker scripts are an alternative setup, but these measurements
were made directly on the VM, not inside Docker.

### Versions and Compiler Flags


| Version | Additional change | Build flags | Runtime threads |
| --- | --- | --- | --- |
| Baseline | Original source | `-std=c++17 -Wall -Wextra -I include -I.` | Serial |
| OPT-001 | Compiler optimization | Above plus `-O2` | Serial |
| OPT-002 | Direct-index stack push/pop | Same as OPT-001 | Serial |
| OPT-003 | Parallel candidate evaluation, dynamic scheduling | Same as OPT-002 plus `-fopenmp` | 4 |
| OPT-004 | Stable pointer sorting of final candidates | Same as OPT-003 | 4 |
| OPT-005 | Replace `-O2` with `-O3` | `-std=c++17 -O3 -fopenmp -Wall -Wextra -I include -I.` | 4 |
| OPT-006 | Parallel log-loss error generation, static scheduling | Same as OPT-005 | 4 |
| FIX-001 | Release previous hoist-mutation output array | Same as OPT-006; add `-g` for diagnostics | 4; 1 for diagnostic control |

OPT-006 directly affects cancer, the only provided workload using log loss.
The other four use MSE and are controls for this change. No `-ffast-math` or
processor-specific instruction flags were used. Setting `OMP_NUM_THREADS=4`
does not make the serial source stages parallel.

The Makefile defaults do **not** enable `-O3` or OpenMP. Pass the documented
flags explicitly. Always run `make clean` when changing flags or headers:
the current Makefile does not track those dependencies. `make -j4` controls
compiler jobs, not the benchmark's runtime thread count.

### Build the Original Baseline

Start with the original assignment source before our edits. Download it from
[the original revision on GitHub](https://github.com/Yidong-Zhang/598APE-HW2/tree/f19cc633664c9a333b208049770424d58d14909e).
The current main branch contains the final changes; compiling main without
optimization flags does not restore the original source.

In the original source folder, build the baseline and run all five tasks using
the test commands below. Keep the results under the baseline label.

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -Wall -Wextra -I include -I.'
```

#### Baseline Tests

Our original measurements are stored directly under `results/`.
The reproduction commands below instead write to `results/reproduction/`
to avoid overwriting those saved records. Only the output location differs;
this does not change the benchmark computation.

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
/usr/bin/time -v \
  -o results/reproduction/baseline-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/baseline-diabetes-run1.rkt \
  2> results/reproduction/baseline-diabetes-run1.stderr.log
```

Cancer:

```bash
/usr/bin/time -v \
  -o results/reproduction/baseline-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/baseline-cancer-run1.rkt \
  2> results/reproduction/baseline-cancer-run1.stderr.log
```

Housing:

```bash
/usr/bin/time -v \
  -o results/reproduction/baseline-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/baseline-housing-run1.rkt \
  2> results/reproduction/baseline-housing-run1.stderr.log
```

Silk-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/baseline-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/baseline-silk-wave-run1.rkt \
  2> results/reproduction/baseline-silk-wave-run1.stderr.log
```

Square-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/baseline-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/baseline-square-wave-run1.rkt \
  2> results/reproduction/baseline-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/baseline-silk-wave-run1.rkt results/reproduction/baseline-silk-wave-run1.png
racket results/reproduction/baseline-square-wave-run1.rkt results/reproduction/baseline-square-wave-run1.png
```

### Apply Optimizations in Experiment Order

Starting from the baseline source above, apply the following changes in order.
Keep each change for the next step. Clean and rebuild after each stage, run
all five tasks, and save results under that stage's name. This follows our
actual experiment sequence; separate stage commits are not required.

1. **OPT-001:** source unchanged; enable `-O2`:

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O2 -Wall -Wextra -I include -I.'
```

#### OPT-001 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt01-O2-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt01-O2-diabetes-run1.rkt \
  2> results/reproduction/opt01-O2-diabetes-run1.stderr.log
```

Cancer:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt01-O2-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt01-O2-cancer-run1.rkt \
  2> results/reproduction/opt01-O2-cancer-run1.stderr.log
```

Housing:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt01-O2-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt01-O2-housing-run1.rkt \
  2> results/reproduction/opt01-O2-housing-run1.stderr.log
```

Silk-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt01-O2-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt01-O2-silk-wave-run1.rkt \
  2> results/reproduction/opt01-O2-silk-wave-run1.stderr.log
```

Square-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt01-O2-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt01-O2-square-wave-run1.rkt \
  2> results/reproduction/opt01-O2-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt01-O2-silk-wave-run1.rkt results/reproduction/opt01-O2-silk-wave-run1.png
racket results/reproduction/opt01-O2-square-wave-run1.rkt results/reproduction/opt01-O2-square-wave-run1.png
```

2. **OPT-002:** in `src/reg_stack.h`, replace only `push` and `pop` with:

```cpp
void push(DataT val) {
  if (elements_ < MaxSize) {
    regs_[elements_] = val;
    ++elements_;
  }
}

DataT pop() {
  if (elements_ > 0) {
    --elements_;
    return regs_[elements_];
  }
  return DataT(0);
}
```

Clean and rebuild with the OPT-001 command, retaining `-O2`.

#### OPT-002 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt02-stack-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt02-stack-diabetes-run1.rkt \
  2> results/reproduction/opt02-stack-diabetes-run1.stderr.log
```

Cancer:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt02-stack-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt02-stack-cancer-run1.rkt \
  2> results/reproduction/opt02-stack-cancer-run1.stderr.log
```

Housing:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt02-stack-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt02-stack-housing-run1.rkt \
  2> results/reproduction/opt02-stack-housing-run1.stderr.log
```

Silk-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt02-stack-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt02-stack-silk-wave-run1.rkt \
  2> results/reproduction/opt02-stack-silk-wave-run1.stderr.log
```

Square-wave:

```bash
/usr/bin/time -v \
  -o results/reproduction/opt02-stack-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt02-stack-square-wave-run1.rkt \
  2> results/reproduction/opt02-stack-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt02-stack-silk-wave-run1.rkt results/reproduction/opt02-stack-silk-wave-run1.png
racket results/reproduction/opt02-stack-square-wave-run1.rkt results/reproduction/opt02-stack-square-wave-run1.png
```

3. **OPT-003:** in `src/program.cpp`, add the following pragma immediately
   before `execute_kernel`'s outer `for (uint64_t pid = 0; pid < n_progs; ++pid)`
   loop. Build with `-O2 -fopenmp` and the common flags from the table.

```cpp
#pragma omp parallel for schedule(dynamic, 1)
```

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O2 -fopenmp -Wall -Wextra -I include -I.'
```

#### OPT-003 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt03-openmp4-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt03-openmp4-diabetes-run1.rkt \
  2> results/reproduction/opt03-openmp4-diabetes-run1.stderr.log
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt03-openmp4-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt03-openmp4-cancer-run1.rkt \
  2> results/reproduction/opt03-openmp4-cancer-run1.stderr.log
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt03-openmp4-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt03-openmp4-housing-run1.rkt \
  2> results/reproduction/opt03-openmp4-housing-run1.stderr.log
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt03-openmp4-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt03-openmp4-silk-wave-run1.rkt \
  2> results/reproduction/opt03-openmp4-silk-wave-run1.stderr.log
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt03-openmp4-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt03-openmp4-square-wave-run1.rkt \
  2> results/reproduction/opt03-openmp4-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt03-openmp4-silk-wave-run1.rkt results/reproduction/opt03-openmp4-silk-wave-run1.png
racket results/reproduction/opt03-openmp4-square-wave-run1.rkt results/reproduction/opt03-openmp4-square-wave-run1.png
```

4. **OPT-004:** in `benchmark/genetic_benchmark.cpp`, replace the original
   `insertionSortPrograms` definition with these two functions. The existing
   includes already provide the required library facilities. Retain the
   function name and call sites.

```cpp
bool compareFitness(const genetic::program *a,
                    const genetic::program *b) {
  if (a->raw_fitness_ < b->raw_fitness_) {
    return true;
  } else {
    return false;
  }
}

void insertionSortPrograms(genetic::program *programs, int size) {
  std::vector<genetic::program *> order(size);
  for (int i = 0; i < size; i++) {
    if (std::isnan(programs[i].raw_fitness_)) {
      programs[i].raw_fitness_ = std::numeric_limits<float>::max();
    }
    order[i] = &programs[i];
  }
  std::stable_sort(order.begin(), order.end(), compareFitness);
  std::vector<genetic::program> sorted(size);
  for (int i = 0; i < size; i++) {
    sorted[i] = *order[i];
  }
  for (int i = 0; i < size; i++) {
    programs[i] = sorted[i];
  }
}
```

Pointer sorting preserves the original handling of NaNs and stable ties.
It avoids the failed direct-object sorting attempt, whose library operations
were incompatible with this class's explicit copy constructor. Two linear
deep-copy passes remain.

Clean and rebuild with the OPT-003 command, keeping `-O2 -fopenmp`.

#### OPT-004 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt04-stablesort-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt04-stablesort-diabetes-run1.rkt \
  2> results/reproduction/opt04-stablesort-diabetes-run1.stderr.log
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt04-stablesort-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt04-stablesort-cancer-run1.rkt \
  2> results/reproduction/opt04-stablesort-cancer-run1.stderr.log
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt04-stablesort-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt04-stablesort-housing-run1.rkt \
  2> results/reproduction/opt04-stablesort-housing-run1.stderr.log
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt04-stablesort-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt04-stablesort-silk-wave-run1.rkt \
  2> results/reproduction/opt04-stablesort-silk-wave-run1.stderr.log
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt04-stablesort-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt04-stablesort-square-wave-run1.rkt \
  2> results/reproduction/opt04-stablesort-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt04-stablesort-silk-wave-run1.rkt results/reproduction/opt04-stablesort-silk-wave-run1.png
racket results/reproduction/opt04-stablesort-square-wave-run1.rkt results/reproduction/opt04-stablesort-square-wave-run1.png
```

5. **OPT-005:** no source edit; clean rebuild the OPT-004 source with `-O3`
   instead of `-O2`, retaining `-fopenmp`.

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O3 -fopenmp -Wall -Wextra -I include -I.'
```

#### OPT-005 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt05-O3-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt05-O3-diabetes-run1.rkt \
  2> results/reproduction/opt05-O3-diabetes-run1.stderr.log
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt05-O3-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt05-O3-cancer-run1.rkt \
  2> results/reproduction/opt05-O3-cancer-run1.stderr.log
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt05-O3-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt05-O3-housing-run1.rkt \
  2> results/reproduction/opt05-O3-housing-run1.stderr.log
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt05-O3-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt05-O3-silk-wave-run1.rkt \
  2> results/reproduction/opt05-O3-silk-wave-run1.stderr.log
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt05-O3-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt05-O3-square-wave-run1.rkt \
  2> results/reproduction/opt05-O3-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt05-O3-silk-wave-run1.rkt results/reproduction/opt05-O3-silk-wave-run1.png
racket results/reproduction/opt05-O3-square-wave-run1.rkt results/reproduction/opt05-O3-square-wave-run1.png
```

6. **OPT-006:** in `include/fitness.h`, add the following pragma immediately
   before the **first outer pid loop inside `logLoss`**, which fills `error`.
   Leave the weight sum and later averaging loop unchanged. Clean rebuild
   with `-O3 -fopenmp` and the common flags.

```cpp
#pragma omp parallel for schedule(static)
```

Clean and rebuild with the OPT-005 command. Test all five tasks before applying
the later memory fix below.

#### OPT-006 Tests

After compiling this stage, run each command below from its source folder.
Outputs go to `results/reproduction/` to preserve the original measurements.
Use another run number for repeat measurements; rerunning these commands
with the same paths overwrites that reproduction run.

```bash
mkdir -p results/reproduction
```

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt06-logloss4-diabetes-run1.time.log \
  ./genetic_benchmark diabetes \
  > results/reproduction/opt06-logloss4-diabetes-run1.rkt \
  2> results/reproduction/opt06-logloss4-diabetes-run1.stderr.log
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt06-logloss4-cancer-run1.time.log \
  ./genetic_benchmark cancer \
  > results/reproduction/opt06-logloss4-cancer-run1.rkt \
  2> results/reproduction/opt06-logloss4-cancer-run1.stderr.log
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt06-logloss4-housing-run1.time.log \
  ./genetic_benchmark housing \
  > results/reproduction/opt06-logloss4-housing-run1.rkt \
  2> results/reproduction/opt06-logloss4-housing-run1.stderr.log
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt06-logloss4-silk-wave-run1.time.log \
  ./genetic_benchmark silk-wave \
  > results/reproduction/opt06-logloss4-silk-wave-run1.rkt \
  2> results/reproduction/opt06-logloss4-silk-wave-run1.stderr.log
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  -o results/reproduction/opt06-logloss4-square-wave-run1.time.log \
  ./genetic_benchmark square-wave \
  > results/reproduction/opt06-logloss4-square-wave-run1.rkt \
  2> results/reproduction/opt06-logloss4-square-wave-run1.stderr.log
```

Render the saved image formulas after timing:

```bash
racket results/reproduction/opt06-logloss4-silk-wave-run1.rkt results/reproduction/opt06-logloss4-silk-wave-run1.png
racket results/reproduction/opt06-logloss4-square-wave-run1.rkt results/reproduction/opt06-logloss4-square-wave-run1.png
```

### Later Memory Fix and Current Build

To reproduce FIX-001 separately, add `delete[] p_out.nodes;` immediately before
`p_out.nodes = new node[p_out.len];` **inside `hoist_mutation`** in
`src/program.cpp`. Do not add a duplicate deletion in `crossover`. Current
training calls use distinct input/output programs. The old output allocation
must be released before its pointer is replaced.

Build the current fixed source with:

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O3 -fopenmp -Wall -Wextra -I include -I.'
```

The performance table below was collected before FIX-001. No ordinary post-fix
timing has been recorded; memory-check timings are not performance results.

### Run and Time All Five Workloads

Use a new directory name for each configuration/run to avoid overwriting saved
evidence. This example measures the fixed current build; change the directory
label when measuring a reconstructed historical stage. No dataset download is
needed; inputs are in `benchmark/`.

```bash
RESULT_DIR=results/reproduction-fixed-final-run1
mkdir -p "$RESULT_DIR"
for task in diabetes cancer housing silk-wave square-wave; do
  OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
    -o "$RESULT_DIR/$task.time.log" \
    ./genetic_benchmark "$task" \
    > "$RESULT_DIR/$task.rkt" \
    2> "$RESULT_DIR/$task.stderr.log"
done
```

Each invocation performs a complete task. Tabular tasks print the two selected
formulas and quality metrics; image tasks print a runnable reconstruction
program. Time statistics and stderr are separate to keep Racket source valid.
Inspect each `.time.log` (including exit status), `.stderr.log`, and the final
`;;Time(...)` line in `.rkt`. Run benchmarks sequentially, without perf or
Valgrind. Repeat into distinct run directories when estimating variability.

Render images afterward; rendering is not part of the measured C++ interval:

```bash
racket "$RESULT_DIR/silk-wave.rkt" "$RESULT_DIR/silk-wave.png"
racket "$RESULT_DIR/square-wave.rkt" "$RESULT_DIR/square-wave.png"
```

Workload settings are unchanged across stages: diabetes 442 samples/10 features,
cancer 569/30, housing 20640/8; images are 256x256 pixels with two coordinate
features. Population sizes are respectively 512, 16384, 512, 128, 128;
all configure 16 generations. Silk-wave trains nine RGB models, square-wave 24.
The deterministic split uses approximately 80% training and 20% testing.
Seeds remain as in the original source; no reduced training budget is used.

### Recorded Performance and Output Checks

The following are internal elapsed seconds from **one run per task/version**,
not averages. Baseline is the original build without an optimization flag.
Final measured version is OPT-006 **before the leak fix**. Complete intermediate
results are in the experiment log and `results/opt*-run1*` files.

| Workload | Original baseline | OPT-001 (-O2 only) | OPT-006 (pre-fix) |
| --- | --- | --- | --- |
| diabetes | 21.419001955 | 10.154784505 | 0.398813292 |
| cancer | 134.527396735 | 60.837659981 | 3.778010087 |
| housing | 251.455379922 | 118.189752884 | 5.851969198 |
| silk-wave | 422.754671735 | 172.247363277 | 12.842426638 |
| square-wave | 1312.860383860 | 547.494688596 | 37.867636479 |

Internal timing covers the benchmark-defined interval including loading,
preparation, training, selection and test/formula output. It excludes compilation,
Racket rendering and, for tabular tasks, final array cleanup after the timer.
External process wall times are retained separately and are not mixed into
reported internal-timer speedup ratios. VM contention and scheduling can affect
results; small single-run differences are not established benefits/regressions.

All six optimization stages retained the full printed output before timing,
including formulas, model sizes and quality metrics. Reference test MSE values:
diabetes 7468.95/7217.17, housing 0.64586/0.728984; cancer accuracy is 0.946903
for both models. Image RGB test MSE before display clamping is 0.0277195631
(silk-wave) and 0.0175210362 (square-wave). All stage PNGs were byte-identical
to baseline. These checks do not prove bitwise equality of internal predictions.

To compare a new diabetes output with its recorded reference, ignoring timing:

```bash
diff -u \
  <(sed '/^;;Time(/,$d' results/opt06-logloss4-diabetes-run1.log) \
  <(sed '/^;;Time(/,$d' "$RESULT_DIR/diabetes.rkt")
cmp results/baseline-silk-wave-run1.png "$RESULT_DIR/silk-wave.png"
cmp results/baseline-square-wave-run1.png "$RESULT_DIR/square-wave.png"
```

No output with exit status 0 means equality. A difference requires investigation;
cross-platform floating-point behavior or PNG encodings may differ. Repeat the
text comparison for other tasks, using their corresponding saved source/log.

### Profiling

Profiles are separate diagnostic runs. For example, sample the fixed current
housing workload on a VM where sudo permits perf:

```bash
sudo env OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE perf record \
  -e cpu-clock:u -F 99 -g --call-graph dwarf \
  -o results/perf-fixed-housing.data \
  -- ./genetic_benchmark housing > results/perf-fixed-housing.rkt
sudo perf report --stdio --no-children --sort symbol --percent-limit 1 \
  -i results/perf-fixed-housing.data > results/perf-fixed-housing.report.txt
```

Our VM reported `perf_event_paranoid=4`; sudo was used without changing that
setting. The software event avoids depending on virtualized hardware counters.
Self shares describe aggregate user-space CPU samples across threads, not exact
wall-time fractions. Inlining changes symbol attribution; do not double-count
caller-tree shares. Historical reports and the reasons for changing profiled
workloads are listed under PROF-001 through PROF-006 in the experiment log.


### Memory Validation and FIX-001

FIX-001 releases the previous node array in `hoist_mutation`. A diagnostic
build retains `-O3`/OpenMP and adds `-g` for source locations:

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O3 -g -fopenmp -Wall -Wextra -I include -I.'
mkdir -p results/memcheck-reproduction
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE valgrind \
  --tool=memcheck --leak-check=full --show-leak-kinds=all \
  --track-origins=yes --error-exitcode=99 \
  --log-file=results/memcheck-reproduction/diabetes.log \
  ./genetic_benchmark diabetes \
  > results/memcheck-reproduction/diabetes.rkt \
  2> results/memcheck-reproduction/diabetes.stderr.log
echo "Memcheck exit status: $?"
```

Use corresponding `cancer` names and argument for classification coverage.
An `OMP_NUM_THREADS=1` run with distinct output names provides a diagnostic
control. Valgrind runs are much slower and their internal timings must not be
used in performance comparisons. Memcheck does not check data races.

| Observed check | Definitely lost | Possibly lost | Still reachable | Error contexts |
| --- | --- | --- | --- | --- |
| Diabetes pre-fix, 4 threads | 58776 bytes | 1008 bytes | 2608 bytes | 6 |
| Diabetes FIX-001, 4 threads | 0 | 1008 bytes | 2608 bytes | 1 |
| Diabetes FIX-001, 1 thread | 0 | 0 | 296 bytes | 0 |
| Cancer FIX-001, 4 threads | 0 | 1008 bytes | 2608 bytes | 1 |

Indirect loss was zero in all four reports. Four-thread residual possible-loss
stacks enter thread TLS allocation (`allocate_dtv`, `_dl_allocate_tls`,
`pthread_create`, libgomp). The diabetes one-thread control removes this context,
supporting association with worker TLS, not proving a benign false positive.
Reachable records originate in libgomp. No suppressions were applied. The
four-thread reports are **not zero-error reports**; `--error-exitcode=99` can
produce a nonzero command marker even without a program crash.



## Original Project Guide

The following is the supplied project guide. Its plain `make -j` command uses
the Makefile defaults; use the explicit optimized flags above for our artifact.
Its example timing is illustrative, not a measurement from our VM.

This repository contains code for homework 2 of 598APE.

In particular, this repository is an implementation of Symbolic Regression and the various contexts it can be used in.

To compile the program run:
```bash
make -j
```

To clean existing build artifacts run:
```bash
make clean
```

This program assumes the following are installed on your machine:
* A working C++ compiler (g++ is assumed in the Makefile)
* make
* racket (to plot images)

Running make builds a sample benchmarking binary(`genetic_benchmark`) which you will use for perf engineering. However, note that the symbolic regression routines exposed in `genetic.h` are general, and can be used as a sub-routine in any data science pipeline (pending a Cython wrapper to Python/Torch).

Once compiled, one can run the benchmark code on the diabetes dataset as follows. The output is a valid racket program, which provides the 2 best programs as callable functions - symbolic regression is used for synthesizing these functions. 

```bash
./genetic_benchmark diabetes
#lang racket

;;===== Symbolic Regression Benchmark =====

(provide best-program-1 best-program-2)
;;Loading dataset...
;;Dataset dimensions: 442 samples x 10 features
;;Training symbolic regressor with 512 population size and 16 generations
;;Best program 1 details:
;;- Length: 91 nodes
;;- Depth: 12
;;- Raw fitness: 6302.75
;;- Test MSE: 7468.95
(define best-program-1 (lambda (X)
  (define (div a b) (if (< (abs b) 0.001) 1.0 (/ a b)))
  (define (fdim a b) (max (- a b) 0.0))
  (define (gt a b) (if (> a b) 1.0 0.0))
  (define (acosh a)
    (log (+ a (* (sqrt (- a 1.0)) (sqrt (+ a 1.0))))))
  (define (asinh a)
    (let* ([v (abs a)] [r (log (+ v (sqrt (+ (* v v) 1.0))))])
      (if (< a 0.0) (- r) r)))
  (define (atanh a) (* 0.5 (log (/ (+ 1.0 a) (- 1.0 a)))))
  (define (cbrt a)
    (let ([r (expt (abs a) (/ 1.0 3.0))])
      (if (< a 0.0) (- r) r)))
  (define (cube a) (* a a a))
  (define (inv a) (if (< (abs a) 0.001) 0.0 (/ 1.0 a)))
  (define (rcbrt a) (/ 1.0 (cbrt a)))
  (define (rsqrt a) (/ 1.0 (sqrt (abs a))))
  (define (gp-log a) (if (< (abs a) 0.001) 0.0 (log (abs a))))
  (define (gp-sqrt a) (sqrt (abs a)))
  (*  (-  (-  (+  (cos  (+  (cos  (-  (gp-log  (-  (list-ref X 0) (abs  (list-ref X 8)) ) )  (-  (-  (list-ref X 1) (list-ref X 8))  (sin  (list-ref X 7)) ) ) )  (sin  (cos  (cos  (*  (sin  (cos  (list-ref X 8)) )  (gp-log  (-  (list-ref X 6) (list-ref X 5)) ) ) ) ) ) ) )  (cos  (*  (list-ref X 7) (list-ref X 7)) ) )  (*  (sin  (cos  (list-ref X 8)) )  (gp-log  (-  (list-ref X 6) (list-ref X 5)) ) ) )  (-  (-  (cos  (fdim  (list-ref X 2) (list-ref X 9)) )  (-  (cos  (fdim  (list-ref X 8) (list-ref X 6)) )  (gp-log  (list-ref X 0)) ) )  (*  -0.960798442 (list-ref X 5)) ) )  (+  (fdim  (-  (sin  (cos  (+  (list-ref X 5) (list-ref X 0)) ) )  (cos  (cos  (list-ref X 4)) ) )  (+  (gp-log  (list-ref X 1))  (cos  (gp-log  (list-ref X 9)) ) ) )  (+  (-  (fdim  (abs  (list-ref X 3))  (gp-log  (*  (list-ref X 1) (list-ref X 1)) ) )  (gp-log  (*  (list-ref X 1) (list-ref X 1)) ) )  (abs  (cos  (list-ref X 6)) ) ) ) ) ))
;;Best program 2 details:
;;- Length: 199 nodes
;;- Depth: 18
;;- Raw fitness: 6408.27
;;- Test MSE: 7217.17
(define best-program-2 (lambda (X)
  (define (div a b) (if (< (abs b) 0.001) 1.0 (/ a b)))
  (define (fdim a b) (max (- a b) 0.0))
  (define (gt a b) (if (> a b) 1.0 0.0))
  (define (acosh a)
    (log (+ a (* (sqrt (- a 1.0)) (sqrt (+ a 1.0))))))
  (define (asinh a)
    (let* ([v (abs a)] [r (log (+ v (sqrt (+ (* v v) 1.0))))])
      (if (< a 0.0) (- r) r)))
  (define (atanh a) (* 0.5 (log (/ (+ 1.0 a) (- 1.0 a)))))
  (define (cbrt a)
    (let ([r (expt (abs a) (/ 1.0 3.0))])
      (if (< a 0.0) (- r) r)))
  (define (cube a) (* a a a))
  (define (inv a) (if (< (abs a) 0.001) 0.0 (/ 1.0 a)))
  (define (rcbrt a) (/ 1.0 (cbrt a)))
  (define (rsqrt a) (/ 1.0 (sqrt (abs a))))
  (define (gp-log a) (if (< (abs a) 0.001) 0.0 (log (abs a))))
  (define (gp-sqrt a) (sqrt (abs a)))
  (*  (-  (-  (+  (cos  (*  (list-ref X 7) (list-ref X 7)) )  (cos  (*  (list-ref X 7) (list-ref X 7)) ) )  (*  (sin  (cos  (list-ref X 8)) )  (gp-log  (-  (list-ref X 6) (list-ref X 5)) ) ) )  (-  (-  (gp-log  (list-ref X 6))  (-  (sin  (list-ref X 5))  (gp-log  (list-ref X 0)) ) )  (sin  (sin  (+  (-  (-  (-  (+  (+  (cos  (list-ref X 6))  (fdim  (list-ref X 4) (list-ref X 0)) )  (abs  (gp-log  (list-ref X 3)) ) )  (*  (gp-log  (+  (list-ref X 4) (list-ref X 8)) )  (cos  (sin  (fdim  (-  (list-ref X 0) (list-ref X 9))  (abs  (list-ref X 8)) ) ) ) ) )  (-  (-  (cos  (fdim  (list-ref X 2) (list-ref X 9)) )  (-  (*  (gp-log  (*  (*  (abs  (-  (abs  (list-ref X 8))  (+  (list-ref X 5) (list-ref X 5)) ) )  (sin  (list-ref X 1)) )  (cos  (sin  (list-ref X 1)) ) ) )  (-  (+  (cos  (cos  (list-ref X 0)) )  (+  (fdim  (list-ref X 2) (list-ref X 9))  (abs  (list-ref X 1)) ) )  (cos  (cos  (gp-log  (abs  (-  (list-ref X 5) (list-ref X 7)) ) ) ) ) ) )  (-  (sin  (list-ref X 5))  (gp-log  (list-ref X 0)) ) ) )  (abs  (+  (list-ref X 2) (list-ref X 9)) ) ) )  (abs  (-  (list-ref X 6) (gp-log  (list-ref X 7)) ) ) )  (list-ref X 9)) ) ) ) )  (+  (fdim  (-  (sin  (-  (-  (+  (cos  (+  (cos  (-  (gp-log  (-  (list-ref X 0) (abs  (list-ref X 8)) ) )  (-  (-  (list-ref X 1) (list-ref X 8))  (sin  (list-ref X 7)) ) ) )  (sin  (cos  (cos  (sin  (list-ref X 5)) ) ) ) ) )  (cos  (*  (list-ref X 7) (list-ref X 7)) ) )  (*  (sin  (cos  (list-ref X 8)) )  (gp-log  (-  (list-ref X 6) (list-ref X 5)) ) ) )  (-  (-  (cos  (fdim  (list-ref X 2) (list-ref X 9)) )  (-  (cos  (fdim  (list-ref X 8) (list-ref X 6)) )  (gp-log  (list-ref X 0)) ) )  (*  -0.960798442 (list-ref X 5)) ) ) )  (cos  (+  (+  (cos  (list-ref X 6))  (list-ref X 9))  (abs  (gp-log  (list-ref X 3)) ) ) ) )  (+  (gp-log  (list-ref X 1))  (cos  (gp-log  (list-ref X 9)) ) ) )  (fdim  (+  (+  (cos  (list-ref X 6))  (*  (list-ref X 7) (list-ref X 7)) )  (abs  (gp-log  (list-ref X 3)) ) )  (gp-log  (-  (list-ref X 6) (list-ref X 5)) ) ) ) ) ))
;;Time(Symbolic Regression (End-to-End)) = 11.512300000 sec

```

`(list-ref X i)` corresponds to the `i`th feature of your dataset (it is Racket syntax to access `X[i]`)

While currently hardcoded to only handle the 3 CSV datasets and 2 image reconstruction tasks, `genetic_benchmark.cpp` in the `benchmarks` folder exposes training code for end to end symbolic regression/binary classification, starting from converting the input data to a columnar format, before dispatching it to `genetic::symFit`. 

The behaviour of the training run can be tuned through the `param` struct, which controls paramenters like the population size, functions being considered, subtree depth and so on.  

```bash
./genetic_benchmark diabetes
```

There are also 2 image reconstruction tasks. The goal of image re-construction is to infer the underlying semantic meaning present in an image. We do this by representing every image as the average of `K` mathematical equations, and output the equations as a valid racket program (which you can also use to draw the final image). All input images have 256x256 pixels, and each pixel stores a r,g,b tuple corresponding to red, green and blue pixel values. For example, on the silk-wave input:

![Silk wave](./benchmark/silk-waves.png)

We now run symbolic regression based reconstruction on the silk-wave picture. 

```bash
./genetic_benchmark silk-wave > silk-ref.rkt
racket silk-ref.rkt silk-waves-reconstructed.png
```

This produces 2 artifacts. [silk-ref.rkt](./benchmark/silk-ref.rkt) contains the symbolic definition of the image. The reconstructed image is 

![Reconstructed Silk wave](./benchmark/silk-waves-reconstructed.png)

Similarly, for the harder square wave benchmark, we start from 

![Square wave](./benchmark/squared-waves.png)

and are able to reconstruct 

![Reconstructed square wave](./benchmark/square-wave-reconstructed.png)

with the source code for the reconstructed wave in [square-wave-ref.rkt](./benchmark/square-wave-ref.rkt)

The time taken for the end to end run can be seen in the last comment of the generated rkt file. Similar to HW1, your goal is to reduce this runtime as much as possible, while maintining/increasing complexity. For the image code, it is sufficient to just optimize image generation, however, you are also free to optimize image drawing too (which is currently implemnented as a DSL in Racket)

## Input Datasets and Images
This project contains 3 input datasets and 2 images for you to optimize. All 3 are standard scikit-learn datasets. The images are generated bitmaps from `benchmark/generate-easy-image.rkt`.

### Diabetes

```bash
./genetic_benchmark diabetes
```

### Cancer

```bash
./genetic_benchmark cancer
```

### California Housing Prices

```bash
./genetic_benchmark housing
```

### Silk waves

```bash
./genetic_benchmark silk-wave > silk.rkt
racket silk.rkt silk.png
```

### Square waves

```bash
./genetic_benchmark square-wave > square.rkt
racket square.rkt square.png
```

## Code Overview

There are multiple core functions

`cpp_evolve`

`cpp_evolve` is the central evolution driver. It takes a population of candidate programs and produces a new generation by first selecting parents through tournament selection and then applying various mutation strategies (crossover, subtree mutation, hoist mutation, and point mutation) or direct reproduction. After mutation, it updates the fitness of the new generation.

`build_program`

`build_program` constructs an initial candidate program (represented as a syntax tree) using a randomized process. It chooses function nodes and terminal nodes based on specified initialization methods (grow, full, or half-and-half) and ensures that the generated program adheres to a maximum depth constraint.

`tournament_kernel` 

`tournament_kernel` implements the tournament selection process. It randomly samples subsets of the population, evaluates their fitness (including a parsimony penalty for program length), and selects the best candidates to serve as parents for the next generation.

`execute_kernel`

Given a bunch of programs(symbolic formulae), `execute_kernel` evaluates those formulae on the given input dataset and outputs predicted values. We evaluate all programs on all dataset rows. 

`compute_metric`

As the name suggests, we compute the loss/accuracy values according to the metric specified in `params`. There are 5 different metrics supported - logLoss (for classification), mean square error and root mean square error(for regression) and correlation coefficients for transformation (both Karl Pearson and Spearman's rank correlation are implemented). 

## Docker

For ease of use and installation, we provide a docker image capable of running and building code here. The source docker file is in /docker (which is essentially a list of commands to build an OS state from scratch). It contains the dependent compilers, and some other nice things.

Run the docker build for the first time to setup all the environments you need
```bash
./dockerbuild.sh
```

If you now run `./dockerrun.sh` you will enter an interactive bash session with all the packages from docker installed (that script by default uses `vim/598ape`, feel free to replace it with whatever location you like if you built from scratch). The current directory (aka this folder) is mounted within `/host`. Any files you create on your personal machine will be available there, and anything you make in the container in that folder will be available on your personal machine.
