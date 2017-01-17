pandoc-slideshow
================

[![Build Status](https://travis-ci.org/ivan-m/pandoc-slideshow.svg)](https://travis-ci.org/ivan-m/pandoc-slideshow)

This is a base repository for how I make presentations using [Pandoc]
and reveal.js hosted on [GitHub Pages].  For example, the sample
slides found in [`Slides.md`] can be found [here].

This assumes that you use pandoc-mode for Emacs to help you write your
slides.  If you don't, you will need to edit [`runPandoc.hs`]
appropriately.

Note: whilst I'm currently using `reveal.js` for this, there's no
reason you can't use this for any other HTML-based slideshow format
that can be generated by Pandoc; you'll just need to change where the
symbolic link `.Slides.md.default.pandoc` points to.

Directions for use
------------------

1. Clone/copy this repository.

2. Edit the `.travis.yml` file:

    a) Specify the `GH_REPO` to point to your actual repository.

    b) Add a `GH_TOKEN` by [generating] one and then encrypting it
        (because unless you're me, mine won't work for you).

    c) Change the version of GHC and cabal-install if necessary.

    d) Any other changes you want to make.

3. Any images, CSS changes or updates to the reveal.js subtree need to
   be made in the `shared-data` branch, with master then rebased upon
   that and force-pushed.

4. Make sure you have enabled your branch on your [Travis-CI profile].

[Pandoc]: http://pandoc.org/
[reveal.js]: http://lab.hakim.se/reveal-js/
[GitHub Pages]: https://pages.github.com/
[`Slides.md`]: Slides.md
[here]: http://ivan-m.github.io/pandoc-slideshow
[`runPandoc.hs`]: runPandoc.hs
[generating]: https://github.com/settings/tokens
[encrypting]: https://docs.travis-ci.com/user/encryption-keys/
[Travis-CI profile]: https://travis-ci.org/profile
