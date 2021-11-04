---
title: "{NHSRplotthedots}"
weight: 2
resources:
    - src: plot_01.png
      params:
          weight: -100
project_timeframe: Ongoing
---

A number of individuals from the NHS-R community ([Simon][swm], [Chris R][crs], [Chris M][cm], [Tom S][ts], [Zoe][zt],
and myself) collaborated to build an R package to implement the ["Making Data Count"][mdc] SPC rules and charts. The
goal was to replicate the functionality provided in their excel tool.

We have just reached version 0.1.0 and published the package to [CRAN][cran]. You can install it yourself by running

``` r
install.packages("NHSRplotthedots")
```

The documentation for the package is available [here][docs], and you can view the code on [GitHub][gh].

[swm]: https://twitter.com/SimonWMnhs
[crs]: https://twitter.com/CReadingSkilton
[cm]: https://twitter.com/chrismainey
[ts]: https://twitter.com/analyst42
[zt]: https://twitter.com/Letxuga007

[mdc]: https://www.england.nhs.uk/publication/making-data-count/
[cran]: https://cran.r-project.org/web/packages/NHSRplotthedots/index.html
[docs]: https://nhs-r-community.github.io/NHSRplotthedots/
[gh]: https://github.com/nhs-r-community/NHSRplotthedots
