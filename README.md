# automated_attack_generative_model

IEEE-style paper sources for *Algorithm Design for Resilient Cyber-Physical Systems using an Automated Attack Generative Model*.

## Build

```bash
pdflatex bare_jrnl.tex && bibtex bare_jrnl && pdflatex bare_jrnl.tex && pdflatex bare_jrnl.tex
```

Baseline PDF: `artifacts/initial_paper_sanity_check.pdf`

## Next work

See **[HANDOVER.md](HANDOVER.md)** for the approved plan:
1. Replace LS observer + residual BDD with **Kalman filter + χ² detector**
2. Quantitative bridge: **sparsity ↔ observability ↔ estimation error ↔ detection precision**

Simulation code: https://github.com/ZYblend/Physics-guided_GAN_Attack_Generation
