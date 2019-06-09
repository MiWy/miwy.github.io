---
title: "Writing workflow: Pandoc scripts"
date: 2019-06-08T11:02:00-04:00
categories:
  - blog
tags:
  - workflow
---

### tl;dr I uploaded my Pandoc workflow to [GitHub](https://github.com/MiWy/my-pandoc-scripts), feel free to use it

Having an unobtrusive workflow is important.

There are things about writing that just take too much time when done by hand. Text formatting and file converting are two that I especially hate. People expect different things. Some want `.docx`, others `.tex` or `.pdfs`, all complying with a set of strict formatting rules. Often enough a faithful conversion of `.odt` to `.docx` means spending hours on fiddling with margins, fonts, and other details. And do not get me started on 'We want `.tex` files!'.

I think that writing itself should be separated from such mundane worries. While I absolutely love a good-looking document, if you have something to write, then write it and do not worry about the looks. Write in Notepad, in `.txt`, in Word, in TeX, whatever. Worry about the final format and aesthetics later.

[Pandoc](https://pandoc.org) is a command-line universal document converter. Html, `.epub`, LaTeX, Markdown, `.rtf`, `.doc`, anything can be converted to anything. You just need to set some rules.

I uploaded my [Pandoc](https://pandoc.org) scripts on  [GitHub](https://github.com/MiWy/my-pandoc-scripts). I use [Scrivener](http://literatureandlatte.com) for most of my writing, but I believe in writing in [plain text](http://plain-text.co/index.html#introduction). So i export my Scrivener projects to Markdown. I set up the script to accept .md file and a .bib file. As most of the times I need a `.pdf` with a `.docx` or `.tex` reference, Pandoc converts `.md` accordingly.

Example, a draft-looking PDF:
![image-center](/assets/images/pandoc/pandoc1.png){: .align-center}

A little bit better looking PDF:
![image-center](/assets/images/pandoc/pandoc2.png){: .align-center}

The `.md` file just needs to have a title field in the **YAML** metadata.

The script is dead simple on purpose, so that anyone with 101 in BASH can understand it and tweak it. It uses free fonts: ([Lato](http://www.latofonts.com) and [STIX](https://www.stixfonts.org)), as well as `apa` and `spbasic` `csl`/`bst` files. 

If you do not use `.bib` for your bibliography, [Mendeley](https://www.mendeley.com/) and other apps have export to bibtex option. In Mendeley you can make it so that every time you add a record to your library, a .bib file is automatically updated. Symlink that file to Pandoc script's directory and you are good to go.

I know there are more academic-oriented Pandoc templates ([pandoc-scholar](https://pandoc-scholar.github.io)), Pandoc automation tools ([pandocomatic](https://github.com/htdebeer/pandocomatic)), and even Scrivener Ruby script for compiling for pandocomatic ([scrivomatic](https://github.com/iandol/scrivomatic)). While these are great, they are too robust for my needs.

Cheers
