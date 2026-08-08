# blog.wegotoeleven.xyz

Built with [Hugo](https://gohugo.io/) using the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to GitHub Pages via GitHub Actions on every push to `main`.

## Structure

- `content/technical/<slug>/index.md` — technical posts
- `content/travelling/<slug>/index.md` — 2011 South East Asia travel diary (migrated from the old `archive/` export)
- Posts with images/attachments are Hugo [page bundles](https://gohugo.io/content-management/page-bundles/): the images live alongside `index.md` in the same folder and are referenced with relative paths.

## Writing a new post

```
hugo new content/technical/my-new-post/index.md
```

Fill in the front matter (`title`, `date`, `categories`, `tags`), write the post, then:

```
hugo server -D
```

to preview locally at `http://localhost:1313`, before pushing to `main`.

## Deployment

GitHub Actions (`.github/workflows/hugo.yml`) builds and deploys on push to `main`. Repo Settings → Pages → Source must be set to "GitHub Actions". Custom domain is set via `static/CNAME` plus a DNS CNAME record for `blog.wegotoeleven.xyz` pointing at `wegotoeleven.github.io`.
