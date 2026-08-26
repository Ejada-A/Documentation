# Documentation

Final project documentation for Group A's Ejada Cloud Build Track deliverable:
an e-commerce application split into microservices and deployed to Oracle
Kubernetes Engine (OKE) with a fully automated Terraform → Docker → Helm →
ArgoCD pipeline.

## Contents

- **[`final_project_documentation/`](final_project_documentation/)**: the
  LaTeX documentation project itself (source, figures, and the compiled PDF).
  See **[its README](final_project_documentation/README.md)** for the full
  breakdown: repository structure, how to build the PDF, chapter contents,
  the team, and links to the related service repositories.
- **`Arch_and_Design_Draft.pdf`**: the original architecture and design
  draft written during the assessment phase, before implementation began.

## Quick start

```bash
cd final_project_documentation
latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex
```

Produces `final_project_documentation/team-a-project-documentation.pdf`.
