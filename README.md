
<!-- README.md is generated from README.Rmd. Please edit that file -->

# (temporary name) dynIRTtest

<!-- badges: start -->
<!-- badges: end -->

## TODOs

- [x] Add descriptions
- [ ] Figure specifications

## Misc

- We will change the package name later.
- Please edit the `README.Rmd` file and do not directly edit the
  `README.md` file.

## Installation

You can install the development version of **dynIRTtest** from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("tomoya-sasaki/dynIRTtest")
```

## Explanation

The R package **dynIRTtest** fits dynamic multidimensional factor models
with binary, ordinal, and/or metric indicators. To do so, it uses the
Bayesian programming language [Stan](https://mc-stan.org), as linked to
R by the package [**CmdStanR**](https://mc-stan.org/cmdstanr/).

The basic workflow involves the following steps:

1.  Shape the data into the list format required by **CmdStanR**
    (`shape_data()`).
2.  Fit a dynamic factor model to the data.
3.  Extract parameter draws from the fitted model.
4.  If needed, identify the model by rotating (or sign-flipping) the
    draws.
5.  Convert the draws into a list of parameter-specific data frames with
    informative labels.
6.  Summarize and visualize the draws.

``` r
library(dynIRTtest)
```

### Step 1: Shape data

``` r
## Load data on societal attributes of U.S. states in 2020 and 2021
data("social_outcomes_2020_2021")

## Remove any rows that contain NA's in `unit_var`, `time_var`, `item_var`, or `value_var`
social_outcomes_2020_2021 |>
    drop_na(st, year, outcome, value) -> social_outcomes_2020_2021

## Shape the data into list form
shaped_data <- shape_data(
    long_data = social_outcomes_2020_2021,
    unit_var = "st",
    time_var = "year",
    item_var = "outcome",
    value_var = "value",
    periods_to_estimate = 2020:2021,
    ordinal_items = NA,
    binary_items = NA,
    max_cats = 10,
    standardize = TRUE,
    make_indicator_for_zeros = TRUE
)
```

### Step 2: Fit the model

- You can specify additional arguments for `cmdstanr::sample()`. Check
  [here](https://mc-stan.org/cmdstanr/reference/model-method-sample.html).

``` r
fitted <- fit(
    data = shaped_data,
    lambda_zeros = data.frame(
        item = c("x_cps_spm_poverty"),
        dim = c(2)
    ), # fix `x_cps_spm_poverty` to zero
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
    refresh = 10, # how often to update printed screen updates
    seed = 123,
    iter_warmup = 1000, # burn in
    iter_sampling = 1000 # samples for posterior inference
)
```

### Step 3: Extract Draws

``` r
## Extract posterior draws
fitted_draws <- extract_draws(fitted)

## Convergence diagnostics
checked <- check_convergence(fitted_draws)
```

### Step 4: Post analysis

``` r
## Identify the sign and rotation of the parameter draws
## Set rotate = FALSE to skip rotation
identified <- identify_draws(fitted_draws, rotate = TRUE)

## Label the posterior draws
labeled <- label_draws(identified)
```

### Step 5: Visualization

- There are currently four plotting functions.
- You can use `item_labels` argument in `plot_intercept` and
  `plot_loadings` to rename item labels to show in plots. Similarly,
  `unit_labels` argument is available for `plot_scores_ave` and
  `plot_scores_timetrend` to change unit labels.

``` r
## Plot intercept
p <- plot_intercept(labeled)

## Plot item loadings
p <- plot_loadings(labeled)

## Plot average scores across different units
p <- plot_scores_ave(labeled)

## Plot time series factor scores
p <- plot_scores_timetrend(labeled)

## Change unit labels to show in plot
## Each element is the new label
new_unit_names <- state.name
## Names assigned to each element are the original labels
names(new_unit_names) <- state.abb

p <- plot_scores_ave(labeled, unit_labels = new_unit_names)
```
