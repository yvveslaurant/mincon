# Intellectual-property audit

*Read-only audit of this repository, its full git history and the published
PyPI packages. Nothing was changed except this file. No MathWorks source code
was downloaded or read. This is an engineering audit, not legal advice: items
marked **needs lawyer** are questions for counsel, not conclusions.*

| | |
|---|---|
| Date | 2026-09-24 |
| Repository | `yvveslaurant/mincon`, a public fork of `atciamb/mincon` (the PyPI project links to the upstream) |
| Tree audited | `3f48bac` (branch `claude/busy-dijkstra-ed2hnp` = `master`), 1,145 tracked files |
| History audited | all 76 commits reachable from any ref, root `616cd5e` (2026-09-05) to `3f48bac` (2026-09-20). The session clone was shallow and was unshallowed first (`git fetch --unshallow`) |
| Published packages audited | PyPI `mincon` 0.1.0 (2 wheels + sdist) and 0.2.0 (3 wheels + sdist): all 7 files downloaded and SHA-256-checked against PyPI. No `mincon*` crate exists on crates.io |
| MathWorks reference | public documentation pages only, read as text: the `fmincon` reference page, *Constrained Nonlinear Optimization Algorithms*, *Optimization Options Reference*, *Exit Flags and Exit Messages*, *Tolerances and Stopping Criteria*, *Lagrange Multiplier Structures* |

Risk levels: **none** (no IP concern), **low** (minor, easy fix),
**needs lawyer** (the answer depends on facts or terms only counsel can weigh).

---

## Summary

**No MathWorks code, P-files, toolbox files, MAT or MEX files exist in the
current tree, in any of the 76 commits, or in any published PyPI file.**
Nothing suggests that `fmincon`'s implementation was read: there is no
P-code, no `edit fmincon`, and no debugger stepping. Where the docs describe
`fmincon`, they cite and paraphrase the public MathWorks pages. Every
algorithm component traces to published literature or is one of the project's
own heuristics. None is derived from MathWorks internals. Where MathWorks
documents a specific mechanism (BFGS damping, per-constraint merit weights, a
primal active-set QP), mincon implements a *different* mechanism taken from
the literature.

**One item needs a lawyer:** whether the MATLAB licence used for the
benchmarks allows using `fmincon` to develop and benchmark a competing product
and publishing the results (F-01). The rest are low-risk hygiene items, all
with easy fixes. The main ones:

- There is no trademark or non-affiliation disclaimer, even though the package
  exports a function named `fmincon`.
- Three committed MATLAB logs name non-public toolbox functions.
- The result records keep MathWorks' exit-message text verbatim.
- Apache-2.0 QDLDL code is adapted without keeping its notice, and the Apache
  licence text is only a stub.

### All findings, highest risk first

