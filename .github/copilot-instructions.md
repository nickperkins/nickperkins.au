# Copilot Instructions for `nickperkins.au`

This repository is a personal website/blog built with [Hugo](https://gohugo.io/) and customized with the Bilberry Hugo theme. Follow these guidelines to be productive and consistent when contributing or using AI coding agents in this codebase.

## Architecture Overview
- **Hugo static site**: Content is in `content/`, layouts and shortcodes in `layouts/`, static assets in `static/`, and theme configuration in `hugo.toml`.
- **Theme**: Uses `github.com/Lednerb/bilberry-hugo-theme/v4` as a Hugo module (see `hugo.toml` and `go.mod`).


## Key Workflows
- **Build the site**: Use the [Hugo CLI](https://gohugo.io/getting-started/usage/) (`hugo`) to build the static site. Output is in `public/`.
- **Local development**: Run `hugo server` to serve the site locally with live reload.


## Project Conventions
- **Content organization**: Articles and pages are organized by type and topic under `content/`. Use front matter in markdown files for metadata.
- **Shortcodes**: Custom shortcodes are in `layouts/shortcodes/` (e.g., `github.html`, `imgproc.html`). Use these for embedding GitHub repo cards or processing images in markdown.
- **Theme updates**: Theme is managed as a Go module. Update via `go get` in `go.mod`.
- **No custom build scripts**: All build logic relies on Hugo and standard tools; avoid introducing custom build steps unless necessary.

## Integration Points
- **External JS**: Some JS modules are loaded via CDN as configured in `hugo.toml` (`js_modules`).

## Examples
- **Image processing in markdown**:
  ```markdown
  {{< imgproc img="myimage.jpg" command="Fit" options="600x" alt="Example image" >}}
  Caption here
  {{< /imgproc >}}
  ```

## References
- `hugo.toml`, `go.mod`: Site and theme configuration
- `layouts/shortcodes/`: Custom shortcodes
- `content/`: All site content

