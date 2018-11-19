# Source code of my [personal webpage](https://saik.at)

The website is built with [Hugo](https://gohugo.io/) — a static site generator made with Go.
It uses the [Cocoa](github.com/nishanths/cocoa-hugo-theme) theme,
with several modifications in CSS and Javascript.
While looking at websites built with the Cocoa theme, I came across the personal webpage of [Fernando Paolo](http://fspaolo.net/),
and I borrowed several design elements, especially the word  **Think**>.

The posts are written in Markdown.

The modifications in the Cocoa theme are as follows:
* Texts in `i18n/en.yaml`.
* Replaced `blog` with `think` in several files.
* Added `style="margin-top:2em;"` for the header of latest posts on the front page (`layouts/index.html`).
* Use separate heading and title for a page (`layouts/index.html`, added `{{ partial "page-heading.html" . }}` before the markdown rendering)
* Javascript in the footer `layouts/partials/footer_scripts.html`
  - open external links in new tab
  - email address created on page load.

The CSS overrides are as follows:
* fancy numbering for ordered list in the 'About Me' page.
* floating image (good for large screens) in 'About Me' page.

The primary font face is Open Sans and the monospace font face is Ubuntu Mono.
The social icons are from the Ionicons font set.
CSS classes for code syntax highlighting are inserted during compile-time by Hugo using Pygments.

