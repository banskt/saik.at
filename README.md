# Source code for personal website

## [saik.at](https://saik.at)

The website is built with [Hugo](https://gohugo.io/).
It uses the [Cocoa-Refresh](https://github.com/banskt/cocoa-refresh-hugo-theme) theme,
which is a fork from the [Cocoa](https://github.com/nishanths/cocoa-hugo-theme/) theme with some changes.
The theme is included in the repository as a submodule.

[Working with submodules](http://dan.mccloy.info/2015/06/11/Git-submodules/).

### CSS overrides
* fancy numbering for ordered list in the 'About Me' page.
* floating image (good for large screens) in 'About Me' page.

### How to deploy
Changes are deployed using Github Actions.
Pushing local changes to the server is as simple as:
```
git push origin master
```
For details, [see here](https://saik.at/think/2020-06-17-deploy-static-sites-with-github-actions/).

### Colophon
The posts are written in Markdown.
The primary font face is Open Sans and the monospace font face is Ubuntu Mono.
The social icons are from the Ionicons font set.
CSS classes for code syntax highlighting are inserted during compile-time by Hugo using Pygments.

### To Do
* Include Latex Support
* Images for Research Projects
