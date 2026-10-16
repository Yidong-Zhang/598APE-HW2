# 598APE-HW2

## Optimized Artifact Guide

This guide documents our six cumulative performance experiments and a later
memory-leak fix.
The inherited project description and examples follow this guide.

Repository: https://github.com/Yidong-Zhang/598APE-HW2

### Dependencies and Tested Environment

Required for building/running: GNU make, a C++17 compiler, and OpenMP support
for optimized parallel builds. GNU `time` provides external process statistics.
Racket with `racket/draw` renders the generated image programs. `perf` and
Valgrind are diagnostic tools, not required for an ordinary benchmark run.

All experiments were conducted on the course-provided VM.

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

Each optimization stage includes all preceding stages. FIX-001 is a separate
correctness fix added after the performance measurements.

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

Start with the original assignment source, before our edits. Download it from
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
/usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
/usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
/usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark silk-wave > baseline-silk-wave-run1.rkt
```

Square-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark square-wave > baseline-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket baseline-silk-wave-run1.rkt baseline-silk-wave-run1.png
racket baseline-square-wave-run1.rkt baseline-square-wave-run1.png
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
/usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
/usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
/usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt01-O2-silk-wave-run1.rkt
```

Square-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark square-wave > opt01-O2-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt01-O2-silk-wave-run1.rkt opt01-O2-silk-wave-run1.png
racket opt01-O2-square-wave-run1.rkt opt01-O2-square-wave-run1.png
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
/usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
/usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
/usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt02-stack-silk-wave-run1.rkt
```

Square-wave:

```bash
/usr/bin/time -v \
  ./genetic_benchmark square-wave > opt02-stack-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt02-stack-silk-wave-run1.rkt opt02-stack-silk-wave-run1.png
racket opt02-stack-square-wave-run1.rkt opt02-stack-square-wave-run1.png
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt03-openmp4-silk-wave-run1.rkt
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark square-wave > opt03-openmp4-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt03-openmp4-silk-wave-run1.rkt opt03-openmp4-silk-wave-run1.png
racket opt03-openmp4-square-wave-run1.rkt opt03-openmp4-square-wave-run1.png
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt04-stablesort-silk-wave-run1.rkt
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark square-wave > opt04-stablesort-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt04-stablesort-silk-wave-run1.rkt opt04-stablesort-silk-wave-run1.png
racket opt04-stablesort-square-wave-run1.rkt opt04-stablesort-square-wave-run1.png
```

5. **OPT-005:** no source edit; clean rebuild the OPT-004 source with `-O3`
   instead of `-O2`, retaining `-fopenmp`.

```bash
make clean
make -j4 CXXFLAGS='-std=c++17 -O3 -fopenmp -Wall -Wextra -I include -I.'
```

#### OPT-005 Tests

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt05-O3-silk-wave-run1.rkt
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark square-wave > opt05-O3-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt05-O3-silk-wave-run1.rkt opt05-O3-silk-wave-run1.png
racket opt05-O3-square-wave-run1.rkt opt05-O3-square-wave-run1.png
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

Run these commands from the project root after compiling this stage.
Repeat each workload five times and record the smallest internal `;;Time(...)`
value. Results, errors, and process statistics appear in the terminal, except
image source saved as `.rkt`. Change image run numbers from run1 to run5 to
preserve each run; render the fastest run's file afterward.

Diabetes:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark diabetes
```

Cancer:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark cancer
```

Housing:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark housing
```

Silk-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark silk-wave > opt06-logloss4-silk-wave-run1.rkt
```

Square-wave:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark square-wave > opt06-logloss4-square-wave-run1.rkt
```

Render the saved image formulas after timing:

