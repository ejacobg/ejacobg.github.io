# Personal Site

This repository contains all the files used to generate the content hosted at [ejacobg.com](https://ejacobg.com).

## Hugo Update/Install

Hugo homepage: https://gohugo.io/

I'm currently installing Hugo via Chocolatey.

```bash
choco upgrade chocolatey
choco list
choco upgrade hugo-extended
hugo version
```

## Build Site

Run locally: `hugo server`

Include drafts: `hugo server -D`

Publish site: `hugo`

- Update build configurations via `hugo.toml` (previously `config.toml`).

## Adding New Posts

Add a new directory or Markdown file to the [`content/`](/content/) directory.

I'm currently just adding Markdown files, and throwing all images/other content into [`static/`](/static/).

## Updating the Theme

Just read the Hugo documentation.
