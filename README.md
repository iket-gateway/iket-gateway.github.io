# Iket Website

This repository contains the public Iket website and the canonical published documentation.

## Docs Ownership

- Product and operator docs now live in `iket-website/docs`
- The website docs are the primary published source for install, operations, plugin, and API guides
- The application repository can still keep local engineering notes, but user-facing documentation should be updated here

## Local Development

Run the site locally with Jekyll:

```bash
bundle install
bundle exec jekyll serve
```

Then open:

```text
http://127.0.0.1:4000
```

## Main Pages

- `/` homepage
- `/docs/` documentation index
- `/about/` project overview

## Publishing Notes

- Add new guides under `docs/`
- Keep permalinks stable once published
- Prefer updating website pages here instead of pointing readers back to repo-only markdown
