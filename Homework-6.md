p8105_hw6_sd3730
================
Stacey Dai
2024-12-02

# Problem 1

Load necessary packages for problem 1.

``` r
library(tidyverse)
library(rnoaa)
library(broom)
```

Clean and prepare the noaa dataset for analysis.

``` r
weather_df = 
  rnoaa::meteo_pull_monitors(
    c("USW00094728"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2017-01-01",
    date_max = "2017-12-31"
  ) |>
  mutate(
    name = recode(id, USW00094728 = "CentralPark_NY"),
    tmin = tmin / 10,
    tmax = tmax / 10
  ) |>
  select(name, id, everything())
```

Generate 5000 bootstrap samples and fit a simple linear regression for
each sample.

``` r
set.seed(123)

bootstrap_results = 
  replicate(
    5000, 
    {
      sample_data = weather_df |> slice_sample(prop = 1, replace = TRUE)
      model = lm(tmax ~ tmin, data = sample_data)
      
      r2 = glance(model)$r.squared
      coefs = tidy(model)
      log_product = log(coefs$estimate[1] * coefs$estimate[2])
      
      tibble(r2 = r2, log_product = log_product)
    },
    simplify = FALSE
  ) |>
  bind_rows()
```

Identify the 2.5% and 97.5% quantiles to provide a 95% confidence
interval.

``` r
ci_r2 = quantile(bootstrap_results$r2, probs = c(0.025, 0.975))
ci_log_product = quantile(bootstrap_results$log_product, probs = c(0.025, 0.975))
```

Plot histograms for both quantiles.

``` r
bootstrap_results |>
  pivot_longer(cols = everything(), names_to = "metric", values_to = "value") |>
  ggplot(aes(x = value)) +
  geom_histogram(bins = 30, fill = "skyblue", color = "black", alpha = 0.7) +
  facet_wrap(~metric, scales = "free", ncol = 1) +
  theme_minimal() +
  labs(
    title = "Bootstrap Distributions",
    x = "Estimated Value",
    y = "Frequency"
  )
```

![](Homework-6_files/figure-gfm/p1plot-1.png)<!-- -->

The distribution for the log_product graph appears to be symmetric andd
approximately normal. The distribution for the r2 graphh is symmetrical
and unimodal.

# Problem 2

``` r
library(purrr)
```