| ID | Risk | Finding | Where | Suggested fix |
|---|---|---|---|---|
| F-01 | **needs lawyer** | 3,672 `fmincon` result records and the comparative claims built on them were produced with MATLAB R2025b (Optimization Toolbox 25.2) on a Windows laptop, under a licence type the repository never states. 10 of the 76 commits are authored from a university (`.edu`) address, so an academic or student licence is possible but not established. Whether that licence allows (a) using `fmincon` to develop and benchmark a competing product and (b) publishing the results depends on its terms | 85 files in 25 `bench/results/*` directories; `README.md:33-92`; docs 15-23 (list in §6.5) | Establish which MATLAB licence was used and have counsel read its terms on benchmarking and publication before relying on these numbers in marketing |
| F-02 | low | **No trademark or non-affiliation disclaimer anywhere**: repo, both READMEs, PyPI metadata, all 7 PyPI files. "MathWorks" appears 0 times in the tree | whole tree + history; PyPI | Add to `README.md`, `crates/mincon-py/README.md` and the `fmincon` docstring: "MATLAB and fmincon are trademarks or registered trademarks of The MathWorks, Inc. mincon is not affiliated with, sponsored by or endorsed by The MathWorks, Inc." |
| F-03 | low | The MathWorks names are used in the package's identity and search metadata | `crates/mincon/Cargo.toml:3` ("a free, fast, pip-installable answer to MATLAB's fmincon", also shipped in every wheel's SBOM); `crates/mincon-py/pyproject.toml:7` ("MATLAB-style interfaces"), `:12` (keyword `fmincon`); exported `mincon.fmincon` (`crates/mincon-py/python/mincon/__init__.py:44`, `:949`); `Options::fmincon_compatible` (`crates/mincon-core/src/options.rs:613`); `README.md:3`, `:17` | Nominative and interoperability use, so it is defensible, but pair it with F-02's disclaimer. Reword `crates/mincon/Cargo.toml:3` and consider dropping the `fmincon` keyword |
| F-04 | low | Three committed MATLAB console logs contain warning stack traces that name **non-public** toolbox functions and line numbers (`fwdFinDiffInsideBnds`, `finitedifferences`, `computeFinDiffGradAndJac`, `sqpInterface`, `OptimFunctions/computeGradAndJac (line 606)`, `fmincon (line 671)`) | `bench/results/s2-dev/fmincon-sqp.A.jsonl.log:25`; `bench/results/s2-dev/fmincon-interior-point.A.jsonl.log:27`; `bench/results/s3-c1-dev/fmincon-interior-point.C.jsonl.log` | Delete these logs, or strip the stack traces. Nothing in the code reuses the names |
| F-05 | low | The harness stores `fmincon`'s exit message verbatim in every record: 7 message templates, about 320 words, repeated across about 3,600 records | `bench/harness/worker_matlab.m:167`; `bench/friction/run_friction_matlab.m:78` | Keep `exitflag` and drop `native_message`/`message` from published records |
| F-06 | low | `docs/20_SQP_MATHEMATICS.md` quotes short runs of the public MathWorks algorithm page (10-12 words each), in quotation marks and marked "(MW)", with URLs and a read date. §7 is an attributed paraphrase | `docs/20_SQP_MATHEMATICS.md:71`, `:246-247`, `:365`, `:577`, `:580`, `:591`; §7 `:563-603` | Acceptable as attributed quotation. Keep the quotes short |
| F-07 | low | mincon's own messages and comments echo MathWorks wording | `crates/mincon-core/src/result.rs:71`, `:74` ("Local minimum possible. …", the opening of `fmincon`'s exit-flag-2 message); `crates/mincon-core/src/error.rs:9-10` (close paraphrase of the sqp "smaller step" sentence) | Reword the two runtime messages. Cite the MathWorks page in `error.rs` |
| F-08 | low | Code adapted from **QDLDL** (Apache-2.0) keeps QDLDL's name but not its copyright notice. The elimination-tree loop matches `QDLDL_etree` statement for statement, and the numeric kernel borrows its structure (`lnext`, `dinv`). `LICENSE-APACHE` is a 17-line pointer, not the licence text. The Rust standard library notice is missing from the wheel's `THIRD_PARTY_LICENSES.txt` | `crates/mincon-linalg/src/ldlt.rs:241-265`, `:442-515`; `LICENSE-APACHE:1-3`; `crates/mincon-py/LICENSE-APACHE`; `crates/mincon-py/python/mincon/THIRD_PARTY_LICENSES.txt` | Add a QDLDL copyright/Apache-2.0 credit to the `ldlt.rs` header (or a NOTICE file), ship the full Apache-2.0 text, and add the Rust std notice |
| F-09 | low | The licence rule does not cover all the code it sends readers to. `docs/20` names GPL (R `quadprog`) and LGPL (`eiquadprog`) QP codes as "reference implementations"; the rule covers EPL/LGPL but not GPL, and does not name CSparse, LDL or CHOLMOD (LGPL). `mincon-qp` shares two helper names with QuadProg++ (MIT) but is not a translation of any of them | `docs/20_SQP_MATHEMATICS.md:709`; `docs/09_RESOURCES.md:177-182`, `:76-77`; `crates/mincon-qp/src/lib.rs:547`, `:583` | Extend the rule to GPL and to the Davis LGPL codes. Add a line to `mincon-qp` saying it was written from Goldfarb & Idnani (1983) |
| F-10 | low | `docs/09` tells readers to read IPOPT's EPL source files "for algorithm structure". No porting was found in `mincon-ip`: no IPOPT identifiers, paper notation, different restoration design | `docs/09_RESOURCES.md:165` | Keep a record of which EPL files were read, per the rule at `docs/09_RESOURCES.md:177-182` |
| F-11 | low | Corpus problem `BADSTART_DISC` is the `fmincon` reference-page example (Rosenbrock on the unit disk) with a different start and no citation. A mathematical problem is not protectable expression | `bench/corpus/adversarial.py:83` | Cite its origin |
| F-12 | low | The deleted `docs/01_FMINCON_ANATOMY.md` (a competitive analysis of `fmincon`) is still in public history. Its content cites public MathWorks pages and shares only the call signature with them | added `616cd5e`, untracked in `8ec00bd`, blob `31e48979` | None needed. Remove it from history only if desired |
| F-13 | low | The root commit records a "supplied" baseline of about 2,900 lines of solver code with no earlier history, so its authorship before 2026-09-05 is not recorded in the repository. Nothing in it matches MathWorks or IPOPT text or identifiers | `616cd5e` | Have the authors record where the baseline came from and who holds the rights |
| F-14 | low | Algorithm components with no published source cited. All are textbook-standard or the project's own heuristics; none looks MathWorks-derived | §3.2 | Add the citations listed in §3.2 |
| F-15 | low | Inaccurate statements about `fmincon`. These are not copying, but comparative claims about a named product should be accurate | §2.4 | Correct them |
| F-16 | low | `README.md:347` says every dependency is MIT, Apache-2.0 or BSD. The optional `bench` extra `matplotlib` has its own PSF-based licence | `README.md:347`; `crates/mincon-py/pyproject.toml:28` | Reword to "permissive" |

Not IP, but found along the way (privacy): the result records in
`bench/results` contain a Windows user name and laptop host name in 50+ files.
The 0.1.0 wheels' SBOMs embed a developer's local paths. The 0.2.0 wheels were
built on CI runners and do not.

---

## 1. MathWorks code, P-files or text

### 1.1 Current tree

| Check | Location | Evidence | Risk |
|---|---|---|---|
| MathWorks file types (`.p`, `.mex*`, `.mat`, `.mlx`, `.mlapp`, `.slx`, `.mdl`, `.mltbx`, `.fig`) | whole tree | none. The only MATLAB files are 6 hand-written `.m` scripts and 208 generated `.m` models (§6) | none |
| MathWorks headers and internal identifiers (`Copyright … The MathWorks`, `$Revision`, P-code header `v01.00v00.00`, `nlconst`, `qpsub`, `sqpLineSearch`, `optimlib`, `computeFinDiffGradAndJac`, `createExitMsg`, `toolbox/optim`, …) | all source, docs and scripts | 0 hits outside `bench/results`. The only hits are the MATLAB console logs in F-04 | none (code); low (F-04) |
| Shared wording with the MathWorks pages (runs of 8+ consecutive words) | whole tree, excluding `bench/results` | 18 runs. The 5 at `docs/20_SQP_MATHEMATICS.md:71`, `:246`, `:247`, `:577`, `:580` are attributed quotations (F-06). The 2 at `docs/09_RESOURCES.md:40-41` and the 2 at `docs/20_SQP_MATHEMATICS.md:659`, `:661` are paper titles (Byrd–Hribar–Nocedal, Waltz et al., Powell) that MathWorks also cites. The rest are the public call signature (`fun, x0, A, b, Aeq, beq, lb, ub, nonlcon`) at `bench/parity/parity_matlab.m:37` and `bench/friction/friction_problems.m:4`, `:8`, or formulas: `bench/parity/parity_matlab.m:28`, `bench/friction/friction_problems.m:29`, Rosenbrock at `crates/mincon-py/tests/test_fmincon.py:289` and `crates/mincon-py/tests/test_batch.py:18`, `:23`, and `bench/corpus/hs.py:68` | none / low (F-06) |
| Hand-written MATLAB scripts | `bench/harness/worker_matlab.m`, `bench/matlab/fmincon_baseline.m`, `bench/parity/parity_matlab.m`, `bench/friction/friction_problems.m`, `bench/friction/run_friction_matlab.m`, `bench/corpus/equivalence_matlab.m` | original drivers; they call only the public `fmincon`/`optimoptions` API and documented `output`/`lambda` fields | none |
| Generated MATLAB models | `bench/corpus/matlab/*.m` (208) | each starts `% NAME: generated by bench/corpus/generate.py from the SymPy definition. Do not edit.`, with no other comments. Regenerating from `git archive HEAD` reproduced 12 of 14 sampled files byte for byte; the other 2 differ only in float rounding | none |
| Test problems taken from MathWorks examples | crates, `bench/corpus`, `bench/friction` | only `BADSTART_DISC` (F-11). HS71 is Hock–Schittkowski #71 and is not a MathWorks example. The chained Rosenbrock tests use the Moré–Garbow–Hillstrom start, not MathWorks' | low (F-11) |
| MathWorks runtime text | `bench/results` records and logs | `fmincon` exit messages (F-05) and warning stack traces (F-04). These are program output captured by the harness | low |

