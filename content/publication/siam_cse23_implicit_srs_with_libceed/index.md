---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Performance-Portable Implicit Scale-Resolving Compressible Flow Using libCEED"
authors: ["James Wright", "Jed Brown", "Kenneth Jansen", "Leila Ghaffari"]
date: 2023-03-28T09:22:48-06:00
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: 2023-03-28T09:22:48-06:00

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["1"]

# Publication name and optional abbreviated publication name.
publication: "SIAM Conference on Computational Science and Engineering 2023"
publication_short: "SIAM CSE 2023"

abstract: "Unstructured mesh CFD solvers with second order accurate spatial discretizations are flexible and ubiquitous in industry. Scale-resolving simulation techniques are viewed as the future for predictive fluid simulations, but standard technology is inefficient when repurposed for scale-resolving simulation. We show that greater efficiency can be gained using a stabilized-continuous-Galerkin finite element method with high-order basis functions on GPUs, using implicit time-stepping schemes. We explore trade-offs in the discretization and solver using a GPU implementation based on libCEED and PETSc, which enables portable performance from a single source code. We demonstrate accuracy, stability, and efficiency of our solver on turbulent compressible boundary layer flows."

# Summary. An optional shortened abstract.
summary: "Presentation of developments in using libCEED and PETSc for compressible flow simulation. The resulting code is portable to diverse hardware and performance is demonstrated."

tags: ["libCEED", "PETSc"]
categories: []
featured: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_pdf:
url_code:
url_dataset:
url_poster:
url_project:
url_slides:
url_source:
url_video:

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---
