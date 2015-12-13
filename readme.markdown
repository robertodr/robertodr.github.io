This is the source for my [personal site](http://www.totaltrash.xyz).
I basically forked this [repository](https://github.com/blaenk/blaenk.github.io)
which contains the sources for [blaenk.denum](http://www.blaenkdenum.com/) website.

The Hakyll site's source (the Haskell code in **src/**) is
[BSD](https://tldrlegal.com/license/bsd-3-clause-license-(revised)) licensed as
stated in the cabal file. The contents of the website are licensed under the  [CC
BY-SA 4.0 license](http://creativecommons.org/licenses/by-sa/4.0/).

File structure and building
---------------------------

Hakyll does not enforce any particular directory structure or convention.
Given that I am basically copying [blaenkdenum](http://www.blaenkdenum.com/) website,
I have his [repository](https://github.com/blaenk/blaenk.github.io) structure.
It looks like this:

Entry           | Purpose
-------         | ----------
provider/       | compilable content
src/            | Hakyll, Pandoc customizations
Setup.hs        | build type
totaltrash.cabal    | dependency management
readme.markdown     | repository information

I use a [Cabal sandbox](http://coldwa.st/e/blog/2013-08-20-Cabal-sandbox.html) to build the website:
```
cabal sandbox init
cabal install -j cabal-install
cabal install -j --only-dep
```
Issuing `cabal build` will generate the site binary in a new top-level directory
directory `dist/`. Object files created by GHC are stored here. The
`site` binary, stored at the top level, is the actual binary which is used for
generating and manipulating the site. This binary has a variety of options, the
ones I commonly use are:

Option   |  Purpose
-------- |  ---------
build    |  Generate the entire site
watch    |  Generate changes on-the-fly and serve them on a preview server
deploy   |  Deploy the site using a custom deploy procedure

**Build** creates a top-level directory `generated/` with two
sub-directories: a directory `cache/` for cached content and a directory
`site/` where the compiled site is stored.

**Deploy** puts the compiled site into top-level directory `deploy/`
which is git-controlled and force pushes the content to the master branch,
effectively deploying (on GitHub).
