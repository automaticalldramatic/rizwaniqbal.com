# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static website for rizwaniqbal.com built using Hugo, a static site generator. The site features a personal blog with technical posts about software engineering, team organization, and developer tools.

## Development Commands

### Local Development
```bash
# Start Hugo development server with live reload and draft posts
hugo server -D
```
The development server runs at http://localhost:1313/ with live reloading enabled.

### Content Creation
```bash
# Create a new blog post
hugo new posts/my-new-post.md
```

### Building
```bash
# Build for production with minification
hugo --minify
```

## Project Architecture

### Directory Structure
- `content/` - Markdown content files
  - `content/posts/` - Blog post markdown files
  - `content/_index.md` - Homepage content
- `layouts/` - Custom Hugo templates and overrides
  - `layouts/partials/` - Partial templates (includes analytics.html for GA4)
  - `layouts/index.html` - Custom homepage template
- `themes/hermit/` - Hugo theme (git submodule)
- `static/` - Static assets (images, etc.)
- `config.toml` - Hugo configuration file
- `firebase.json` - Firebase Hosting configuration

### Theme and Customization
- Uses the "hermit" Hugo theme as a git submodule
- Custom layouts in `layouts/` override theme defaults
- Google Analytics 4 configured in `layouts/partials/analytics.html`
- Site configuration in `config.toml` includes social links, author info, and theme customization

### Deployment
- Automated deployment via GitHub Actions (`.github/workflows/main.yml`)
- Deploys to Firebase Hosting on tag push (format: `v*`)
- Build-only on pull requests to master branch
- Requires Hugo Extended version

### Git Submodules
The project uses git submodules for the theme:
```bash
# Initialize and update submodules when setting up
git submodule init
git submodule update
```

## Content Guidelines

Blog posts are stored in `content/posts/` as Markdown files with frontmatter. The site focuses on technical content related to software engineering, team management, and developer productivity.