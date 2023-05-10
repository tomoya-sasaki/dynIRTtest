
<!-- README.md is generated from README.Rmd. Please edit that file -->

# (temporary name) dynIRTtest

<!-- badges: start -->
<!-- badges: end -->

## Misc

We will change the package name later. Please edit the `README.Rmd` file
and do not directly edit the `README.md` file.

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
Bayesian programming language Stan, as linked to R by the package
**CmdStanR**.

The basic workflow involves the following steps:

1.  Shape the data into the list format required by **CmdStanR**.
2.  Fit a dynamic factor model to the data.
3.  Extract posterior draws from the fitted object
4.  Identify and label parameter draws
5.  Visualize the results

### Step 0: load data

- The package contsins state-level societal outcomes from 2020 and 2021.

``` r
data("social_outcomes_2020_2021")
```

### Step 1: Reshape data

``` r
shaped_data <- shape_data(
    long_data = social_outcomes_2020_2021,
    unit_var = "st",
    time_var = "year",
    item_var = "outcome",
    value_var = "value",
    standardize = TRUE,
    periods_to_estimate = 2020:2021
)
```

### Step 2: Fit the model

- You can specify additional arguments for `cmdstanr::sample()`

``` r
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
    refresh = 10, # how often to update printed screen updates
    seed = 123,
    iter_warmup = 1000, # burn in
    iter_sampling = 1000 # samples for posterior inference
)
```

### Step 3: Extract Draws

``` r
# extract posterior draws
fitted_draws <- extract_draws(fitted)

# convergence diagnostics
checked <- check_convergence(fitted_draws)
```

### Step 4: Post analysis

``` r
# identify the sign and rotation of the parameter draws
# set rotate = FALSE to skip rotation
identified <- identify_draws(fitted_draws, rotate = TRUE)

# label the posterior draws
labeled <- label_draws(identified)
```

### Step 5: Visualization

- There are curretly four plotting functions.
- You can use `item_labels` argument in `plot_intercept` and
  `plot_loadings` to rename item labels to show in plots. Similarly,
  `unit_labels` argument is available for `plot_scores_ave` and
  `plot_scores_timetrend` to change unit labels.

``` r
# plot intercept
p <- plot_intercept(labeled)

# plot item loadings
p <- plot_loadings(labeled)

# plot average scores across different units
p <- plot_scores_ave(labeled)

# plot time series factor scores
p <- plot_scores_timetrend(labeled)

# each element is the new label
new_unit_names <- state.name
# names assigned to each element are the original labels
names(new_unit_names) <- state.abb

p <- plot_scores_ave(labeled, unit_labels = new_unit_names)
```
