# CV source (reconciled)

This directory holds the authoritative LaTeX source for the CV published at
https://pankajgarkoti.github.io/cv.pdf.

## Provenance

The previous Overleaf-based CV used a stale body (a `co/re` founder role, a
BCAS research apprenticeship, PlanWeaver/Vance projects and an extra reference).
The previously-live CV was rebuilt from the real live PDF
(`/tmp/opencode/cv-original-live.pdf`, SHA-256 `0f3e3821…`), using the
`prometheus_cv` Overleaf scaffold only for class/style. Every content section
was rewritten to match that PDF, then exactly three additions were applied:

1. Zonko Labs — Member of Technical Staff (Aug–Nov 2026) above Jouster.
2. Nishank Jain (Zonko AI) in References.
3. mAGI in Projects.

Compile with **XeLaTeX** (`main.tex` is the root file). The reconciled source
was also uploaded to the Overleaf project `cv`
(60001c8a176331e60ab9c9e6); `cv.pdf` in the repository root is the PDF
downloaded from that clean compile (2 warnings, no errors, 2 pages).
