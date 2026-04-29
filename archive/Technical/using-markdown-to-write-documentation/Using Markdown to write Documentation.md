---
title: "Using Markdown to write Documentation"
category: technical
published: 2015-09-28
modified: 2022-08-29
tags: [documentation, markdown]
---

I absolutely cannot stand writing documentation. Well, that's kind of a lie; I don't mind the actual writing, but I get too hung up on trying to make it look pretty. If you—the reader—write your own, then maybe you'll know what I mean.

One of the best things about writing docs that'll be hosted on-line is that the format will be based upon the site that it lives in. Take Github for example; all documentation up there looks the same. This is because they're all written in [Markdown][1]. Markdown is a language syntax that standardises formatting which allows a writer to focus on the content that's being written. If you've never seen a Github readme, check out a few:

- [Munki][2]
- [Autopkg][3]
- Docker container for [Transmission][4]

This blog is also written in markdown. The CSS on this blog is used to apply styles to the various components of a converted markdown document. I'm not going to go into every piece of Markdown syntax as it's not the point of this post, but hopefully you get the idea.		

So how cool would it be to leverage this when writing printed documentation?

Let's get one thing straight; Markdown was build for web rendering. So out of the box, there's no easy way to create a document in Markdown, and export to, say, PDF or Word document.

That's where [Pandoc][5] comes in.

Pandoc is a tool that converts files from one markup format to another. It's a downloadable application for [Mac][6], [Windows][7] and [Linux][8] that runs in the command line in the format of:

```bash
$ pandoc -f ${input-format} ${input-file} \
    -t ${output-format} -o ${output-file}
```

As standard, it uses a predefined format when it spits out a Word document, that looks not unlike Word's default normal.dot. However, with the use of the `--reference-docx` option, you can specify a docx file that contains a custom format in order to format the output of the command.

For example:

```bash
$ pandoc -f markdown ~/Desktop/Documentation.markdown \
    -t docx -o ~/Desktop/Documentation.docx \
    --reference-docx ~/Desktop/Reference.docx
```

This will convert the file "`Documentation.markdown`" into a Word document called "`Documentation.docx`", using the reference file "`Reference.docx`"

So, about these reference docx files. "Where can I get them from" I hear you shout? Well, I've been rather unsuccessful in my attempts to find any. [So I made my own!][file-01] Here's one that looks like a rendered Markdown webpage. Plug this in to the above commands, and your markdown will "accurately" translate into printed documentation!

If you're a Markdown writer/user, and you have a hard time focusing on content rather than design, give this a go.

[1]:	https://daringfireball.net/projects/markdown/
[2]:	https://github.com/munki/munki/blob/master/README.md
[3]:	https://github.com/autopkg/autopkg/blob/master/ReadMe.md
[4]:	https://github.com/wegotoeleven/dockerfiles/blob/master/boot2docker-transmission/README.md
[5]:	www.pandoc.org
[6]:	https://github.com/jgm/pandoc/releases/download/1.15.0.6/pandoc-1.15.0.6-osx.pkg
[7]:	https://github.com/jgm/pandoc/releases/download/1.15.0.6/pandoc-1.15.0.6-windows.msi
[8]:	https://github.com/jgm/pandoc/releases/download/1.15.0.6/pandoc-1.15.0.6-1-amd64.deb
[file-01]: assets/2015-09-28-01.doc
