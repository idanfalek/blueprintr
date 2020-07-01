
<!-- README.md is generated from README.Rmd. Please edit that file -->

# blueprintr <img src="man/figures/logo.png" align="right" height="139" />

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![CRAN
status](https://www.r-pkg.org/badges/version/blueprintr)](https://CRAN.R-project.org/package=blueprintr)
[![R build
status](https://github.com/Global-TIES-for-Children/blueprintr/workflows/R-CMD-check/badge.svg)](https://github.com/Global-TIES-for-Children/blueprintr/actions)
<!-- badges: end -->

bluepintr is a plugin to [drake](https://github.com/ropensci/drake) that
adds automated steps for tabular dataset documentation and testing.
Designed for social science research projects, this package creates a
framework to build trust in your data and to prevent programming issues
from affecting your analysis results.

## Usage

Define blueprints of your data using `blueprint()`. Blueprints combine
drake target commands with some extra metadata about the output tabular
dataset, including the name and description of the data.

``` r
library(blueprintr)

blueprint1 <- blueprint(
  "blueprint1",
  description = "My first blueprint",
  command = {
    # Put all code related to building this dataset here
    mtcars
  }
)

blueprint1
#> <blueprint: 'blueprint1'>
#> 
#> Description: My first blueprint
#> Metadata location: '/Users/patrickanker/dev/blueprintr/blueprints/blueprint1.csv'
#> 
#> -- Command --
#> Drake command:
#> {
#>     mtcars
#> }
#> 
#> Raw command:
#> {
#>     mtcars
#> }
```

Refer to other datasets using `.TARGET()` to guarantee that parent
datasets are also tested and documented. Run checks on the dataset
entirely with the `checks` parameter and define variable tests in the
metadata files.

``` r
no_missing_cyl <- function(df) {
  all(!is.na(df$cyl))
}

blueprint2 <- blueprint(
  "blueprint2",
  description = "My second blueprint that depends on another",
  checks = check_list(
    no_missing_cyl()
  ),
  command =
    .TARGET("blueprint1") %>% 
      filter(cyl == 4)
)

blueprint2
#> <blueprint: 'blueprint2'>
#> 
#> Description: My second blueprint that depends on another
#> Metadata location: '/Users/patrickanker/dev/blueprintr/blueprints/blueprint2.csv'
#> 
#> -- Dataset content checks --
#> <check list>
#> no_missing_cyl()
#> 
#> -- Command --
#> Drake command:
#> blueprint1 %>% filter(cyl == 4)
#> 
#> Raw command:
#> .TARGET("blueprint1") %>% filter(cyl == 4)
```

Once all blueprints are defined, attach them to a plan so drake can run
the needed tasks.

``` r
library(magrittr)
library(drake)

plan_from_blueprint(blueprint1) %>% 
  attach_blueprint(blueprint2)
#> # A tibble: 10 x 2
#>    target            command                                                    
#>    <chr>             <expr>                                                     
#>  1 blueprint1_initi… {     mtcars }                                            …
#>  2 blueprint1_bluep… blueprint(name = "blueprint1", command = {     mtcars }, d…
#>  3 blueprint1_meta   create_metadata_file(blueprint1_initial, blueprint1_bluepr…
#>  4 blueprint1_checks eval_checks(all_variables_present(blueprint1_initial, blue…
#>  5 blueprint1        accept_content(blueprint1_checks, blueprint1_initial, blue…
#>  6 blueprint2_initi… blueprint1 %>% filter(cyl == 4)                           …
#>  7 blueprint2_bluep… blueprint(name = "blueprint2", command = .TARGET("blueprin…
#>  8 blueprint2_meta   create_metadata_file(blueprint2_initial, blueprint2_bluepr…
#>  9 blueprint2_checks eval_checks(all_variables_present(blueprint2_initial, blue…
#> 10 blueprint2        accept_content(blueprint2_checks, blueprint2_initial, blue…
```

## Installation

As `blueprintr` is not yet on CRAN, you must install the package from
this repository:

``` r
install.packages("remotes")
remotes::install_github("Global-TIES-for-Children/blueprintr")
```

## Contributing

Please note that the ‘blueprintr’ project is released with a
[Contributor Code of Conduct](.github/CODE_OF_CONDUCT.md). By
contributing to this project, you agree to abide by its terms.
