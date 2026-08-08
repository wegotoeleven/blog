---
title: "Bringing this blog back from the dead"
date: 2026-08-09T00:45:52+01:00
categories: ["technical"]
tags: ["hugo", "github-pages", "claude-code"]
---

This blog has been dead for years. It wasn't deliberate, more the sort of dead that happens when you keep meaning to sort it out and then don't, for the better part of a decade. At some point I'd archived everything into a flat folder of Markdown files and images and left it there. Today I actually finished the job.

The site now runs on [Hugo](https://gohugo.io/) with the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. Hugo takes a folder of Markdown files and turns them into a full static website at build time, and PaperMod is a ready-made visual theme for it, so I didn't have to design anything from scratch, which was exactly the problem that had killed this blog's relaunch more than once before.

Migrating the old export was mostly mechanical: every post became a Hugo "page bundle", meaning its own folder containing an `index.md` and any images it needs, sitting right next to each other. Two sections came out of it: `technical` (this stuff) and `travelling`, an archive of a five-month trip round South East Asia in 2011 that had never been properly online before.

The genuinely fiddly part was dates. The technical posts were fine, with dates already sitting in the front matter. Most of the 35-odd travel posts weren't so simple: no date field, no dated photo, and no useful git history, because the whole lot had been dumped into the repo in a single bulk-import commit years after they were written. Working out when they'd actually happened meant:

- Pulling dates out of the handful of posts that did have a dated photo attached
- Cross-checking offhand details in the text, like "Thursday 17th" and "Happy Saturday", against the 2011 calendar, which turned out to line up exactly
- Going back to the source and asking for the actual trip itinerary, then using that to pin down everything else

Every date I had to guess got flagged as such right in the post's own front matter (`dateApprox: true`) rather than quietly presented as fact. A wrong-looking date is still more useful than a suspiciously precise one nobody can trust.

Getting it live was the easy part by comparison: a GitHub Actions workflow builds the Hugo site and deploys it to GitHub Pages on every push to `main`, with the custom domain wired up through a CNAME record. The only hiccup since was GitHub deprecating Node.js 20 on its Actions runners mid-migration, which meant bumping a couple of pinned action versions to clear the warnings, though one of them still doesn't have a fix upstream, so that warning's stuck until GitHub ships one.
