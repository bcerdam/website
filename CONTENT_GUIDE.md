# Updating the website

## Profile and CV

Edit `_data/profile.yml` to change the name, summary, email, or profile links. Replace `assets/cv.pdf` with the current public CV PDF.

## News

Add an item to `_data/news.yml` with a date, short text, URL, and an `external` value. External links use `true`; links to pages on this site use `false`. News appears newest first.

## Projects and write-ups

Add one Markdown file under `_projects/` for each preprint, paper, or technical write-up. The project list is sorted by date, oldest first. A project's title links to its `external_url` when present; otherwise it links to the generated project page.

```yaml
---
title: "A project title"
type: "Technical write-up"
date: "2026-10-01"
summary: "A short summary for the project list."
permalink: /projects/a-project-title/
---

Write the full article here in Markdown.
```

For an external preprint or paper, add an `external_url` field. For a technical write-up, put its full text in the Markdown file; its title will link to its own page.

## Preview locally

From the repository root, install Ruby and Bundler if they are not already installed, then run:

```sh
bundle install
bundle exec jekyll serve
```

Open `http://127.0.0.1:4000/` in a browser. Do not open the source `index.html` directly; Jekyll must render its Liquid expressions first.