```bash
racket opt06-logloss4-silk-wave-run1.rkt opt06-logloss4-silk-wave-run1.png
racket opt06-logloss4-square-wave-run1.rkt opt06-logloss4-square-wave-run1.png
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

This example runs the fixed current build. No dataset download or output
folder is needed; inputs are in `benchmark/`. Run commands sequentially from
the project root, without perf or Valgrind. Repeat each task five times and
record the minimum internal `;;Time(...)` value, not GNU time's process wall
time. The commands do not automatically save terminal output. For images,
change run1 to run2 through run5 to retain the generated source files.

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark diabetes
```

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark cancer
```

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark housing
```

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark silk-wave > fixed-silk-wave-run1.rkt
```

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE /usr/bin/time -v \
  ./genetic_benchmark square-wave > fixed-square-wave-run1.rkt
```

Render images afterward; rendering is not part of the measured C++ interval.
Use each image workload's fastest run number in both filenames below:

```bash
racket fixed-silk-wave-run1.rkt fixed-silk-wave-run1.png
racket fixed-square-wave-run1.rkt fixed-square-wave-run1.png
```

Workload settings are unchanged across stages: diabetes 442 samples/10 features,
cancer 569/30, housing 20640/8; images are 256x256 pixels with two coordinate
features. Population sizes are respectively 512, 16384, 512, 128, 128;
all configure 16 generations. Silk-wave trains nine RGB models, square-wave 24.
The deterministic split uses approximately 80% training and 20% testing.
Seeds remain as in the original source; no reduced training budget is used.

### Recorded Performance and Output Checks

Each workload was run **five times for the baseline and OPT-001 through OPT-006**. The following values are
the **minimum internal elapsed time across those five runs**, not averages.
The saved artifacts in `results/` correspond to the fastest run for each
workload/version; the retained `run1` filename suffix is an artifact label,
not an indication that only one run was performed. Baseline is the original
build without an optimization flag. Final measured version is OPT-006
**before the leak fix**. Intermediate best-run results are in the
`results/opt*-run1*` files.

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
results. Minimum-of-five comparisons describe best observed performance,
not typical runtime or variability; small differences alone do not establish
consistent benefits or regressions.

All six optimization stages retained the full printed output before timing,
including formulas, model sizes and quality metrics. Reference test MSE values:
diabetes 7468.95/7217.17, housing 0.64586/0.728984; cancer accuracy is 0.946903
for both models. Image RGB test MSE before display clamping is 0.0277195631
(silk-wave) and 0.0175210362 (square-wave). All stage PNGs were byte-identical
to baseline. These checks do not prove bitwise equality of internal predictions.

For an optional text comparison, save a diabetes output first. This additional
validation run is not part of timing selection. Compare image files rendered
above, substituting the selected run numbers as needed:

```bash
OMP_NUM_THREADS=4 OMP_DYNAMIC=FALSE ./genetic_benchmark diabetes > fixed-diabetes-check.rkt
diff -u \
  <(sed '/^;;Time(/,$d' results/opt06-logloss4-diabetes-run1.log) \
  <(sed '/^;;Time(/,$d' fixed-diabetes-check.rkt)
cmp results/baseline-silk-wave-run1.png fixed-silk-wave-run1.png
cmp results/baseline-square-wave-run1.png fixed-square-wave-run1.png
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
caller-tree shares. Historical text reports are retained in `results/`.
The commands above are reproduction examples, not additional recorded results.

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

Evidence: `results/memcheck-diabetes.log`,
`results/memcheck-fix01-diabetes.log`,
`results/memcheck-fix01-diabetes-1thread.log`, and
`results/memcheck-fix01-cancer.log`. VM diff comparisons confirmed post-fix
diabetes/cancer printed formulas and metrics match OPT-006 before timing.
Image-path Memcheck, race checks, and ordinary post-fix timings were not run.
The compiler's Pearson `y_mu` warning also remains unresolved; these five
benchmarks use MSE/log loss and do not validate Pearson behavior.

Commit source, documentation and diagnostic text outputs together. Keep
executable backups, object files and binary perf `.data` captures out of source
commits; textual perf reports suffice for inspecting the recorded findings.

