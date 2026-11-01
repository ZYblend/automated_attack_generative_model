# Hand-over: next paper revisions

**Baseline:** `main` @ merge of [#1](https://github.com/ZYblend/automated_attack_generative_model/pull/1) (sanity-check PDF approved).  
**Compile:** `pdflatex → bibtex → pdflatex ×2` on `bare_jrnl.tex` → `bare_jrnl.pdf`; revised artifact: `artifacts/kf_chisquare_revision.pdf` (7 pp).  
**Sim code (external):** https://github.com/ZYblend/Physics-guided_GAN_Attack_Generation  
**Status (2026-11-01):** The authoritative revised sections use `*_revised.tex` files on branch `cursor/kf-chisquare-sparsity-bridge-ac49`. The theory has been rewritten for KF/$\chi^2$ consistency. Archived simulations support only empirical observability and legacy data-augmentation results; they are not presented as numerical validation of the revised generator, calibrated selector, or subset KF.

---

## Goal (two changes)

1. **Replace** the least-squares / receding-horizon \(\ell_2\) observer and residual-based bad-data detector with a **Kalman filter (KF)** and a **\(\chi^2\) detector**.
2. **Build a quantitative bridge** among **attack sparsity**, **observability**, **estimation error**, and **detection precision** (PPV / localization precision after pruning).

Keep the rest of the pipeline: physics-guided attack generator → MLP detect/localize → pruning → resilient estimation on the retained safe sensors.

---

## Current baseline (what to replace)

| Piece | Where | Role today |
|--------|--------|------------|
| Receding-horizon LS observer | `2-System_model.tex` (`equ:RHE`, `equ:state_estimate`) | Nominal state estimate \(\hat x_i\) |
| Residual BDD | `3-attack_generation.tex` (`equ:residual`, \(\|r\|\le\tau\)) | Stealthiness side of successful FDIA; also used in sims |
| Attack maps \(M_1,M_2\) | `3-attack_generation.tex` | Stealthiness / effectiveness discriminators for the GAN |
| MLP + pruning | `4-detection_localization.tex` | Support \(\hat{\mathcal T}\), pruned safe set \(\hat{\mathcal T}^c_\eta\), PPV, \(\eta\)–\(k_0\) link |
| Resilient \(\ell_2\) on pruned sensors | `5-resilient_estimation.tex` | Same LS form on \(C_{\hat{\mathcal T}^c_\eta}\); Thm. error = attack-free w.p. \(\ge\eta\) |
| WDS sims | `6-simulation.tex` + `new_sim_image/` | Observability \(k_0\), residuals, pSNO/pTA, precision vs #attacks |

Assumption **A2** (`2-System_model.tex`): \(\exists k_0\) s.t. \((A,C_{\mathcal S})\) observable for all \(|\mathcal S|\ge k_0\). Pruning already ties \(\eta\) to \(l_\eta\ge k_0\) (`Thm:eta_upper_bound`, `equ:localization_performance_criterion`).

---

## Change 1 — Kalman filter + \(\chi^2\) detector

### Modeling
- Add process/measurement noise to the CPS model (needed for KF and \(\chi^2\)):
  \[
  x_{i+1}=Ax_i+Bu_i+w_i,\quad
  y_i=Cx_i+v_i+e_i,
  \]
  with \(w\sim\mathcal N(0,Q)\), \(v\sim\mathcal N(0,R)\) (match WDS noise used in sim).
- **KF (predict/update)** as the nominal (and resilient) estimator in place of `equ:RHE` / `equ:state_estimate` / `equ:resilient_L2`.
- **\(\chi^2\) detector** on the innovation \(r_i=y_i-C\hat x_{i|i-1}\) (or a windowed sum of normalized innovations):
  \[
  g_i = r_i^\top S_i^{-1} r_i \;\underset{\mathcal H_1}{\overset{\mathcal H_0}{\gtrless}}\; \tau_{\chi^2},
  \]
  with \(S_i\) the innovation covariance; threshold from \(\chi^2\) tables (chosen false-alarm rate) instead of ad-hoc \(\|r\|\le\tau\).

### Paper touch points
1. **`2-System_model.tex`** — noise model; KF equations; remove or demote pure LS RHE as the main observer.
2. **`3-attack_generation.tex`** — redefine stealthiness vs \(\chi^2\) (and effectiveness vs KF-induced critical-output bias). Rebuild \(M_1,M_2\) (or equivalents) for the physics discriminators so the GAN still targets “successful” FDIAs under the new detector/estimator.
3. **`5-resilient_estimation.tex`** — resilient KF on \(y_{\hat{\mathcal T}^c_\eta}\) (reduced \(C\), \(R\)); retarget Fig. `image/resilient_L2_observer.png` (L2 block → KF; detection block → \(\chi^2\) / MLP as designed).
4. **`1-Introduction.tex` / abstract / keywords / `7-conclusion.tex`** — LS/residual → KF/\(\chi^2\).
5. **`6-simulation.tex`** — residual plots / pTA definitions → innovation / \(\chi^2\) statistic; regenerate figures from sim repo as needed.

### Sim / code
- Repo: `Physics-guided_GAN_Attack_Generation` (MATLAB/Simulink WDS + Python generator).
- Port observer + BDD paths to KF + \(\chi^2\); retrain generator discriminators; refresh `new_sim_image/` and any schematics under `image/`.

### Suggested theorem restatement
- With probability \(\ge\eta\), pruned measurements are clean and \(|\hat{\mathcal T}^c_\eta|\ge k_0\) ⇒ KF on that subset matches the **attack-free KF** error (analog of current `Thm:resilient_estimation_gaurentee`), under standard KF detectability on \((A,C_{\hat{\mathcal T}^c_\eta})\).

---

## Change 2 — Quantitative bridge: sparsity ↔ observability ↔ error ↔ precision

Make the chain **explicit and quantitative** (not only qualitative + Monte Carlo).

### Intended chain
```text
attack sparsity k
    → remaining safe set size / pruning length l_η
        → observability (A, C_S), |S| ≥ k_0
            → KF error (P, or ||x̂−x|| bound / MSE)
                ↔ localization precision PPV_η / η
```

### Concrete deliverables in the paper
1. **Sparsity → observability**  
   - Keep / sharpen A2 and empirical \(k_0(m)\) (Fig. observability).  
   - Relate feasible attack sparsity \(k\) to worst-case remaining sensors after miss-localization + pruning: e.g. \(|\hat{\mathcal T}^c_\eta| \ge m-k-\text{(false safe)}\), and require \(\ge k_0\).

2. **Observability / subset → estimation error**  
   - For KF: express \(P_{i|i}\) (or steady-state \(P_\infty(S)\)) vs sensor subset \(S=\hat{\mathcal T}^c_\eta\).  
   - Optional compact bound: \(\mathbb E\|x-\hat x\|^2 \le f(\sigma_{\min}(O_S), Q, R)\) or compare \(P_\infty(S)\) vs \(P_\infty(\{1..m\})\).  
   - This replaces “error equals attack-free LS” with a **graded** error vs how much is pruned / which sensors remain.

3. **Precision ↔ error**  
   - Use existing \(\Pr\{\mathrm{PPV}_\eta=1\}\ge\eta\) (`equ:precision_after_pruning`).  
   - Bridge: higher \(\eta\) ⇒ shorter \(l_\eta\) ⇒ risk of \(|S|<k_0\) or larger \(P_\infty(S)\); lower \(\eta\) ⇒ longer \(S\) but more contaminated sensors ⇒ bias in KF.  
   - State an explicit trade-off (theorem or proposition + corollary): e.g. for given \(k,k_0,\mathbf p\), admissible \(\eta\) interval such that both observability and precision hold, and bound \(\mathbb E[\|e_{\mathrm{est}}\|^2]\) on that interval.

4. **Detection precision in the \(\chi^2\) sense**  
   - Separate **channel localization PPV** (MLP/pruning) from **detection power** of \(\chi^2\) (false alarm / miss vs attack energy and \(k\)).  
   - Optionally couple: stealthy GAN attacks constrained by \(\chi^2\) threshold ⇒ bound on detectable \(k\) or on innovation SNR.

5. **Simulation proof of the bridge**  
   - Joint plots (new or revised): \(k\) vs \(k_0\)-feasibility; \(k\) vs PPV; \(k\) / \(|S|\) vs KF MSE; \(\eta\) sweep vs MSE and PPV.  
   - Align captions/text with the theorem chain so Sec. VI is evidence for Sec. IV–V, not a separate story.

### Best place for new theory
- Core math in **`4-detection_localization.tex`** (extend pruning theorems) and **`5-resilient_estimation.tex`** (KF error vs \(S\), \(\eta\)).  
- Short pointer in intro contribution bullet; one synthesis paragraph in simulation + conclusion.

---

## Suggested implementation order

1. Branch off updated `main` (e.g. `cursor/kf-chisquare-sparsity-bridge-ac49`).
2. Rewrite system model + KF + \(\chi^2\) (Change 1, theory first).
3. Retarget attack success definition + discriminator matrices; note sim retrain.
4. Write sparsity–observability–error–precision propositions (Change 2).
5. Update resilient section + schematic.
6. Update intro/abstract/conclusion.
7. Re-run / adapt simulations; refresh figures; compile PDF; sanity-check layout (`width=\columnwidth` for schematics).

---

## File / asset checklist

| Path | Notes |
|------|--------|
| `bare_jrnl.tex` | Root; `\input` order Intro → Notation → Model → Attack gen → Detect → Resilient → Sim → Conclusion |
| `image/*.png` | Schematics; size with `width=\columnwidth` (not brittle `scale=`) |
| `new_sim_image/*` | Result plots; several have DPI metadata—prefer width-based include if scales break |
| `MLP_training.png` | Referenced but inside `\iffalse` — ignore unless re-enabled |
| `IEEEabrv.bib`, `IEEEtran.bst`, `mybib.bib` | Bibliography OK on `main` |
| Appendix `8-appendix.tex` | Currently not `\input`’d (`\iffalse`) |

---

## Out of scope unless asked

- Removing blue editorial markup wholesale  
- Switching resilient estimator to weighted \(\ell_1\) (commented block in `5-resilient_estimation.tex`)  
- Unrelated refactor of the GAN architecture beyond detector/estimator consistency  

---

## Done in the sanity-check pass

- Restored missing `image/{GM,WDS,resilient_L2_observer}.png`, IEEE bib support files  
- Fixed overfull figures → column width  
- User confirmed PDF looks good; **PR #1 merged to `main`**
