ISLR-Chapter-3-Lab
================
Samuel Hansen
2017-06-29

-   [3.6 Lab: Linear Regression](#lab-linear-regression)
    -   [3.6.1 Libraries](#libraries)
    -   [3.6.2 Simple Linear Regression](#simple-linear-regression)
    -   [3.6.3 Multiple Linear Regression](#multiple-linear-regression)

``` r
#Libraries to be used.
library(tidyverse)
```

    ## Loading tidyverse: ggplot2
    ## Loading tidyverse: tibble
    ## Loading tidyverse: tidyr
    ## Loading tidyverse: readr
    ## Loading tidyverse: purrr
    ## Loading tidyverse: dplyr

    ## Conflicts with tidy packages ----------------------------------------------

    ## filter(): dplyr, stats
    ## lag():    dplyr, stats

3.6 Lab: Linear Regression
==========================

3.6.1 Libraries
---------------

The `library()` function is used to load libraries, or groups of functions and `library()` data sets that are not included in the base R distribution. Basic functions that perform least squares linear regression and other simple analyses come standard with the base distribution, but more exotic functions require additional libraries. Here we load the `MASS` package, which is a very large collection of data sets and functions. We also load the `ISLR` package, which includes the data sets associated with this book.

``` r
library(MASS)
```

    ## 
    ## Attaching package: 'MASS'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     select

``` r
library(ISLR)
```

If you receive an error message when loading any of these libraries, it likely indicates that the corresponding library has not yet been installed on your system. Some libraries, such as `MASS`, come with R and do not need to be separately installed on your computer. However, other packages, such as `ISLR`, must be downloaded the first time they are used. This can be done directly from within R. For example, on a Windows system, select the `Install package` option under the `Packages` tab. After you select any mirror site, a list of available packages will appear. Simply select the package you wish to install and R will automatically download the package. Alternatively, this can be done at the R command line via `install.packages("ISLR")`. This installation only needs to be done the first time you use a package. However, the `library()` function must be called each time you wish to use a given package.

3.6.2 Simple Linear Regression
------------------------------

The `MASS` library contains the `Boston` data set, which records `medv` (median house value) for 506 neighborhoods around Boston. We will seek to predict medv using 13 predictors such as `rm` (average number of rooms per house), `age` (average age of houses), and `lstat` (percent of households with low socioeconomic status).

``` r
names(Boston)
```

    ##  [1] "crim"    "zn"      "indus"   "chas"    "nox"     "rm"      "age"    
    ##  [8] "dis"     "rad"     "tax"     "ptratio" "black"   "lstat"   "medv"

To find out more about the data set, we can type `?Boston`. We will start by using the `lm()` function to fit a simple linear regression model, with `medv` as the response and `lstat` as the predictor. The basic syntax is `lm(y ∼ x, data)`, where `y` is the response, `x` is the predictor, and `data` is the data set in which these two variables are kept.

``` r
 lm.fit = lm(medv ~ lstat)
```

The command causes an error because R does not know where to find the variables `medv` and `lstat`. The next line tells R that the variables are in `Boston`. If we attach `Boston`, the first line works fine because R now recognizes the variables.

``` r
lm.fit = lm(medv ~ lstat, data = Boston)
```

If we type `lm.fit`, some basic information about the model is output. For more detailed information, we use `summary(lm.fit)`. This gives us p-values and standard errors for the coefficients, as well as the *R*<sup>2</sup> statistic and F-statistic for the model.

``` r
lm.fit
```

    ## 
    ## Call:
    ## lm(formula = medv ~ lstat, data = Boston)
    ## 
    ## Coefficients:
    ## (Intercept)        lstat  
    ##       34.55        -0.95

``` r
lm.fit %>% 
  summary()
```

    ## 
    ## Call:
    ## lm(formula = medv ~ lstat, data = Boston)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -15.168  -3.990  -1.318   2.034  24.500 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 34.55384    0.56263   61.41   <2e-16 ***
    ## lstat       -0.95005    0.03873  -24.53   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 6.216 on 504 degrees of freedom
    ## Multiple R-squared:  0.5441, Adjusted R-squared:  0.5432 
    ## F-statistic: 601.6 on 1 and 504 DF,  p-value: < 2.2e-16

We can use the `names()` function in order to find out what other pieces of information are stored in `lm.fit`. Although we can extract these quantities by name — e.g. `lm.fit$coefficients` — it is safer to use the extractor functions like `coef()` to access them.

``` r
names(lm.fit)
```

    ##  [1] "coefficients"  "residuals"     "effects"       "rank"         
    ##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
    ##  [9] "xlevels"       "call"          "terms"         "model"

``` r
coef(lm.fit)
```

    ## (Intercept)       lstat 
    ##  34.5538409  -0.9500494

In order to obtain a confidence interval for the coefficient estimates, we can use the `confint()` command.

``` r
confint(lm.fit)
```

    ##                 2.5 %     97.5 %
    ## (Intercept) 33.448457 35.6592247
    ## lstat       -1.026148 -0.8739505

The `predict()` function can be used to produce confidence intervals and `predict()` prediction intervals for the prediction of `medv` for a given value of `lstat`.

``` r
new_data <- tibble(lstat = c(5, 10, 15))

predict(
  object = lm.fit, 
  newdata = new_data, 
  interval = "confidence"
)
```

    ##        fit      lwr      upr
    ## 1 29.80359 29.00741 30.59978
    ## 2 25.05335 24.47413 25.63256
    ## 3 20.30310 19.73159 20.87461

``` r
predict(
  object = lm.fit, 
  newdata = new_data, 
  interval = "prediction"
)
```

    ##        fit       lwr      upr
    ## 1 29.80359 17.565675 42.04151
    ## 2 25.05335 12.827626 37.27907
    ## 3 20.30310  8.077742 32.52846

For instance, the 95% confidence interval associated with a lstat value of 10 is (24.47, 25.63), and the 95% prediction interval is (12.828, 37.28). As expected, the confidence and prediction intervals are centered around the same point (a predicted value of 25.05 for `medv` when `lstat` equals 10), but the latter are substantially wider.

We will now plot `medv` and lstat along with the least squares regression line using the `ggplot()` and `geom_smooth()` functions.

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point() +
  geom_smooth(method = "lm")
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-13-1.png" width="100%" />

**NOTE: THIS TEXT NEEDS UPDATING**

*There is some evidence for non-linearity in the relationship between `lstat` and `medv`. We will explore this issue later in this lab. The `abline()` function can be used to draw any line, not just the least squares regression line. To draw a line with intercept `a` and slope `b`, we type `abline(a, b)`. Below we experiment with some additional settings for plotting lines and points. The `lwd = 3` command causes the width of the regression line to be increased by a factor of 3; this works for the `plot()` and `lines()` functions also. We can also use the pch option to create different plotting symbols.*

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point() +
  geom_smooth(
    method = "lm", 
    size = 3
  )
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-1.png" width="100%" />

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point() +
  geom_smooth(
    method = "lm", 
    size = 3,
    color = "red"
  )
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-2.png" width="100%" />

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point(color = "red")
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-3.png" width="100%" />

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point(
    color = "red",
    shape = 20
  )
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-4.png" width="100%" />

``` r
Boston %>% 
  ggplot(mapping = aes(x = lstat, y = medv)) +
  geom_point(
    color = "red",
    shape = "+"
  )
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-5.png" width="100%" />

``` r
tibble(
  x = 1:20,
  y = 1:20,
  shape = 1:20
) %>% 
  ggplot(mapping = aes(
    x = x, 
    y = y, 
    shape = shape)
  ) +
  geom_point() +
  scale_shape_identity()
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-14-6.png" width="100%" />

**NOTE: THIS TEXT NEEDS UPDATING**

*Next we examine some diagnostic plots, several of which were discussed in Section 3.3.3. Four diagnostic plots are automatically produced by applying the `plot` function directly to the output from `lm()`. In general, this command will produce one plot at a time, and hitting `Enter` will generate the next plot. However, it is often convenient to view all four plots together. We can achieve this by using the `par()` function, which tells R to split the display screen into separate panels so that multiple plots can be viewed simultaneously. For example, `par(mfrow = c(2, 2))` divides the plotting region into a 2 × 2 grid of panels.*

**NOTE: THIS CODE NEEDS UPDATING (perhaps delete section)**

``` r
par(mfrow = c(2, 2))
plot(lm.fit)
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-15-1.png" width="100%" />

``` r
plot(predict (lm.fit), residuals (lm.fit))
plot(predict (lm.fit), rstudent (lm.fit))
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-15-2.png" width="100%" />

On the basis of the residual plots, there is some evidence of non-linearity. Leverage statistics can be computed for any number of predictors using the `hatvalues()` function.

``` r
tibble(
  hat_value = lm.fit %>% hatvalues(),
  index = seq_along(hat_value)
) %>% 
  ggplot(mapping = aes(x = index, y = hat_value)) +
  geom_point()
```

<img src="lab3_files/figure-markdown_github/unnamed-chunk-16-1.png" width="100%" />

``` r
lm.fit %>% 
  hatvalues() %>% 
  which.max()
```

    ## 375 
    ## 375

The `which.max()` function identifies the index of the largest element of a vector. In this case, it tells us which observation has the largest leverage statistic.

3.6.3 Multiple Linear Regression
--------------------------------

In order to fit a multiple linear regression model using least squares, we again use the `lm()` function. The syntax `lm(y ∼ x1 + x2 + x3)` is used to fit a model with three predictors, `x1`, `x2`, and `x3`. The `summary()` function now outputs the regression coefficients for all the predictors.

``` r
lm.fit = lm(medv ~ lstat + age, data = Boston)
summary(lm.fit)
```

    ## 
    ## Call:
    ## lm(formula = medv ~ lstat + age, data = Boston)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -15.981  -3.978  -1.283   1.968  23.158 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 33.22276    0.73085  45.458  < 2e-16 ***
    ## lstat       -1.03207    0.04819 -21.416  < 2e-16 ***
    ## age          0.03454    0.01223   2.826  0.00491 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 6.173 on 503 degrees of freedom
    ## Multiple R-squared:  0.5513, Adjusted R-squared:  0.5495 
    ## F-statistic:   309 on 2 and 503 DF,  p-value: < 2.2e-16

The `Boston` data set contains 13 variables, and so it would be cumbersome to have to type all of these in order to perform a regression using all of the predictors. Instead, we can use the following short-hand:

``` r
lm.fit = lm(medv ~., data = Boston)
summary(lm.fit)
```

    ## 
    ## Call:
    ## lm(formula = medv ~ ., data = Boston)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -15.595  -2.730  -0.518   1.777  26.199 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  3.646e+01  5.103e+00   7.144 3.28e-12 ***
    ## crim        -1.080e-01  3.286e-02  -3.287 0.001087 ** 
    ## zn           4.642e-02  1.373e-02   3.382 0.000778 ***
    ## indus        2.056e-02  6.150e-02   0.334 0.738288    
    ## chas         2.687e+00  8.616e-01   3.118 0.001925 ** 
    ## nox         -1.777e+01  3.820e+00  -4.651 4.25e-06 ***
    ## rm           3.810e+00  4.179e-01   9.116  < 2e-16 ***
    ## age          6.922e-04  1.321e-02   0.052 0.958229    
    ## dis         -1.476e+00  1.995e-01  -7.398 6.01e-13 ***
    ## rad          3.060e-01  6.635e-02   4.613 5.07e-06 ***
    ## tax         -1.233e-02  3.760e-03  -3.280 0.001112 ** 
    ## ptratio     -9.527e-01  1.308e-01  -7.283 1.31e-12 ***
    ## black        9.312e-03  2.686e-03   3.467 0.000573 ***
    ## lstat       -5.248e-01  5.072e-02 -10.347  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 4.745 on 492 degrees of freedom
    ## Multiple R-squared:  0.7406, Adjusted R-squared:  0.7338 
    ## F-statistic: 108.1 on 13 and 492 DF,  p-value: < 2.2e-16

We can access the individual components of a summary object by name (type `?summary.lm` to see what is available). Hence `summary(lm.fit)$r.sq` gives us the *R*<sup>2</sup>, and `summary(lm.fit)$sigma` gives us the `RSE`. The `vif()` function, part of the `car` package, can be used to compute variance inflation factors. Most VIF’s are low to moderate for this data. The `car` package is not part of the base R installation so it must be downloaded the first time you use it via the `install.packages` option in R.

``` r
library(car)
```

    ## 
    ## Attaching package: 'car'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     recode

    ## The following object is masked from 'package:purrr':
    ## 
    ##     some

``` r
vif(lm.fit)
```

    ##     crim       zn    indus     chas      nox       rm      age      dis 
    ## 1.792192 2.298758 3.991596 1.073995 4.393720 1.933744 3.100826 3.955945 
    ##      rad      tax  ptratio    black    lstat 
    ## 7.484496 9.008554 1.799084 1.348521 2.941491

What if we would like to perform a regression using all of the variables but one? For example, in the above regression output, `age` has a high p-value. So we may wish to run a regression excluding this predictor. The following syntax results in a regression using all predictors except `age`.

``` r
lm.fit1 = lm(medv ~. -age, data = Boston)
summary(lm.fit1)
```

    ## 
    ## Call:
    ## lm(formula = medv ~ . - age, data = Boston)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -15.6054  -2.7313  -0.5188   1.7601  26.2243 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  36.436927   5.080119   7.172 2.72e-12 ***
    ## crim         -0.108006   0.032832  -3.290 0.001075 ** 
    ## zn            0.046334   0.013613   3.404 0.000719 ***
    ## indus         0.020562   0.061433   0.335 0.737989    
    ## chas          2.689026   0.859598   3.128 0.001863 ** 
    ## nox         -17.713540   3.679308  -4.814 1.97e-06 ***
    ## rm            3.814394   0.408480   9.338  < 2e-16 ***
    ## dis          -1.478612   0.190611  -7.757 5.03e-14 ***
    ## rad           0.305786   0.066089   4.627 4.75e-06 ***
    ## tax          -0.012329   0.003755  -3.283 0.001099 ** 
    ## ptratio      -0.952211   0.130294  -7.308 1.10e-12 ***
    ## black         0.009321   0.002678   3.481 0.000544 ***
    ## lstat        -0.523852   0.047625 -10.999  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 4.74 on 493 degrees of freedom
    ## Multiple R-squared:  0.7406, Adjusted R-squared:  0.7343 
    ## F-statistic: 117.3 on 12 and 493 DF,  p-value: < 2.2e-16
