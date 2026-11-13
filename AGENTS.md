# AGENTS.md

Instructions for AI coding agents (OpenCode and others) working in this repository.
Read this file fully before making any change.

## 1. Project overview

- This repository is a **website hosted on GitHub Pages**.
- The site is built by **Jekyll** (GitHub Pages' built-in static site generator). There is no server, database, or backend.
- Content pages live in **`_posts/`**. Each file there is one page/post.
- Pushing to the publishing branch (usually `main`) **automatically deploys the live site**. Treat every commit to that branch as a production release.

> TODO (owner): fill in the values below so agents don't have to guess.
> - Live URL: `https://<username>.github.io/<repo>/`
> - Publishing branch: `main`
> - Theme / layout used: `<theme name>`

## 2. Repository layout

```
.
├── _posts/          # Content pages (main place agents work). Markdown/HTML with front matter.
├── _config.yml      # Site-wide configuration (see rules below)
├── index.md|html    # Home page (if present)
└── AGENTS.md        # This file
```

This repo is intentionally minimal. There is **no** `Gemfile`, `assets/`, `_sass/`, `_data/`, or `_includes/` folder. Do not reference, assume, or create any of them. Check what is actually present before assuming a structure, and never create a new top-level folder or file such as a `Gemfile` without being asked.

## 3. Working with `_posts/`

### File naming (strict)
Jekyll only recognizes posts named exactly:

```
YYYY-MM-DD-title-in-kebab-case.md
```

- Lowercase letters, numbers, and hyphens only. No spaces, underscores, or accented characters in the slug.
- The date must be a valid date. **A date in the future makes Jekyll skip the post** (it will not appear on the live site) unless `future: true` is set in `_config.yml`. Use today's date or earlier unless the user explicitly wants a scheduled post.
- Extension is `.md` (preferred) or `.markdown` / `.html`.
- Never rename an existing post without asking. The filename (date + slug) determines the public URL, so renaming breaks existing links.

### Front matter (required)
Every post must begin with YAML front matter between `---` lines:

```yaml
---
layout: post
title: "Post title"
date: 2026-01-31 10:00:00 +0000
categories: [category-one]
tags: [tag-one, tag-two]
---
```

- Match the front matter fields and style of the **existing posts**. Read two or three before writing a new one.
- Quote titles that contain a colon, quotes, or special characters.
- Use spaces for indentation in YAML, never tabs.
- A missing or malformed front matter block means the file is published as raw text or not at all.
- Do not invent new front matter keys unless a layout actually uses them.

### Content conventions
- Write in Markdown. Use raw HTML only when Markdown cannot do the job.
- Use a single `#` heading only if the layout does not already render the title; otherwise start body headings at `##`.
- There is no `assets/` folder. Before adding an image or file, check how existing posts reference them (inline links, external URLs, or a folder in the repo root) and follow that pattern. If no pattern exists, ask the user where files should go instead of creating a folder.
- Always add meaningful `alt` text to images.
- Use `relative_url` / `absolute_url` filters or the existing link pattern for internal links so the site works under the `/<repo>/` base path on project pages.
- Do not paste large binary files or unoptimized images into the repo.
- Liquid syntax (`{{ }}` and `{% %}`) inside post content is processed. To show these literally in code samples, wrap them in `{% raw %}` ... `{% endraw %}`.

## 4. Things that can break the live site

Do **not** do any of the following without explicit user approval:

1. Edit `_config.yml`. A mistake there can break the whole build, and there is no local build to catch it first.
2. Change `baseurl` or `url` in `_config.yml`.
3. Delete, rename, or move files in `_posts/` (breaks URLs and inbound links).
4. Add Jekyll plugins or gems that are not on the [GitHub Pages dependency list](https://pages.github.com/versions/). Unsupported plugins silently fail on GitHub Pages.
5. Add or remove a `CNAME` file (controls the custom domain).
6. Change layouts or the theme (if any exist), since these affect every page.
7. Run any state-changing git command (see section 6), including force-pushing, rewriting history, or changing the publishing branch.
8. Commit secrets, API keys, tokens, or personal data. Everything in this repo is **public**.

## 5. Build and verification

- **There is no local build.** The owner does not run Jekyll locally. Do not run `bundle`, `jekyll build`, or `jekyll serve`, and do not install Ruby or gems.
- GitHub Pages builds the site automatically after each push. Nothing is checked before it goes live, so **be careful and verify by reading**.

Before reporting a task as done, re-read every file you changed and check:
1. The filename matches `YYYY-MM-DD-slug.md` and the date is not in the future.
2. Front matter is present, valid YAML, and matches existing posts.
3. Every image and internal link path points to a file that actually exists in the repo.
4. Liquid tags (`{% %}`, `{{ }}`) are balanced and correctly spelled.

Never claim the site builds correctly, since you cannot verify it. After the owner pushes, a failed build shows up in the repository's **Actions** tab or under **Settings > Pages**. If the owner reports a build error, ask for the error message and fix the cause.

## 6. Git workflow

- **The user handles all git.** Your job is to make file edits only.
- Do **not** run git commands that change state: no `git add`, `commit`, `push`, `pull`, `merge`, `rebase`, `reset`, `checkout`, `stash`, or branch operations. Read-only commands such as `git status`, `git diff`, and `git log` are fine if you need context.
- Do not create commits or pull requests, and do not add commit messages or attribution lines.
- Pushing to the publishing branch deploys the live site, so the user decides when that happens. When you finish, list the files you changed so the user can review them and commit themselves.

## 7. Style and quality rules

- **Match existing style.** Mirror tone, heading structure, front matter, and formatting of current posts.
- **Do not rewrite content you were not asked to change.** Keep edits minimal and scoped.
- Preserve the author's voice. When asked to edit prose, fix what was requested and leave the rest.
- Use UTF-8 and LF line endings. End files with a single newline.
- Fix typos you notice only when asked, or mention them instead of silently changing them.
- Keep file and slug names in English kebab-case unless the existing site does otherwise.
- Accessibility: descriptive link text (not "click here"), alt text on images, logical heading order.

## 8. Agent behavior

- **Read before writing.** Look at existing posts and `_config.yml` before creating content.
- **Ask when ambiguous**, especially about dates, categories, URLs, and anything affecting the live site.
- **Do not guess facts.** If a post needs facts, links, or quotes you don't have, ask or leave a clearly marked placeholder for the user.
- **Do not touch** `.git/`.
- Report what changed at the end: list the files created or edited and the checks you did by reading them (not a build).

## 9. Quick checklist for a new post

- [ ] File is in `_posts/` and named `YYYY-MM-DD-slug.md`
- [ ] Date is valid and not in the future
- [ ] Front matter is present, valid YAML, and matches existing posts
- [ ] Images follow the existing posts' pattern, have alt text, and the paths resolve
- [ ] Internal links use the site's existing link pattern
- [ ] Re-read the file once more for YAML and Liquid mistakes
- [ ] No secrets or private data included