### 1.2 Git history (76 commits)

| Check | Location | Evidence | Risk |
|---|---|---|---|
| MathWorks file types ever committed | `git rev-list --all --objects` | none. No `.p`, `.mat`, `.mex*`, `.mlx`, `.slx` or `.mltbx` object in any commit | none |
| Internal identifiers and MathWorks headers ever committed | pickaxe (`git log -S`/`-G`) and a grep of every blob | hits only in the three `bench/results` logs of F-04 (still present at HEAD) | low (F-04) |
| Content later deleted | `8ec00bd` "Keep internal briefs and planning notes local" removed `AGENT_PROMPT.md`, `docs/00_MISSION.md`, `docs/01_FMINCON_ANATOMY.md` and `docs/10_ROADMAP.md` (969 lines) | `docs/01_FMINCON_ANATOMY.md` is a competitive analysis built from three cited MathWorks doc pages, Kronqvist et al. and IPOPT's docs. It shares only the call signature with MathWorks' text and describes no internals | low (F-12) |
| Earlier wording in mincon's own messages | `616cd5e:crates/mincon-core/src/result.rs:68` | `"Local minimum found. First-order optimality and constraints satisfied."`, close to `fmincon`'s exit-flag-1 heading. Replaced in `e6c95ed` | low (history only) |
| Baseline origin | `616cd5e` | "Record supplied mincon baseline". The crates arrive complete, with no earlier history | low (F-13) |

### 1.3 Published PyPI files

All 7 files (0.1.0 built from `5e0f136`, 0.2.0 from `f69ff68`): no `.m`,
`.mat`, `.p` or `.mex` file, no `bench/` directory, no `fmincon` record. The
sdists ship only `crates/` and the root manifests. `strings` on the 5 compiled
extensions finds no MathWorks string. It finds only two `fmincon` notes, both
naming public options (`ScaleProblem`, `TypicalX`). Details are in §6.6.

---

## 2. `fmincon` internals and MathWorks wording

### 2.1 References to `fmincon` internals

Every statement about `fmincon` in the code and docs was checked against the
saved public pages. **None describes non-public behaviour.** The only
non-public names in the repository are MATLAB's own stack traces captured in
three logs (F-04). A grep for signs of reading the implementation ("in the
source", "line N of", `private/`, `edit fmincon`, `type fmincon`, p-code,
debugger, "step through", "undocumented", `toolbox/`) found nothing.
`docs/14_CAPABILITY_AND_DEFECT_INVENTORY.md:36` "Defects confirmed in the source" refers to mincon's own source.

Two statements go slightly beyond the saved docs and should be sourced:
- `crates/mincon-core/src/options.rs:485-486` claims interior-point
  finite-difference steps also respect bounds; the public page documents this
  for `sqp` only.
- `docs/20_SQP_MATHEMATICS.md:462` labels the Armijo constant `1e-4`
  "fmincon-like"; no public page gives it. It is the textbook value
  (Nocedal & Wright §3.1).

### 2.2 Reproduced MathWorks wording

| Location | Text in repo (≤ 1 line) | Closest MathWorks text | Verdict | Risk |
|---|---|---|---|---|
| `docs/20_SQP_MATHEMATICS.md:71` | `sqp` documents as "takes every iterative step in the region constrained by …" (MW) | same 10 words, in quotation marks | attributed quotation | low |
| `docs/20_SQP_MATHEMATICS.md:246-247` | "combines the objective and constraint functions into a merit function" (MW) | same | attributed quotation | low |
| `docs/20_SQP_MATHEMATICS.md:365` | "the most negative element of `q_k·s_k` is repeatedly halved" (MW) | same | attributed quotation | low |
| `docs/20_SQP_MATHEMATICS.md:577`, `:580` | "attempts to take a smaller step" / "attempts to obtain feasibility using a second-order …" | same (11-12 words) | attributed quotation in §7 | low |
| `docs/20_SQP_MATHEMATICS.md:563-603` (§7) | "What `fmincon`'s `sqp` does, as documented" | about 40 lines of compressed paraphrase: defaults, damping, merit, exit flags | attributed paraphrase of public facts | low |
| `docs/03_SPEC_SQP.md:56`; `docs/18_EVIDENCE_MATRIX.md:11`, `:26`; `docs/19_SQP_RD_PLAN.md:175` | "MathWorks lists that as one of the four things …" | no 6-word overlap | paraphrase and public facts | none |
| `crates/mincon-core/src/result.rs:71`, `:74` | "Local minimum possible. Step size below tolerance; constraints satisfied." | exit-flag-2 heading "Local minimum possible. Constraints satisfied." | 3 shared words, then original | low |
| `crates/mincon-core/src/result.rs:68` | "First-order optimality and constraint tolerances satisfied." | "Local minimum found that satisfies the constraints." | different wording | none |
| `crates/mincon-core/src/result.rs:81` | "No feasible point found." | "No feasible point was found." | generic 4-word functional phrase | none |
| `crates/mincon-core/src/error.rs:9-10` | "returns `Inf`, `NaN` or a complex value, it retries with a smaller step" | the sqp "Robustness to Non-Double Results" sentence | close paraphrase in a comment | low |
| `crates/mincon-py/python/mincon/__init__.py:765-774` | `_MATLAB_OPTIONS` / `_MATLAB_DISPLAY` alias tables | public option names and Display values | names only, no descriptions copied | none |
| `crates/mincon-py/python/mincon/__init__.py:133` | `Iter  F-count  f(x)  Feasibility  Optimality …` | iterative-display column names | short functional labels | none |
| all other user-facing strings (errors, notes, docstrings, README) | e.g. "model could not be evaluated at the initial point" | none | no 6-word run shared with any MathWorks page, apart from the public call signature | none |

