# Minimal Quarto Blog Setup

A complete, low-maintenance research blog with interactive plots and equations,
deployed to GitHub Pages.

---

## 1. Prerequisites

```bash
# Install Quarto (one-time)
# https://quarto.org/docs/get-started/
# Download the installer for your OS, or on Linux:
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.5.57/quarto-1.5.57-linux-amd64.deb
sudo dpkg -i quarto-1.5.57-linux-amd64.deb

# Python dependencies
pip install jupyter plotly pandas numpy
```

---

## 2. Project Structure

```
my-blog/
├── _quarto.yml          # Site config (theme, nav, metadata)
├── index.qmd            # Homepage / post listing
├── about.qmd            # Optional about page
├── posts/
│   ├── hello-world/
│   │   └── index.qmd    # First post
│   └── my-second-post/
│       ├── index.qmd
│       └── data.csv     # Assets live next to the post
└── .github/
    └── workflows/
        └── publish.yml  # GitHub Actions deploy
```

---

## 3. Site Config — `_quarto.yml`

```yaml
project:
  type: website
  output-dir: _site

website:
  title: "My Research Blog"
  navbar:
    left:
      - href: index.qmd
        text: Posts
      - about.qmd

format:
  html:
    theme: cosmo          # clean default; alternatives: flatly, litera, solar
    highlight-style: github
    code-fold: true       # code blocks collapsed by default — reader can expand
    toc: true
    toc-depth: 3
    number-sections: false
    fig-responsive: true

execute:
  freeze: auto            # only re-execute cells when source changes
```

**`freeze: auto`** is important — it means Quarto caches cell outputs, so you
don't re-run all your notebooks on every build. Outputs are committed to git.

---

## 4. Homepage — `index.qmd`

```markdown
---
title: "My Research Blog"
listing:
  contents: posts
  sort: "date desc"
  type: default
  categories: true
---
```

That's literally it — Quarto auto-generates the post listing from your `posts/`
directory.

---

## 5. A Sample Post — `posts/hello-world/index.qmd`

````markdown
---
title: "Attention is All You Need — Visualised"
date: 2024-06-01
categories: [transformers, interpretability]
description: "Walking through scaled dot-product attention with interactive plots."
---

## The attention formula

Scaled dot-product attention is defined as:

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$

The $\sqrt{d_k}$ scaling prevents the dot products from growing large when
$d_k$ is high, which would push softmax into regions with tiny gradients.

## Visualising attention weights

```{python}
import numpy as np
import plotly.express as px

# Simulate attention weights for a 6-token sequence
np.random.seed(42)
tokens = ["The", "cat", "sat", "on", "the", "mat"]
attn = np.random.dirichlet(np.ones(6), size=6)  # rows sum to 1

fig = px.imshow(
    attn,
    x=tokens, y=tokens,
    color_continuous_scale="Blues",
    labels=dict(x="Key", y="Query", color="Weight"),
    title="Simulated attention matrix",
)
fig.update_layout(width=500, height=450)
fig.show()
```

The plot is fully interactive — hover for exact values, zoom in on specific
token pairs.

## Key insight

Notice that attention is computed *per head* in practice. Each head learns to
attend to different syntactic or semantic relationships independently.
````

---

## 6. GitHub Pages Setup

### 6a. Create the repo

```bash
cd my-blog
git init
git remote add origin git@github.com:YOUR_USERNAME/my-blog.git
```

Add a `.gitignore`:

```
/.quarto/
/_site/
__pycache__/
.jupyter_cache/
```

Note: `_freeze/` (cached outputs) **should** be committed — that's how CI
builds without re-running your code.

### 6b. GitHub Actions workflow — `.github/workflows/publish.yml`

```yaml
name: Publish to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install jupyter plotly pandas numpy

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2

      - name: Publish
        uses: quarto-dev/quarto-actions/publish@v2
        with:
          target: gh-pages
```

### 6c. Enable Pages on GitHub

1. Push to `main`
2. Go to **Settings → Pages**
3. Set source to **Deploy from a branch → `gh-pages`**

First deploy takes ~2 minutes; subsequent pushes are faster because frozen
outputs are cached.

---

## 7. Local authoring loop

```bash
# Start local preview with live reload
quarto preview

# Or render without serving
quarto render

# Render a single post only
quarto render posts/hello-world/index.qmd
```

Browse to `localhost:4200`. Save a `.qmd` and the page refreshes automatically.

---

## 8. Adding a new post

```bash
mkdir posts/my-new-post
touch posts/my-new-post/index.qmd
```

Add the YAML front matter (title, date, categories) and start writing. The
homepage listing updates automatically on next render.

---

## 9. Useful extras (all optional)

| Want | How |
|---|---|
| **Dark mode toggle** | `theme: [cosmo, custom.scss]` + one CSS var override |
| **Comments** | Add `comments: {giscus: {repo: ...}}` to `_quarto.yml` — free, GitHub-backed |
| **Citation / bibliography** | `bibliography: refs.bib` in post front matter, cite with `[@key]` |
| **Custom domain** | Add `CNAME` file to repo root; set in GitHub Pages settings |
| **Mermaid diagrams** | Native in Quarto — just use ` ```{mermaid} ` fences |
| **Observable JS** | Native — ` ```{ojs} ` fences for truly reactive, D3-style plots |

---

## Summary

The entire maintenance surface is:
- **Write** `.qmd` files (Markdown + code cells)
- **`git push`** — Actions handles the rest
- **Never touch** HTML, CSS, or deploy config again

