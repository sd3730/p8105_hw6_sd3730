p8105_hw6_sd3730
================
Stacey Dai
2024-12-02

``` r
library(tidyverse)
library(rnoaa)
library(broom)
```

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

``` r
set.seed(123) # For reproducibility

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