### 2.3 Public API names used for compatibility

Option names (`OptimalityTolerance`, `ConstraintTolerance`, `StepTolerance`,
`MaxIterations`, `TypicalX`, `HonorBounds`, `OutputFcn`, `HessianFcn`,
`FiniteDifferenceType`, …), `Display` values, `lambda` field names
(`ineqlin`, `eqlin`, `ineqnonlin`, `eqnonlin`, `lower`, `upper`) and the
exit-flag integers (1, 2, 3, 0, −1, −2, −3) appear in
`crates/mincon-core/src/options.rs`, `crates/mincon-core/src/result.rs:9-33`
and `crates/mincon-py/python/mincon/__init__.py:765-825`, `:943`. They are used
as names and numbers for interoperability, with the project's own
descriptions. Risk: **none**. The exported function name `fmincon` is covered
under item 4.

### 2.4 Inaccurate statements about `fmincon` (F-15)

| Location | Statement | Public documentation says |
|---|---|---|
| `crates/mincon-core/src/options.rs:379`, `:622` | `fmincon` uses "a flat 400" iterations; `fmincon_compatible()` sets 400 | 400 for every algorithm except interior-point, which uses 1000 (`docs/20_SQP_MATHEMATICS.md:584` has it right) |
| `crates/mincon-diff/src/lib.rs:14`; `crates/mincon-diff/src/detect.rs:25` | "`fmincon` requires `JacobPattern`" | `JacobPattern` is not a `fmincon` option (it belongs to fsolve and lsq*) |
| `crates/mincon-ip/src/bfgs.rs:11` | `fmincon` uses Powell's 0.2 damping | MathWorks documents an element-wise modification. `docs/20_SQP_MATHEMATICS.md:364-366` describes it correctly |
| `crates/mincon-core/src/options.rs:144`; `crates/mincon-diff/src/check.rs:3`; `crates/mincon/src/lib.rs:119` | refer to the `CheckGradients` option | the option was removed in R2026a and replaced by the `checkGradients` function |
| `crates/mincon-sqp/src/lib.rs:49-50` | describes the Powell per-row penalty update | the code uses a single scalar penalty (`docs/03_SPEC_SQP.md:6-8`), so this crate doc is stale |

---

## 3. Algorithm provenance

The master sources are `docs/09_RESOURCES.md` (§1 papers, §2 books,
§6 techniques) and `docs/20_SQP_MATHEMATICS.md` §10. The specs add more:
`docs/02` (interior point), `docs/04` (linear algebra), `docs/05` (derivatives),
`docs/06` (scaling and termination), `docs/12` (restoration), `docs/21` and
`docs/22`. The licence rule at `docs/09_RESOURCES.md:177-182` says that
EPL/LGPL code may be read and cited but never vendored, and it was followed.

### 3.1 Components with a traceable published source

| Component | Implementation | Published source | Cited in |
|---|---|---|---|
| Primal-dual barrier method, slack form, fraction-to-boundary, bound-multiplier reset | `crates/mincon-ip/src/solver.rs` | Wächter & Biegler 2006, *Math. Prog.* 106 | `docs/09` §1; `docs/02` |
| Filter line search, switching condition, second-order correction | `crates/mincon-ip/src/filter.rs`; `crates/mincon-ip/src/solver.rs:1387-1449` | Fletcher & Leyffer 2002; Wächter & Biegler 2006 §2.3-2.4; Maratos 1978 | `docs/09` §1, §6; `docs/02` §3 |
| Inertia correction (Algorithm IC) | `crates/mincon-ip/src/kkt.rs:36-79` | Wächter & Biegler 2006 §3.1 | `docs/02` §2.2 |
| Monotone and adaptive (LOQO) barrier update | `crates/mincon-ip/src/solver.rs:1009-1045` | Fiacco & McCormick 1968; Vanderbei & Shanno 1999; Nocedal, Wächter & Waltz 2009 | `docs/02` §4; `docs/18` (not in `docs/09`) |
| Feasibility restoration (soft, then elastic) | `crates/mincon-ip/src/solver/restoration.rs` | Wächter & Biegler 2006 §3.3. The analytic p/n elimination is a declared original variant | `docs/12` |
| Gradient-based scaling, scaled KKT error `s_d`/`s_c` | `crates/mincon-ip/src/solver.rs:401-461`, `:1510-1563` | Wächter & Biegler 2006 §3.8, eq. (5); IPOPT options | `docs/06`; `docs/09` §1 |
| Damped BFGS on the Lagrangian | `crates/mincon-ip/src/bfgs.rs:392-543` | Powell 1978; Nocedal & Wright Procedure 18.2 | `docs/09` §6; `docs/20` §4 |
| Curvature test without inertia (helper) | `crates/mincon-ip/src/kkt.rs:626-678` | Chiang & Zavala 2016 | `docs/09` §1 |
| SQP outer loop and local Newton-KKT step | `crates/mincon-sqp/src/solver.rs:1293-2064` | Wilson 1963; Han 1976/77; Powell 1978; Nocedal & Wright ch. 18 | `docs/20` §1, §10 |
| Elastic ℓ1 QP | `crates/mincon-sqp/src/solver.rs:696-770` | Fletcher 1985 (Sℓ1QP); Gill, Murray & Saunders 2005 (SNOPT) | `docs/20` §3, §10 |
| ℓ1 merit function, penalty update, Armijo | `crates/mincon-sqp/src/solver.rs:366-389`, `:1662-1790` | Han 1977; Han & Mangasarian 1979; Nocedal & Wright (18.36); Byrd, Nocedal & Waltz 2008 | `docs/20` §5, §10 |
| SQP second-order correction | `crates/mincon-sqp/src/solver.rs:1811-1876` | Maratos 1978; Fletcher 1982; Mayne & Polak 1982 | `docs/20` §5.4 |
| Adaptive QP step bound | `crates/mincon-sqp/src/solver.rs:1886-1902` | SNOPT major step limit (Gill, Murray & Saunders 2005); trust-region update (Nocedal & Wright Alg. 4.1) | `docs/20` §11.2 |
| Goldfarb–Idnani dual active-set QP, Givens add/drop, equalities, infeasibility certificate | `crates/mincon-qp/src/lib.rs:265-616` | Goldfarb & Idnani 1983; Powell 1985 (dependence) | `docs/20` §2, §10 |
| Sparse up-looking LDLᵀ, elimination tree, row reach, triangular solves | `crates/mincon-linalg/src/ldlt.rs` | Davis 2006; Liu 1990; QDLDL (Apache-2.0, see F-08) | `docs/04`; `docs/09` §2, §4 |
| Quasi-definite factorization, inertia by Sylvester's law, growth guard, iterative refinement | `crates/mincon-linalg/src/ldlt.rs:91-144`, `:505-678` | Vanderbei 1995; Higham 2002 | `docs/04`; `docs/09` §2, §6 |
| CSC storage and triplet compression | `crates/mincon-linalg/src/csc.rs`; `crates/mincon-core/src/sparsity.rs` | Davis 2006 ch. 2 | `docs/09` §2 |
| Curtis–Powell–Reid column grouping; distance-1 and distance-2 colouring | `crates/mincon-diff/src/coloring.rs` | Curtis, Powell & Reid 1974; Coleman & Moré 1983, 1984 | `docs/09` §1; `docs/05` §3 |
| Finite-difference error estimate D(h) − D(2h) | `crates/mincon-diff/src/fd.rs:244-352` | Richardson extrapolation (named in code) | `crates/mincon-diff/src/fd.rs:351` |
| Performance and data profiles | `bench/profiles.py:7`; `bench/harness/analyze.py:129` | Dolan & Moré 2002; Moré & Wild 2009 | `docs/09` §3 |
| Test problems | `crates/mincon-testset/src/hs.rs`; `bench/corpus/hs.py` | Hock & Schittkowski 1981 (LNEMS 187) | `docs/09` §3 |

