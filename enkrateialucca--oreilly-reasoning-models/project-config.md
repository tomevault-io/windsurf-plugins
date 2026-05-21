---
trigger: always_on
description: Create professional, engaging slides in HTML/Markdown format using the remark framework from a course outline. The slides should be presentation-ready, visually organized, and structured to facilitate effective teaching.
---

# Course Slide Creator - Project Instructions

## Project Goal
Create professional, engaging slides in HTML/Markdown format using the remark framework from a course outline. The slides should be presentation-ready, visually organized, and structured to facilitate effective teaching.

## Slide Structure & Format
- Create slides in HTML format with Remark.js, using markdown syntax within the `<textarea id="source">` element
- Use three dashes `---` to separate individual slides
- Include the proper HTML boilerplate and styling as shown in the example
- Create a cohesive visual style that enhances learning
- Follow modern slide design principles: clean layout, readable fonts, appropriate white space
- Consider this documentation from remark

A simple, in-browser, markdown-driven slideshow tool targeted at people who know their way around HTML and CSS, featuring:

*   Markdown formatting, with smart extensions
*   Presenter mode with markdown formatted speaker notes and cloned slideshow view
*   Syntax highlighting, supporting a range of languages
*   Slide scaling, thus similar appearance on all devices / resolutions
*   Simple markdown templates for customized slides
*   Touch support for smart phones and pads, i.e. swipe to navigate slides

Check out [this remark slideshow](https://remarkjs.com/) for a brief introduction.

To render your Markdown-based slideshow on the fly, checkout [Remarkise](https://gnab.github.io/remark/remarkise).

### Getting Started

[](https://github.com/gnab/remark#getting-started)

It takes only a few, simple steps to get up and running with remark:

1.  Create an HTML file to contain your slideshow (see boilerplate below)
2.  Open the HTML file in a decent browser
3.  Edit the Markdown and/or CSS styles as needed, save and refresh!
4.  Press `C` to clone a display; then press `P` to switch to presenter mode. Open help menu with `h`.

See any of the boilerplate-\*.html files (the -local one requires building remark first), or just copy the boilerplate HTML below to start:

<!DOCTYPE html\>
<html\>
  <head\>
    <title\>Title</title\>
    <meta charset\="utf-8"\>
    <style\>
      @import url(https://fonts.googleapis.com/css?family=Yanone+Kaffeesatz);
      @import url(https://fonts.googleapis.com/css?family=Droid+Serif:400,700,400italic);
      @import url(https://fonts.googleapis.com/css?family=Ubuntu+Mono:400,700,400italic);

      body { font-family: 'Droid Serif'; }
      h1, h2, h3 {
        font-family: 'Yanone Kaffeesatz';
        font-weight: normal;
      }
      .remark-code, .remark-inline-code { font-family: 'Ubuntu Mono'; }
    </style\>
  </head\>
  <body\>
    <textarea id\="source"\>

class: center, middle

# Title

---

# Agenda

1. Introduction
2. Deep-dive
3. ...

---

# Introduction

    </textarea\>
    <script src\="https://remarkjs.com/downloads/remark-latest.min.js"\>
    </script\>
    <script\>
      var slideshow \= remark.create();
    </script\>
  </body\>
</html\>

### How To Use remark

[](https://github.com/gnab/remark#how-to-use-remark)

The [wiki](http://github.com/gnab/remark/wiki) pages contain all the how-to, templating, and API help.

### Real-world remark slideshows

[](https://github.com/gnab/remark#real-world-remark-slideshows)

On using remark:

*   [The Official remark Slideshow](https://remarkjs.com/)
*   [Coloured Terminal Listings in remark](https://joshbode.github.io/remark/ansi.html) by [joshbode](https://github.com/joshbode)

Other interesting stuff:

*   [gnab.github.com/editorjs](http://gnab.github.io/editorjs)
*   [judoole.github.com/GroovyBDD](http://judoole.github.io/GroovyBDD)
*   [bekkopen.github.com/infrastruktur-som-kode](http://bekkopen.github.io/infrastruktur-som-kode)
*   [ivarconr.github.com/Test-Driven-Web-Development/slides](http://ivarconr.github.io/Test-Driven-Web-Development/slides)
*   [havard.github.com/node.js-intro-norwegian](http://havard.github.io/node.js-intro-norwegian)
*   [mobmad.github.com/js-tdd-erfaringer](http://mobmad.github.io/js-tdd-erfaringer)
*   [roberto.github.com/ruby-sinform-2012](http://roberto.github.io/ruby-sinform-2012)
*   [http://asmeurer.github.io/python3-presentation/slides.html](http://asmeurer.github.io/python3-presentation/slides.html)
*   [Lecture notes using remark](http://keysan.me/ee361/)
*   [Time series analysis: theory and practice (a course using Remark)](http://jmlilly.net/course.html)
*   [sinedied.github.com/backslide](https://github.com/sinedied/backslide)

## Required Slide Elements
For each slide set, include:
- Title slide with course name, instructor name, and date
- Table of contents/agenda slide
- Clearly marked section dividers
- Properly formatted code examples (when needed)
- Summary/conclusion slide
- Reference slide (if applicable)

## Content Development Guidelines
- Transform bullet-point outlines into fully-developed slides
- Use the principle of "one main idea per slide"
- Create appropriate transitions between topics
- Add visual hierarchy through headers (h1, h2, h3)
- Include presenter notes where helpful
- Break complex concepts into sequential slides

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnkrateiaLucca/oreilly-reasoning-models](https://github.com/EnkrateiaLucca/oreilly-reasoning-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
