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

Note: your `CNAME` file needs to be present in the `docs/` directory (or wherever you publish your output to) in order for the site to be published on your domain. Hugo will not replace this file if you happen to delete and recreate the `docs/` directory.

## Adding New Posts

Add a new directory or Markdown file to the [`content/`](/content/) directory.

You can use the [`hugo new`](https://gohugo.io/commands/hugo_new/) command for this.

I'm currently just adding Markdown files, and throwing all images/other content into [`static/`](/static/).

## Updating the Theme

Just read the Hugo documentation.
