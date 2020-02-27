dplyr non-standard evaluation for programming
================
Julian Barg
27/02/2020

``` r
library(tidyverse)
```

I am using mtcars for this vignette.

``` r
glimpse(mtcars)
```

    ## Observations: 32
    ## Variables: 11
    ## $ mpg  <dbl> 21.0, 21.0, 22.8, 21.4, 18.7, 18.1, 14.3, 24.4, 22.8, 19.2, 17.8…
    ## $ cyl  <dbl> 6, 6, 4, 6, 8, 6, 8, 4, 4, 6, 6, 8, 8, 8, 8, 8, 8, 4, 4, 4, 4, 8…
    ## $ disp <dbl> 160.0, 160.0, 108.0, 258.0, 360.0, 225.0, 360.0, 146.7, 140.8, 1…
    ## $ hp   <dbl> 110, 110, 93, 110, 175, 105, 245, 62, 95, 123, 123, 180, 180, 18…
    ## $ drat <dbl> 3.90, 3.90, 3.85, 3.08, 3.15, 2.76, 3.21, 3.69, 3.92, 3.92, 3.92…
    ## $ wt   <dbl> 2.620, 2.875, 2.320, 3.215, 3.440, 3.460, 3.570, 3.190, 3.150, 3…
    ## $ qsec <dbl> 16.46, 17.02, 18.61, 19.44, 17.02, 20.22, 15.84, 20.00, 22.90, 1…
    ## $ vs   <dbl> 0, 0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    ## $ am   <dbl> 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0…
    ## $ gear <dbl> 4, 4, 4, 3, 3, 3, 3, 4, 4, 4, 4, 3, 3, 3, 3, 3, 3, 4, 4, 4, 3, 3…
    ## $ carb <dbl> 4, 4, 1, 1, 2, 1, 4, 2, 2, 4, 4, 3, 3, 3, 4, 4, 4, 1, 2, 1, 1, 2…

## dplyr non-standard evaluation for programming

I created this vignette after noticing a lack of high-quality guides on
how to use non-standard evaluation (NSE) for programming with dplyr. A
vignette by Hadley Wickham and his team exists
(<https://dplyr.tidyverse.org/articles/programming.html>), but
unfortunately it is unusable because it starts off by showing a lot of
way of *how not to use NSE for dplyr*. Every time I try to use the
vignette as a reference material, I end up using some code examples of
how not to do NSE for dplyr before finding the right code example.
Hence, this vignette will not include any wrong code examples. The
second purpose of this vignette is to be brief, in order for the
relevant information to be quick to find.

### Variable ingestion and usage with `enquo`

`enquo` provides a quosure that stores the input rather than evaluating
it. Execution is delayed until the final computation. `enquo` thus is
ideal for creating functions with dplyr.

#### bang-bang operator (`!!`)

Use the bang-bang operator for a literal evaluation of the quosure.

``` r
filter_by <- function(df, x, y) {
    x <- enquo(x)
    df %>%
        filter(!!x == 6)
}

filter_by(mtcars, cyl, 6)
```

    ##    mpg cyl  disp  hp drat    wt  qsec vs am gear carb
    ## 1 21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
    ## 2 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
    ## 3 21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
    ## 4 18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
    ## 5 19.2   6 167.6 123 3.92 3.440 18.30  1  0    4    4
    ## 6 17.8   6 167.6 123 3.92 3.440 18.90  1  0    4    4
    ## 7 19.7   6 145.0 175 3.62 2.770 15.50  0  1    5    6

#### Obtaining input as string with quo\_name

``` r
describe_variable <- function(df, variable, description) {
    variable <- enquo(variable)
    varname <- paste0(quo_name(variable), "_described")
    df %>%
        mutate(!! varname := paste(!! variable, description))
}

head(describe_variable(mtcars, cyl, "cylinders"))[-1:-5]
```

    ##      wt  qsec vs am gear carb cyl_described
    ## 1 2.620 16.46  0  1    4    4   6 cylinders
    ## 2 2.875 17.02  0  1    4    4   6 cylinders
    ## 3 2.320 18.61  1  1    4    1   4 cylinders
    ## 4 3.215 19.44  1  0    3    1   6 cylinders
    ## 5 3.440 17.02  0  0    3    2   8 cylinders
    ## 6 3.460 20.22  1  0    3    1   6 cylinders