### 3.2 Components with no traceable published source

None of these is derived from MathWorks internals. Each is either
textbook-standard with a citation missing (a fix is suggested) or the project's
own measured heuristic ("original"). All are **low** (F-14) or **none**.

| Component | Implementation | Status | Suggested citation |
|---|---|---|---|
| Algorithm portfolio (racing configurations) | `crates/mincon/src/portfolio.rs:80`, `:339` | textbook idea, uncited | Rice 1976; Gomes & Selman 2001 |
| Quadratic-program probe (third differences along random lines), structured band Hessian build, decaying bands, convex quadratic rows | `crates/mincon/src/quadratic.rs:15`, `:302`, `:843-1083` | original heuristic built on textbook finite differences | Nocedal & Wright §8.1; Boyd & Vandenberghe §3.1 |
| Variable scaling (`ScaledNlp`, `max(|x0|, typical_x)`) | `crates/mincon/src/scaled.rs:17-123` | textbook, uncited | Gill, Murray & Wright 1981 §7.5; Dennis & Schnabel 1983 §7.1 |
| XorShift and LCG generators (probes and tests) | `crates/mincon/src/quadratic.rs:148`; `crates/mincon-linalg/src/ldlt.rs:735-753` | textbook, uncited | Marsaglia 2003; Knuth |
| Finite-difference step sizes `sqrt(eps)` / `eps^(1/3)` scaled by `max(|x|, typical)` | `crates/mincon-diff/src/fd.rs:79-164` | cited only as "matching `fmincon`" | Dennis & Schnabel 1983 §5.4; Nocedal & Wright §8.1 |
| Bounds-respecting finite differences, retreat on non-finite values, unequal-step and one-sided stencils | `crates/mincon-diff/src/fd.rs:106-131`, `:394-413`, `:923-1033` | behaviour described publicly by MathWorks; stencils textbook; derivation in `docs/05` §2 | Fornberg 1988 (unequal steps) |
| Forward→central escalation; error-aware termination | `crates/mincon-diff/src/fd.rs:176-242`; `crates/mincon-ip/src/solver.rs:765-826` | original | none needed (mark as original) |
| Black-box sparsity detection by probing | `crates/mincon-diff/src/detect.rs:81-181` | original, simpler than published schemes | Griewank & Mitev 2002 (related work) |
| Derivative checkers (component-wise, directional) | `crates/mincon-diff/src/check.rs:129-448` | textbook practice | Nocedal & Wright §8.1 |
| Reverse Cuthill–McKee ordering | `crates/mincon-linalg/src/ordering.rs:87-130` | named only | Cuthill & McKee 1969; George & Liu 1981 |
| Positive diagonal congruence retry; natural-order fallback | `crates/mincon-ip/src/kkt.rs:523-624` | original | — |
| Certificate guards (D9, D11), divergence and stall exits, progress verdict | `crates/mincon-ip/src/solver.rs:827-989`, `:1565-1601`, `:1863-1899` | original | — |
| Adaptive→monotone barrier fallback | `crates/mincon-ip/src/solver.rs:1026-1040` | original safeguard | — |
| Curvature-drift BFGS rebuild (off by default) | `crates/mincon-ip/src/bfgs.rs:93-302` | original (`docs/21`) | — |
| Exact-Hessian regularisation along constraint normals δ(JᵀJ + E_B) | `crates/mincon-sqp/src/solver.rs:566-615` | textbook idea, uncited | Debreu 1952; Nocedal & Wright §17.3 |
| Initial penalty ρ₀ = max(1, ‖g₀‖∞/‖J₀‖∞) | `crates/mincon-sqp/src/solver.rs:1321-1324` | only analogue cited is MathWorks' public per-row rule | cite a paper or label original |
| Non-finite retreat in the SQP line search | `crates/mincon-sqp/src/solver.rs:1786-1790` | standard practice, uncited | — |
| Saddle probe and escape, zero-step re-test, degenerate-multiplier verdict, B₀ curvature rescale, line-search failure cascade | `crates/mincon-sqp/src/solver.rs:825-1116`, `:1502-1585`, `:1699-1722`, `:1886-1995` | original (`docs/20` §11, `docs/22` §7.18) | Moré & Sorensen 1979 (negative curvature) |
| Dense kernels: Gaussian elimination, Gram–Schmidt, cyclic Jacobi | `crates/mincon-sqp/src/solver.rs:877-926`, `:2071-2190` | textbook, uncited (the structure is not Numerical Recipes') | Golub & Van Loan |
| QP constraint selection by scaled violation; warm-start hint ordering | `crates/mincon-qp/src/lib.rs:149-153`, `:324-348` | textbook, uncited | — |
| Multistart | `crates/mincon-py/python/mincon/__init__.py:1161-1249` | textbook, uncited | Rinnooy Kan & Timmer 1987 |
| KKT fixture checker `relative_stationarity` | `crates/mincon-testset/src/lib.rs:101-229` | textbook, uncited | Nocedal & Wright ch. 12 |
| Torture problems | `crates/mincon-testset/src/torture.rs:70-281` | original constructions | — |

---

## 4. Use of the MATLAB and `fmincon` names

| Location | Use (≤ 1 line) | Risk |
|---|---|---|
| `README.md:3` | "A nonlinear constrained optimizer in Rust, aimed at what MATLAB's `fmincon` does well" | low (nominative) |
| `README.md:17-20` and throughout (30 mentions of `fmincon`, 4 of MATLAB) | `from mincon import fmincon`; benchmark claims against `fmincon-sqp` and `fmincon-interior-point` | low (F-02, F-01) |
| `crates/mincon/Cargo.toml:3` | "a free, fast, pip-installable answer to MATLAB's fmincon". Unchanged since `616cd5e`; shipped in every wheel's `dist-info/sboms/mincon-py.cyclonedx.json` and in both sdists | low (F-03) |
| `crates/mincon-py/pyproject.toml:7` | PyPI Summary: "… with simple Python and MATLAB-style interfaces" (toned down in `5e0f136` from "A free answer to MATLAB's fmincon.") | low |
| `crates/mincon-py/pyproject.toml:12` | PyPI keyword `fmincon` | low |
| `crates/mincon-py/python/mincon/__init__.py:44`, `:949` | public function `fmincon(fun, x0, A, b, Aeq, beq, lb, ub, nonlcon, …)`; its docstring says it is "not MATLAB output-tuple compatibility" (`:997`) | low (interoperability; see F-02) |
| `crates/mincon-core/src/options.rs:613` | public `Options::fmincon_compatible()` | low |
| `crates/mincon-py/README.md:8-9`, `:235` | "not a completed or proven superior replacement for MATLAB's `fmincon`"; "no MATLAB installation or license is required" | none (accurate, and signals independence) |
| crate rustdoc (`crates/mincon-ip/src/lib.rs:3`, `crates/mincon-sqp/src/lib.rs:9-55`, `crates/mincon-diff/src/lib.rs:14`, `crates/mincon-testset/src/lib.rs:21`, …) | comparative references; the crates have no `publish = false`, so these would appear on docs.rs | low |
| package name `mincon` | `fmincon` without the "f" | low (descriptive; a trademark search before wider publication would settle it) |
| history: `docs/01_FMINCON_ANATOMY.md` | trademark in a historical file name | low (F-12) |

**Disclaimer:** none exists. The search covered `affiliat`, `endorse`,
`trademark`, `registered` and `®` across the tree, the history, the PyPI page
and all 7 PyPI files; the only hits are Apache licence boilerplate. "MATLAB"
is a registered trademark of The MathWorks, Inc. This audit did not check
whether "fmincon" itself is registered. Suggested text is in F-02.

---

## 5. Licences

### 5.1 Project licence

`MIT OR Apache-2.0` (`Cargo.toml:19`; `crates/mincon-py/pyproject.toml:9-10`).
`LICENSE-MIT` is complete. `LICENSE-APACHE` (root and `crates/mincon-py/`) is
the 17-line Apache boilerplate notice with a URL, not the licence text (F-08).
The wheels ship both files, plus a full Apache-2.0 text inside
`THIRD_PARTY_LICENSES.txt`.

### 5.2 Rust dependencies (all 33 third-party crates in `Cargo.lock`)

Each crate's licence was checked against the crates.io API and against the
bundled `crates/mincon-py/python/mincon/THIRD_PARTY_LICENSES.txt`, which is
generated by `scripts/build_license_notice.py` and fails if a crate has no
licence file. All 33 agree. Lockfile checksums match crates.io.

| Licence | Crates | Risk |
|---|---|---|
| MIT OR Apache-2.0 | crossbeam-deque 0.8.7, crossbeam-epoch 0.9.20, crossbeam-utils 0.8.22, either 1.18.0, heck 0.5.0, libc 0.2.189, matrixmultiply 0.3.11, ndarray 0.17.2, num-complex 0.4.6, num-integer 0.1.47, num-traits 0.2.19, once_cell 1.21.4, proc-macro2 1.0.107, pyo3 / pyo3-build-config / pyo3-ffi / pyo3-macros / pyo3-macros-backend 0.29.2, quote 1.0.47, rawpointer 0.2.1, rayon 1.12.0, rayon-core 1.13.0, syn 2.0.119 and 3.0.5, thiserror / thiserror-impl 2.0.20, autocfg 1.5.1, portable-atomic 1.15.0, portable-atomic-util 0.2.8, rustc-hash 2.1.3 | none |
| BSD-2-Clause | numpy (rust-numpy) 0.29.0 | none (the notice is reproduced in the wheel) |
| Apache-2.0 WITH LLVM-exception | target-lexicon 0.13.5 (build-time) | none |
| (MIT OR Apache-2.0) AND Unicode-3.0 | unicode-ident 1.0.24 (build-time) | none |
| MIT OR Apache-2.0 (toolchain) | Rust standard library, statically linked into the extension | low: not listed in `THIRD_PARTY_LICENSES.txt` (F-08) |

### 5.3 Python dependencies

| Package | Role | Licence (PyPI) | Risk |
|---|---|---|---|
| numpy ≥ 1.21 | runtime | BSD-3-Clause (current releases also bundle 0BSD, MIT, Zlib and CC0-1.0 parts) | none |
| scipy, pandas | `bench` extra | BSD-3-Clause | none |
| matplotlib | `bench` extra | Matplotlib licence (PSF-based, permissive) | low (F-16) |
| sympy (+ mpmath) | corpus generator | BSD | none |
| pytest / maturin / twine | `dev` extra | MIT / MIT OR Apache-2.0 / Apache-2.0 | none |
| cyipopt | optional import in `bench/runner.py:169`, not declared | EPL-2.0 | none (not redistributed) |

### 5.4 CI actions (not distributed)

`actions/checkout`, `actions/setup-python`, `actions/upload-artifact` and
`actions/download-artifact` are MIT. `dtolnay/rust-toolchain` is MIT.
`PyO3/maturin-action` is MIT. `pypa/gh-action-pypi-publish` is BSD-3-Clause.
`Swatinem/rust-cache` is LGPL-3.0 but is only used in CI. Risk: **none**.

### 5.5 Third-party test sets, corpus and data

| Material | Where | Licence / status | Risk |
|---|---|---|---|
| Hock–Schittkowski problems (45 in Rust, 93 in the corpus) | `crates/mincon-testset/src/hs.rs`; `bench/corpus/hs.py` | Transcribed from the 1981 book as formulas and published optima (facts). They are not copied from Schittkowski's PROB.FOR (which has no licence) or from CUTEst SIF: the reference values differ from both, and there is no Fortran or SIF structure | none |
| Engineering design problems (CANTILEVER, CORRUGATED_BULKHEAD, PRESSURE_VESSEL, WELDED_BEAM, SPEED_REDUCER, …) | `bench/corpus/structured.py`, `heldout5.py`, `heldout6.py`; `bench/friction/problems.py` | Published formulas with source strings (Sandgren 1990, Golinski, Arora, Kim & Lee 1998, Hoburg & Abbeel 2014, …). `pressure_vessel` (`bench/friction/problems.py:381`) says only "published" | none |
| Structured families (CSTR, DISPATCH, CATENARY, EXPFIT, …) and torture problems | `bench/corpus/structured.py`, `heldout*.py`; `crates/mincon-testset/src/torture.rs` | Original, with seeded generated data | none |
| S2MPJ / CUTEst problems | `bench/s2mpj_bridge.py:57-96` | Downloaded at run time into `~/.cache/s2mpj`; nothing is committed. S2MPJ is BSD-3-Clause (verified from its `LICENCE.txt`); CUTEst is BSD-3 | none |
| Generated models, probes, targets, manifests | `bench/corpus/matlab/*.m`, `probes.json`, `matlab_probes.json`, `targets_v*.json`, `manifest.json`, `bench/friction/friction_data.json` | Project-generated | none |
| Result records | `bench/results/**` | Project-generated, but produced with MATLAB (F-01) and containing MathWorks runtime text (F-04, F-05) | see F-01, F-04, F-05 |
| QDLDL-derived code | `crates/mincon-linalg/src/ldlt.rs` | Apache-2.0; compatible, but the notice is not kept | low (F-08) |

---

## 6. Inventory of MATLAB-related material

### 6.1 MATLAB scripts (hand-written, original)

| File | Lines | Purpose | Notes |
|---|---:|---|---|
| `bench/harness/worker_matlab.m` | 177 | runs corpus problems through `fmincon`, writes JSONL records | stores `output.message` (`:167`) and the MATLAB version string |
| `bench/matlab/fmincon_baseline.m` | 155 | older S2MPJ-based baseline driver | records `version()` and `computer()` |
| `bench/parity/parity_matlab.m` | 39 | 5 small problems × 2 algorithms | records release and toolbox version |
| `bench/friction/friction_problems.m` | 147 | MATLAB port of the 15 friction problems | problems are original or published |
| `bench/friction/run_friction_matlab.m` | 91 | runs the friction audit, writes JSONL | stores `output.message` (`:78`) and `lastwarn` (`:88`) |
| `bench/corpus/equivalence_matlab.m` | 22 | evaluates the generated models at probe points | writes `bench/corpus/matlab_probes.json` |

### 6.2 Generated MATLAB models

There are 208 files, `bench/corpus/matlab/*.m`, one per `bench/corpus/manifest.json`
entry. They are generated by `bench/corpus/generate.py` through SymPy's
`OctaveCodePrinter` (`bench/corpus/codegen.py`). Their sources: structured
families 87, Hock–Schittkowski 93, closed-form adversarial 18, published
engineering designs 10. Every commit that touches them also changes the
generator (`6315a27`, `1c952c0`, `14ccf70`, `985bdbf`, `273ea7e`, `8ce240a`).

### 6.3 Probe and parity files

| File | Content | MathWorks text |
|---|---|---|
| `bench/corpus/probes.json` | NumPy values of f, ∇f, c, J at probe points | none |
| `bench/corpus/matlab_probes.json` | the same quantities evaluated by MATLAB from the generated models. `equivalence.py` compares them (0 mismatches reported) | none (numbers only) |
| `bench/parity/parity_matlab.json` | 10 `fmincon` runs: x, fval, exitflag, iterations, funcCount, lambda, release "2025b", toolbox "25.2" | none |
| `bench/parity/parity_mincon.json` | mincon's side of the parity check | none |

### 6.4 `fmincon` result records

There are 3,672 records whose `solver` starts with `fmincon`: interior-point
2,250, sqp 1,354, and 68 option-override runs. They sit in 85 files across
25 directories (the scored copies are included):

`s2-dev` 328 · `s3-c1-dev` 656 · `s4-c2-dev` 328 · `s4-c2-val` 192 ·
`s6-final` 192 · `s6-diagnostic` 6 · `s6-timing` 72 · `s6v2-final2` 128 ·
`s6v2-timing` 48 · `s6v3-final3` 96 · `s6v3-timing` 72 · `s6v4-final4` 132 ·
`s6v4-final4-timing` 66 · `s6v5-final5` 104 · `s6v5-final5-timing` 78 ·
`ref-final2` 32 · `abl-c4`, `abl-c5`, `abl-c6` 260 each ·
`r4-budget/fmincon-caps` 68 · `r3-basins` 58 · `s7-friction`, `-b`, `-c`, `-d`
56, 60, 60, 60.

Fields: `solver_version` ("MATLAB 2025b optim 25.2"), `host.matlab`
("25.2.0.3042426 (R2025b) Update 1"), options, x, multipliers, `native_status`
(exitflag), `native_message` (F-05), iterations, function counts, first-order
optimality, constraint violation and timing. No licence number, host ID,
activation or licence-type string appears. Three `.log` files hold MATLAB
console output (F-04).

### 6.5 Docs that report `fmincon` numbers

- `README.md:33-92` (status block: attainment, evaluation ratios, "13× faster", friction audit), `:245-248`
- `bench/parity/README.md:13-33`
- `docs/16_FAILURE_ATLAS.md:12-50`
- `docs/17_CLAIM_AUDIT.md:9-30`
- `docs/18_EVIDENCE_MATRIX.md:11`, `:25`, `:26`
- `docs/19_SQP_RD_PLAN.md:24`, `:57-58`, `:69-116`, `:304-306`
- `docs/20_SQP_MATHEMATICS.md:595-599`
- `docs/21_LARGE_N_CURVATURE_PLAN.md:14-16`
- `docs/22_ROUND5_ROBUSTNESS_PLAN.md:18-19`, `:733-743`
- `docs/23_ROADMAP_TO_1_0.md:56`, `:129`, `:170`, `:224`
- `docs/15_BENCHMARK_PROTOCOL_V2.md:118-167` (protocol; names "the same Windows laptop" at `:99`)
- 87 of the 140 `bench/results/**/*.md` summaries

Third-party `fmincon` figures, quoted with attribution: Kronqvist et al.'s
75.9% in `bench/README.md:107-112`, `docs/08_BENCHMARK_PROTOCOL.md:111-117` and
`docs/09_RESOURCES.md:147-148`.

### 6.6 Published PyPI packages

The PyPI project links to `github.com/atciamb/mincon`, and the 0.2.0 metadata
has no Author field. Common to all 7 files: Summary "Experimental nonlinear constrained
optimization in Rust with simple Python and MATLAB-style interfaces"; keyword
`fmincon`; `License-Expression: MIT OR Apache-2.0`; no disclaimer.

| File | Built from | Contents | MATLAB material / records | Risk |
|---|---|---|---|---|
| `mincon-0.1.0-cp39-abi3-manylinux_2_17_x86_64….whl` | `5e0f136` | `mincon/{__init__.py, _mincon.abi3.so, THIRD_PARTY_LICENSES.txt}`, dist-info with both licences and a CycloneDX SBOM | none | low (F-02, F-03) |
| `mincon-0.1.0-cp39-abi3-win_amd64.whl` | `5e0f136` | same, `_mincon.pyd` | none | low |
| `mincon-0.1.0.tar.gz` | `5e0f136` | 58 entries: `crates/*` sources and root manifests | none; no `bench/` | low |
| `mincon-0.2.0-cp39-abi3-macosx_…universal2.whl` | `f69ff68` | same layout, universal2 `.so` | none | low |
| `mincon-0.2.0-cp39-abi3-manylinux_2_17_x86_64….whl` | `f69ff68` | same | none | low |
| `mincon-0.2.0-cp39-abi3-win_amd64.whl` | `f69ff68` | same | none | low |
| `mincon-0.2.0.tar.gz` | `f69ff68` | 70 entries: `crates/*` sources and tests, root manifests. 66 are byte-identical to git; the rest are maturin rewrites | none; no `bench/` | low |

---

## Appendix: method and coverage

The audit was split across 11 read-only subagents: one per crate under
`crates/` (`mincon`, `mincon-core`, `mincon-diff`, `mincon-ip`,
`mincon-linalg`, `mincon-py`, `mincon-qp`, `mincon-sqp`, `mincon-testset`), one
for `bench/` + `docs/` (with the README, CHANGELOG, scripts and CI), and one
for the full git history. Each agent read every file in its scope (the corpus
models and result records were sampled and aggregated rather than read one by
one), then checked each file's history. The coordinator then re-checked every
**needs lawyer** item, the cross-cutting claims and a sample of **low** items
against the files themselves.

Tools and checks:
- **Word-overlap scan** against the saved MathWorks documentation text, at 8
  words and, for user-facing strings, 6 and 5 words. The doc text is kept
  outside the repository; no MathWorks code was fetched.
- **Identifier and header grep** for non-public Optimization Toolbox internals,
  MathWorks copyright and `$Revision` markers and the P-code header. It ran
  over the tree, every blob in history (`git rev-list --all --objects`), and
  the pickaxe (`git log -S` / `-G`).
- **Structural comparison** against the open-source reference codes the docs
  mention: QDLDL, LDL, CSparse, R `quadprog`, QuadProg++, eiquadprog and IPOPT
  identifiers.
- **Licences** from the crates.io and PyPI APIs, the raw licence files of the
  CI actions and S2MPJ, and `THIRD_PARTY_LICENSES.txt`.
- **PyPI**: all 7 files downloaded and verified by SHA-256; contents, metadata,
  SBOMs and the compiled extensions (via `strings`) inspected.

Limitations:
- The history of the upstream repository `atciamb/mincon` beyond the 76
  commits present here was not audited.
- The origin of the baseline code before `616cd5e` is not recorded (F-13).
- Wächter–Biegler section numbers in §3 were checked against the repository
  docs, not re-read from the paper.
- The MATLAB licence terms were not available (F-01).
