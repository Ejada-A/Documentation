# E-Commerce Microservices on Oracle Kubernetes Engine

**Final Project Documentation: Group A, Ejada Cloud Build Track**

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

The system as it is actually implemented across the six `Ejada-A` repositories
(four backend microservices, the Next.js frontend, and the Terraform/Kubernetes/Helm/
ArgoCD infrastructure), read directly from source, not summarized from the team's
earlier Architecture & Design Draft. Every technical claim traces to a specific file
at a specific commit (listed in Chapter 5, "Sources Reviewed"). Where the draft
and the repositories disagree, the documentation says so explicitly and follows the
repositories (Chapter 2, "Design Draft vs. Implementation"), including gaps such as
the request-level authorization and payment-confirmation gaps found by the initial
QA pass, their later remediation, and unfinished storefront pages (Chapter 6,
"Limitations and Future Improvements").

## Repository structure

```
final_project_documentation/
├── main.tex                        # Main LaTeX entry point, compile this file
├── team-a-project-documentation.pdf   # Compiled output (regenerate with the commands below)
├── .latexmkrc                      # Sets the compiled output's filename (jobname)
├── .gitignore
│
├── Sections/                    # Document class, packages, macros, title page, front matter
│   ├── 01_class_and_packages.tex
│   ├── 02_new_commands.tex      # Custom commands, TikZ styles, heading/pagestyle/callout-box setup
│   ├── 03_Authors_info.tex      # Title, subtitle, author, team name (edit metadata here)
│   ├── 04_titlepage.tex         # Title page layout (logos, metadata table, team roster)
│   ├── 09_abstract.tex          # Technical Summary
│   └── 10_toc_figures_tables.tex# Table of contents
│
├── chapters/                    # The actual documentation content, one file per chapter
│   ├── overview.tex             # Ch.1: Project Overview
│   ├── context_and_sources.tex  # Ch.2: System Architecture
│   ├── security.tex             # Ch.3: Security Considerations
│   ├── approach.tex             # Ch.4: Infrastructure and Deployment (Terraform, CI/CD, GitOps, deployment guide)
│   ├── findings.tex             # Ch.5: System Components and Runtime Behavior
│   ├── takeaways.tex            # Ch.6: Limitations and Future Improvements
│   ├── next_steps.tex           # Ch.7: Troubleshooting
│   └── appendix.tex             # Appendix A/B/C: Glossary, Network Configuration Reference, Command Reference
│
├── Figures/                     # All images referenced by \includegraphics
│
├── tablesFiguresTemplates.tex   # Standalone reusable table/figure snippet library (not \include'd)
└── prompts/, agent-transfer-instructions.txt, figure-style-conversion-prompt.txt
                                  # AI-agent authoring workflow tooling, gitignored, not
                                  # part of the documentation itself
```

## Building the PDF

**Prerequisites:** a LaTeX distribution (MiKTeX or TeX Live) with `latexmk`, `pdflatex`,
and the standard packages this document uses (`tikz`, `listings`, `tabularx`, `longtable`,
`booktabs`, `titlesec`, `fancyhdr`, `tcolorbox`, `hyperref`, `adjustbox`, and others
declared in `Sections/01_class_and_packages.tex`). A first compile will prompt
MiKTeX/TeX Live to auto-install anything missing.

The output filename is controlled by `.latexmkrc` (`$jobname = 'team-a-project-documentation'`),
so both commands below produce `team-a-project-documentation.pdf` even though the
source file is `main.tex`.

**Recommended (one command, handles all passes automatically):**

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

**Cleaning up build artifacts** after a successful compile (safe to run any time;
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
| **1. Project Overview** | Objectives and scope, in/out-of-scope boundaries, and project ownership |
| **2. System Architecture** | Repository-by-repository breakdown, domain ownership (and the shared-database pattern behind it), technology stack by evidence, deployed architecture diagrams, the network design as currently coded, and a Design Draft vs. Implementation reconciliation table |
| **3. Security Considerations** | The security baseline at the reviewed commits: network access, authentication, the original payment-confirmation gap, secrets management, IAM, the data layer, and pod hardening, with the newer remediation evidence cross-referenced |
| **4. Infrastructure and Deployment** | The Terraform `network`/`oke` module composition, CI/CD and bootstrap workflows, container and health-check design, deployment and configuration references, plus the complete QA and team-remediation record culminating in a 93/95 production retest |
| **5. System Components and Runtime Behavior** | A component reference, path-based routing, a discrepancy between the two committed autoscaling policies, frontend feature completeness (what works vs. what ships as an unfinished scaffold), and the commits this documentation was verified against |
| **6. Limitations and Future Improvements** | Current limitations grounded in the security/completeness findings above, recommended next steps, monitoring recommendations, and open questions |
| **7. Troubleshooting** | Common symptoms, diagnostic commands, and likely causes |
| **Appendix A** | Glossary of terms |
| **Appendix B** | Network Configuration Reference: every NSG rule as currently coded, including which "optional" rules are actually enabled |
| **Appendix C** | Command reference |

## Key figures

All architecture diagrams are native TikZ figures drawn directly in the `.tex`
sources (following the palette and node styles defined in
`Sections/02_new_commands.tex`), not image files:

- **Deployed architecture** (`chapters/context_and_sources.tex`, `fig:topology`): the full VCN, all four
  subnets, both gateways, and the OCIR pull path, as deployed.
- **Application-layer access structure** (`chapters/context_and_sources.tex`, `fig:app-access`): Ingress
  routing to all five workloads and the shared MongoDB instance.
- **Terraform module interfaces** (`chapters/approach.tex`, `fig:network-module` and `fig:oke-module`):
  the typed input/output contract each module exposes, and how `oke` consumes `network`'s outputs.

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
| [`auth-service`](https://github.com/Ejada-A/auth-service) | Auth domain: accounts, credentials, roles |
| [`products-service`](https://github.com/Ejada-A/products-service) | Product domain: catalog |
| [`orders-service`](https://github.com/Ejada-A/orders-service) | Orders domain |
| [`payments-service`](https://github.com/Ejada-A/payments-service) | Payments domain |
| [`ecomm-ui`](https://github.com/Ejada-A/ecomm-ui) | Next.js frontend (backend-for-frontend) |
| [`Terraform-code`](https://github.com/Ejada-A/Terraform-code) | Network + OKE Terraform modules, Kubernetes manifests, Helm chart, ArgoCD config |

## Editing this documentation

- **Chapter content** lives entirely in `chapters/*.tex` (one file per chapter);
  edit directly.
- **Title, subtitle, and metadata** (title page text, document type) are set via the
  commands defined in `Sections/03_Authors_info.tex`; change them there rather than
  hardcoding text in `Sections/04_titlepage.tex`.
- **Adding a figure or table**: drop the image in `Figures/`, then reference it with
  `\includegraphics{Figures/<file>.png}` inside a `figure` environment in the relevant
  chapter. Reusable table/figure skeletons are available in `tablesFiguresTemplates.tex`.
- After any edit, recompile with the `latexmk` command above and check the log for new
  `Overfull \hbox` warnings or undefined references before considering the change done.
