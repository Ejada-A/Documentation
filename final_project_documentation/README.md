# E-Commerce Microservices on Oracle Kubernetes Engine

**Final Project Documentation — Group A — Ejada Cloud Build Track**

This repository contains the complete written documentation for Group A's final-week
deliverable of the Ejada Cloud Build Track internship: taking an e-commerce application,
splitting it into independently deployable microservices, and deploying that
architecture to Oracle Kubernetes Engine (OKE) with a fully automated Terraform → Docker
→ Helm → ArgoCD pipeline.

The documentation is written in LaTeX and compiles to a single PDF, `team-a-project-documentation.pdf`.

---

## Table of Contents

- [What this documents](#what-this-documents)
- [Repository structure](#repository-structure)
- [Building the PDF](#building-the-pdf)
- [Documentation contents](#documentation-contents)
- [Key figures](#key-figures)
- [Team](#team)
- [Related repositories](#related-repositories)
- [Editing this documentation](#editing-this-documentation)

---

## What this documents

Two things, as one cross-referenced write-up rather than two separate ones:

1. **The assessment phase** — how the application's description was analyzed into four
   independent domains (Auth, Product, Cart, Purchase), and the network, scalability,
   routing, registry, and storage decisions made before any infrastructure was built.
2. **The implementation and deployment** — the Terraform modules, Kubernetes manifests,
   Helm chart, and GitOps pipeline that actually build, containerize, and continuously
   deploy the five resulting services (four backend microservices plus the frontend) to
   a live OKE cluster.

Every claim in the documentation is traceable to a specific file in a specific
repository under the [`Ejada-A`](https://github.com/Ejada-A) GitHub organization, the
team's own architecture diagrams, or the assessment/design documentation written before
implementation began — not to unverified assertions. Where that verification stops
short of a live cluster session, the documentation says so explicitly (see
Chapter 4, "Verification Performed").

## Repository structure

```
final_project_documentation/
├── main.tex                        # Main LaTeX entry point — compile this file
├── team-a-project-documentation.pdf   # Compiled output (regenerate with the commands below)
├── .latexmkrc                      # Sets the compiled output's filename (jobname)
├── .gitignore
│
├── Sections/                    # Document class, packages, macros, title page, front matter
│   ├── 01_class_and_packages.tex
│   ├── 02_new_commands.tex      # Custom commands, TikZ styles, code-listing styles
│   ├── 03_Authors_info.tex      # Title, subtitle, author, team name — edit metadata here
│   ├── 04_titlepage.tex         # Title page layout (logos, metadata table, team roster)
│   ├── 09_abstract.tex          # Summary chapter
│   └── 10_toc_figures_tables.tex# Table of contents, abbreviations, list of figures/tables
│
├── chapters/                    # The actual documentation content, one file per chapter
│   ├── overview.tex             # Ch.1 — Introduction and Scope
│   ├── context_and_sources.tex  # Ch.2 — Background and Context (network + app-layer design)
│   ├── approach.tex             # Ch.3 — Implementation Approach (Terraform, CI/CD, GitOps)
│   ├── findings.tex             # Ch.4 — Deployment Results and Verification
│   ├── takeaways.tex            # Ch.5 — Conclusions
│   ├── next_steps.tex           # Ch.6 — Recommendations and Next Steps
│   ├── appendix.tex             # Glossary and command reference
│   └── bibliography.tex         # References (see note in the chapter itself)
│
├── Figures/                     # All images referenced by \includegraphics
├── References/
│   └── myref.bib                # Bibliography database (intentionally empty — see Ch. 6)
│
├── tablesFiguresTemplates.tex   # Standalone reusable table/figure snippet library (not \include'd)
└── prompts/, agent-transfer-instructions.txt, figure-style-conversion-prompt.txt
                                  # AI-agent authoring workflow tooling — gitignored, not
                                  # part of the documentation itself
```

## Building the PDF

**Prerequisites:** a LaTeX distribution (MiKTeX or TeX Live) with `latexmk`, `pdflatex`,
and the standard packages this document uses (`tikz`, `listings`, `tabularx`, `booktabs`,
`acronym`, `hyperref`, `adjustbox`, and others declared in
`Sections/01_class_and_packages.tex`). A first compile will prompt MiKTeX/TeX Live to
auto-install anything missing.

The output filename is controlled by `.latexmkrc` (`$jobname = 'team-a-project-documentation'`),
so both commands below produce `team-a-project-documentation.pdf` even though the
source file is `main.tex`.

**Recommended — one command, handles all passes automatically:**

```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex
```

**Fallback, if `latexmk` isn't available** (`.latexmkrc` isn't read by plain `pdflatex`,
so pass `-jobname` explicitly to get the same output filename):

```bash
pdflatex -interaction=nonstopmode -halt-on-error -jobname=team-a-project-documentation main.tex
pdflatex -interaction=nonstopmode -halt-on-error -jobname=team-a-project-documentation main.tex
pdflatex -interaction=nonstopmode -halt-on-error -jobname=team-a-project-documentation main.tex
```

(Multiple passes are needed to resolve the table of contents, cross-references, list of
figures, and list of tables.)

**Cleaning up build artifacts** after a successful compile (safe to run any time —
`team-a-project-documentation.pdf` is untouched):

```bash
rm -f team-a-project-documentation.aux team-a-project-documentation.log \
      team-a-project-documentation.fls team-a-project-documentation.fdb_latexmk \
      team-a-project-documentation.toc team-a-project-documentation.lof \
      team-a-project-documentation.lot team-a-project-documentation.out
rm -f Sections/*.aux chapters/*.aux
```

These artifacts are also excluded via `.gitignore`, so they won't get committed even if
you forget to clean them.

## Documentation contents

| Chapter | Covers |
| --- | --- |
| **1. Introduction and Scope** | The assignment, objective, scope/boundaries, stakeholder map, and how the documentation is organized |
| **2. Background and Context** | Domain-ownership analysis; the full OKE network design (four subnets, gateways, route tables, every NSG rule); the security design; and the application-layer design decisions (HPA strategy, routing, container registry, storage) from the assessment phase |
| **3. Implementation Approach** | The Terraform `network`/`oke` module composition, configuration inputs, the `for_each`-driven rule-flattening pattern, the CI → OCIR → Helm → ArgoCD promotion pipeline, container image build and health-check design, and environment teardown |
| **4. Deployment Results and Verification** | The system as actually deployed — architecture diagrams, path-based routing, autoscaling and resource sizing, the GitOps pipeline in practice, what was and wasn't independently verified |
| **5. Conclusions** | Main conclusion, strongest supporting evidence, biggest uncertainty, and confidence level |
| **6. Recommendations and Next Steps** | Further testing needed, recommended actions, monitoring indicators, open questions |
| **Appendix** | Glossary of terms and a command reference |

## Key figures

- **Deployed architecture** (`Figures/week4Architecture.png`) — the full VCN, all four
  subnets, both gateways, and the OCIR pull path, as actually deployed.
- **Application-layer access structure** (`Figures/appAccessStructure.png`) — Ingress
  routing to all five workloads and the shared MongoDB instance.
- **Terraform module interfaces** (`Figures/networkModuleInterface.png`,
  `Figures/okeModuleInterface.png`) — the typed input/output contract each module
  exposes, and how `oke` consumes `network`'s outputs.

## Team

**Group A**

| | |
|---|---|
| Basel Alaa | Ali Yousef |
| Yara Adel Ejada | Chris |
| Abdelrahman Darwish | Habiba Fawzy |
| Ali Hamad | Kholosy |
| Ali Tallawy | Peter Kameel |
| Salma | Beshoy |

## Related repositories

All under the [`Ejada-A`](https://github.com/Ejada-A) GitHub organization:

| Repository | Role |
| --- | --- |
| [`auth-service`](https://github.com/Ejada-A/auth-service) | Auth domain — accounts, credentials, roles |
| [`products-service`](https://github.com/Ejada-A/products-service) | Product domain — catalog |
| [`orders-service`](https://github.com/Ejada-A/orders-service) | Cart/Orders domain |
| [`payments-service`](https://github.com/Ejada-A/payments-service) | Purchase/Payments domain |
| [`ecomm-ui`](https://github.com/Ejada-A/ecomm-ui) | Next.js frontend (backend-for-frontend) |
| [`Terraform-code`](https://github.com/Ejada-A/Terraform-code) | Network + OKE Terraform modules, Kubernetes manifests, Helm chart, ArgoCD config |

## Editing this documentation

- **Chapter content** lives entirely in `chapters/*.tex` — one file per chapter, edit
  directly.
- **Title, subtitle, and metadata** (title page text, document type) are set via the
  commands defined in `Sections/03_Authors_info.tex` — change them there rather than
  hardcoding text in `Sections/04_titlepage.tex`.
- **Adding a figure or table**: drop the image in `Figures/`, then reference it with
  `\includegraphics{Figures/<file>.png}` inside a `figure` environment in the relevant
  chapter. Reusable table/figure skeletons are available in `tablesFiguresTemplates.tex`.
- **Abbreviations**: add new ones to the `acronym` environment in
  `Sections/10_toc_figures_tables.tex` — only add an abbreviation if it's actually used
  somewhere in the body text, and remove one from the list if it stops being used.
- After any edit, recompile with the `latexmk` command above and check the log for new
  `Overfull \hbox` warnings or undefined references before considering the change done.
