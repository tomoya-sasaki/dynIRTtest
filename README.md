---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# (temporary name) dynIRTtest

<!-- badges: start -->
<!-- badges: end -->

We will change the package name later. 
Please edit the `README.Rmd` file and do not directly edit the `README.md` file.

## Installation

You can install the development version of **dynIRTtest** from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("tomoya-sasaki/dynIRTtest")
```

## TODOs

- [x] Add descriptions
- [ ] Figure specifications

## Explanation

The R package **dynIRTtest** fits dynamic multidimensional factor models with
binary, ordinal, and/or metric indicators. To do so, it uses the Bayesian
programming language [Stan](https://mc-stan.org), as linked to R by the package
[**CmdStanR**](https://mc-stan.org/cmdstanr/).

The basic workflow involves the following steps:
  1. Shape the data into the list format required by **CmdStanR**
     (`shape_data()`).
  2. Fit a dynamic factor model to the data.
  3. Extract parameter draws from the fitted model.
  4. If needed, identify the model by rotating (or sign-flipping) the draws.
  5. Convert the draws into a list of parameter-specific data frames with
     informative labels.
  6. Summarize and visualize the draws.

### Step 1: Shape the data


```r
library(dynIRTtest)
## Load data on societal attributes of U.S. states in 2020 and 2021
data("social_outcomes_2020_2021")
## Shape the data into list form
shaped_data <- shape_data(
    long_data = social_outcomes_2020_2021,
    unit_var = "st",
    time_var = "year",
    item_var = "outcome",
    value_var = "value"
    periods_to_estimate = 2020:2021,
    ordinal_items = NA,
    binary_items = NA,
    max_cats = 10,
    standardize = TRUE,
    make_indicator_for_zeros = TRUE
)
```

### Step 3: Fit the model


```r
fitted <- fit(
    data = shaped_data,
    lambda_zeros = data.frame(
        item = c("x_cps_spm_poverty"),
        dim = c(2)
    ),
    n_dim = 2,
    chains = 2,
    parallelize_within_chains = TRUE,
    threads_per_chain = 2,
    constant_alpha = FALSE,
    separate_eta = TRUE,
    init_kappa = FALSE,
    force_recompile = FALSE,
    iter_warmup = 50,
    iter_sampling = 50,
    adapt_delta = .9,
    refresh = 10,
    seed = 123
)
```

### Step 4: Post analysis



