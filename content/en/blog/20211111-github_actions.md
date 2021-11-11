---
title: "GitHub Actions for R"
date: 2021-11-11T15:00:00-00:00
tags: ["R", "GitHub", "GitHub Actions"]
---

[GitHub Actions](https://github.com/features/actions) is a super useful way to automate some of the boring tasks that
we have to do day in, day out. And the wonderful [r-lib](https://github.com/r-lib) have created a bunch of actions for
R that will make your life super easy. Along with the `{usethis}` package, and the function
`usethis::use_github_action()`, you can easily set up a bunch of
[predefined actions](https://github.com/r-lib/actions/tree/master/examples).

Below is an example of an action that I created for my [talks](https://github.com/tomjemmett/talks) repository recently.

Hopefully the explanation can be of use to you in creating your own actions! One thing that isn't mentioned below is
this was a bit of a painful process to create, with many, many failures along the way. Creating GitHub actions can be a
painful process if you do encounter build failures, but persistence does pay off!

### full yaml file

``` yaml
on:
  push:
    branches: [main, master]
    tags: ['*']

name: "Build Slides"

jobs:
  build_slides:
    runs-on: ubuntu-latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v2

      - name: install libcurl
        run: |
          sudo apt install -y libcurl4-openssl-dev
          
      - uses: r-lib/actions/setup-pandoc@v1
      - uses: r-lib/actions/setup-r@v1
        with:
          use-public-rspm: true
      - uses: r-lib/actions/setup-renv@v1

      - name: Cache icons
        uses: actions/cache@v2
        with:
          path: icons
          key: ${{ runner.os }}-${{ hashFiles('get_icons.R') }}

      - name: Get icons
        run: Rscript get_icons.R

      - name: Build slides
        run: Rscript .github/workflows/build_slides.R

      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@4.1.5
        with:
          branch: gh-pages
          folder: public
```

## how it works

This can be broken down into a few steps:

### checkout the branch

Pretty much all actions will start out with

``` yaml
- uses: actions/checkout@v2
```

This will simply checkout the branch that triggered this action into the current folder.

### install system dependencies

I got caught out at a later step because `libcurl` wasn't installed, so I simply install this in Ubuntu with

``` yaml
- name: install libcurl
  run: |
    sudo apt install -y libcurl4-openssl-dev
```

This step may not be necessary in all of your actions, but if you get issues when installing R packages you may need
to run a step like this. Some of the r-lib actions will sort out system dependencies for you, I didn't take this
approach though as I was using `{renv}` and I'm not too sure yet how to get the system dependencies using this.

### setting up R

The next chunk does 3 things:

1) it installs pandoc (used by `{rmarkdown}` for converting from markdown to html and other formats)
2) it installs R, and set's it to use the public [RStudio Package Manager](https://packagemanager.rstudio.com/client/)
(this will make installing packages on Linux much, much quicker)
3) it installs the `{renv}` package, then uses that to install all of the packages you need for your project.
Additionally it caches the package library. Next time the action runs, if the `renv.lock` file hasn't changed then it
will just reload the installed packages from the cache.

``` yaml
  - uses: r-lib/actions/setup-pandoc@v1
  - uses: r-lib/actions/setup-r@v1
    with:
      use-public-rspm: true
  - uses: r-lib/actions/setup-renv@v1
```

If you haven't used [`{renv}`](https://github.com/rstudio/renv) before it's well worth checking out, it makes life
significantly easier for GitHub actions as it will keep a track of all the packages that you need to run, and then
adding in the one step automatically configures the remote environment like your own.

### my stuff

The next chunk is some stuff I created. Specifically, I have two scripts that I want to run:

* `get_icons.R`
* `.github/workflows/build_slides.R`

The [`{icons}`](https://github.com/mitchelloharawild/icons) package is a great way to include svg icons in your
Rmarkdown reports. I use it to insert nice twitter/github etc. logo's into my about me slides. However, you need to
run the `download_*()` functions in order to get the icons to work. The `get_icons.R` script does this for me, but I
don't want to always re-run the entire download, so I cache the folder that I download the icons to.

The `build_slides.R` file simply looks for any `.Rmd` files in the folder, renders them, then keeps just the files that
are needed for the deployment (e.g. the `.html` files and the `css`/`img`/`libs` folders).

``` yaml
- name: Cache icons
  uses: actions/cache@v2
  with:
    path: icons
    key: ${{ runner.os }}-${{ hashFiles('get_icons.R') }}

- name: Get icons
  run: Rscript get_icons.R

- name: Build slides
    run: Rscript .github/workflows/build_slides.R
```

### deploy to gh-pages

The final chunk handles deploying the pages to GitHub pages. You simply say which branch you want to deploy to, and the
folder that it should deploy. I render my files to the folder `"public"`, so that is what I deploy.

``` yaml
- name: Deploy 🚀
  uses: JamesIves/github-pages-deploy-action@4.1.5
  with:
    branch: gh-pages
    folder: public
```
