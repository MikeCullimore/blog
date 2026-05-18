# My blog

A place to share what I'm learning.

Deployed at https://mikecullimore.github.io/blog/

Built with [Quarto](https://quarto.org/)

## Local authoring loop

Create a new post:

```bash
mkdir posts/my-new-post
touch posts/my-new-post/index.qmd
```

add the YAML front matter (title, date, categories) and start writing. The
homepage listing updates automatically on next render.

Start local preview with live reload

```bash
quarto preview
```

Or render without serving

```bash
quarto render
```

Render a single post only

```bash
quarto render posts/hello-world/index.qmd
```

## How to publish

```bash
quarto publish gh-pages
```

This will build and push via a GitHub action.
