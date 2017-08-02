Examples from lab 2 - 6
================
Hershel Mehta and Chirstopher Oh
2017-07-29

-   [ggplot example](#ggplot-example)
    -   [Original](#original)
    -   [Revised](#revised)
-   [Logistic regression example](#logistic-regression-example)
    -   [Original](#original-1)
    -   [Revised](#revised-1)
-   [Bootstrap](#bootstrap)
    -   [Original](#original-2)
    -   [Revised](#revised-2)
-   [Model Selection using Cross-Validation](#model-selection-using-cross-validation)
    -   [Original](#original-3)
    -   [Revised](#revised-3)

ggplot example
--------------

Read in the file

``` r
auto <-
  ISLR::Auto %>% 
  filter_all(all_vars(!is.na(.))) %>% 
  mutate(cylinders = as.factor(cylinders))
```

### Original

``` r
plot(auto$cylinders, auto$mpg, col="red", varwidth=T, xlab="cylinders", ylab="MPG")
```

<img src="examples_lab2-4_files/figure-markdown_github/unnamed-chunk-2-1.png" width="100%" />

### Revised

``` r
auto %>%
  ggplot(aes(cylinders, mpg)) +
  geom_boxplot(varwidth = TRUE) +
  labs(
    x = "Cylinders",
    y = "MPG"
  )
```

<img src="examples_lab2-4_files/figure-markdown_github/unnamed-chunk-3-1.png" width="100%" />

Logistic regression example
---------------------------

### Original

Separate the training / test set

``` r
attach(Smarket)
train=(Year <2005)
Smarket.2005=Smarket[!train ,]
dim(Smarket.2005)
```

    ## [1] 252   9

``` r
Direction.2005= Direction [!train]
```

Fit the logistic regression model using the training set

``` r
glm.fits=glm(Direction~Lag1+Lag2+Lag3+Lag4+Lag5+Volume ,
             data=Smarket ,family=binomial ,subset=train)
```

Add predictions to the test set

``` r
glm.probs=predict (glm.fits,Smarket.2005, type="response")
glm.pred=rep("Down",252)
glm.pred[glm.probs >.5]="Up"
```

Compute the prediction error

``` r
table(glm.pred ,Direction.2005)
```

    ##         Direction.2005
    ## glm.pred Down Up
    ##     Down   77 97
    ##     Up     34 44

``` r
mean(glm.pred==Direction.2005)
```

    ## [1] 0.4801587

``` r
mean(glm.pred!=Direction.2005)
```

    ## [1] 0.5198413

### Revised

Get the data into a tibble format

``` r
smarket <- as.tibble(Smarket)
```

Separate the training / test set

``` r
train <-
  smarket %>%
  filter(Year < 2005)
test <-
  smarket %>% 
  filter(Year >= 2005)
```

Logistic regression model on the training set

``` r
glm_fit <- 
  glm(
    Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 + Volume,
    data = train,
    family = binomial
  )
```

Add prediction to the test set

``` r
test <-
  test %>%
  mutate(
    pred = predict(glm_fit, newdata = test, type = "response"),
    pred_dir = ifelse(pred > .5, "Up", "Down")
  )
```

Calculate the error on the test set

``` r
test %>%
  select(Direction, pred_dir) %>% 
  mutate(err = ifelse(Direction != pred_dir, 1, 0)) %>% 
  summarize(mean(err))
```

    ## # A tibble: 1 x 1
    ##   `mean(err)`
    ##         <dbl>
    ## 1   0.5198413

Bootstrap
---------

### Original

``` r
alpha.fn=function(data,index){
 X=data$X[index]
 Y=data$Y[index]
 return((var(Y)-cov(X,Y))/(var(X)+var(Y)-2*cov(X,Y)))
}
```

``` r
set.seed(1)
boot(Portfolio,alpha.fn,R=1000)
```

    ## 
    ## ORDINARY NONPARAMETRIC BOOTSTRAP
    ## 
    ## 
    ## Call:
    ## boot(data = Portfolio, statistic = alpha.fn, R = 1000)
    ## 
    ## 
    ## Bootstrap Statistics :
    ##      original       bias    std. error
    ## t1* 0.5758321 6.936399e-05  0.08868935

### Revised

First, we write a function that takes in a dataframe, and outputs the alpha statistic of variables `X` and `Y`

``` r
sample_alpha <- function(x) {
  data <- as_tibble(x)
  
  X <- data$X
  Y <- data$Y
  
  (var(Y) - cov(X,Y)) / (var(X) + var(Y) - 2 * cov(X,Y))
}
```

We create 1000 bootsrapped samples, calculate the alpha statistic for each sample, and then calculate the mean and standard deviation of the alpha statistic

``` r
Portfolio %>% 
  resamplr::bootstrap(R = 1000) %>% 
  mutate(
    alpha = map_dbl(sample, sample_alpha)
  ) %>% 
  summarise(
    alpha_mean = mean(alpha),
    alpha_stddev = sd(alpha)
  )
```

    ## # A tibble: 1 x 2
    ##   alpha_mean alpha_stddev
    ##        <dbl>        <dbl>
    ## 1  0.5790302   0.08691761

Model Selection using Cross-Validation
--------------------------------------

### Original

NOTE: Parts of the original appear to be deprecated (since `leaps::regsubsets()` appears to randomly add suffixes to many of the variables). I've commented those lines out, so we can still knit the file

``` r
set.seed(1)

train=sample(c(TRUE,FALSE), nrow(Hitters),rep=TRUE)
test=(!train)
```

``` r
predict.regsubsets=function(object,newdata,id,...){
  form=as.formula(object$call[[2]])
  mat=model.matrix(form,newdata)
  coefi=coef(object,id=id)
  xvars=names(coefi)
  mat[,xvars]%*%coefi
}
```

``` r
regfit.best=regsubsets(Salary~.,data=Hitters,nvmax=19)
coef(regfit.best,10)
```

    ##  (Intercept)        AtBat         Hits        Walks       CAtBat 
    ##  162.5354420   -2.1686501    6.9180175    5.7732246   -0.1300798 
    ##        CRuns         CRBI       CWalks    DivisionW      PutOuts 
    ##    1.4082490    0.7743122   -0.8308264 -112.3800575    0.2973726 
    ##      Assists 
    ##    0.2831680

``` r
k=10

folds=sample(1:k,nrow(Hitters),replace=TRUE)
cv.errors=matrix(NA,k,19, dimnames=list(NULL, paste(1:19)))

# for(j in 1:k){
#   best.fit=regsubsets(Salary~.,data=Hitters[folds!=j,],nvmax=19)
#   for(i in 1:19){
#     pred=predict(best.fit,Hitters[folds==j,],id=i)
#     cv.errors[j,i]=mean( (Hitters$Salary[folds==j]-pred)^2)
#     }
#   }
# 
# mean.cv.errors=apply(cv.errors,2,mean)
# mean.cv.errors
```

### Revised

``` r
set.seed(1)

train <- sample_frac(Hitters, 0.5)
test <- setdiff(Hitters, train)
```

First, we write a function that takes a dataset and number of predictors "p\_", and does the following:

1.  runs exhaustive search for models of size "p\_" with the lowest R^2 value (using `leaps::regsubsets()` function)
2.  finds the variables that produce the best model of size "p\_"
3.  fits and returns the best linear model

``` r
best_subset_model <- function(x, p_) {
  train_data <- as_tibble(x)
  
  best_subset <- leaps::regsubsets(Salary ~ ., train_data, nvmax = p_)
  
  best_subset_coef <- coef(best_subset, p_)
  
  best_subset_var_names <- tail(names(best_subset_coef), n = -1)
  
  lm(Salary ~ ., data = train_data %>% select(Salary, one_of(best_subset_var_names)))
}
```

Then, we write a function that takes the number of predictors "p\_" for best subset and number of folds "k\_", and does the following:

-   Splits the data using k-Fold Cross-Validation (using `resamplr::crossv_kfold`)
-   Creates two new columns containing:
    -   the best model of degree "n\_" fit to the training split
    -   The "RMSE" of the model tested on the testing split
-   Returns the mean "RMSE" across all folds

``` r
best_subset_kfold <- function(p_, k_) {
  best_subset <- 
    Hitters %>% 
    resamplr::crossv_kfold(k_) %>% 
    mutate(
      model = map2(train, p_, best_subset_model),
      rmse = map2_dbl(model, test, modelr::rmse)
    )

  mean(best_subset$rmse)
}
```

First, create a dataframe with one column, "p" = the size of the model (i.e., the number of predictors used in best subset)

``` r
num_variables <- tibble(
  p = 1:19
)
```

We call the 'beson each of the desired polynomial degrees, to obtain the mean "RMSE" using k-Fold Cross-Validation on degree "n" with "k" folds.

``` r
num_variables %>% 
  mutate(
    mean_rmse = map2_dbl(p, 10, best_subset_kfold)
  )
```

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## Warning: Unknown variables: `LeagueN`, `DivisionW`, `NewLeagueN`

    ## # A tibble: 19 x 2
    ##        p mean_rmse
    ##    <int>     <dbl>
    ##  1     1  370.2391
    ##  2     2  339.9855
    ##  3     3  356.7968
    ##  4     4  340.9845
    ##  5     5  354.2618
    ##  6     6  334.7246
    ##  7     7  337.6276
    ##  8     8  336.4643
    ##  9     9  350.4384
    ## 10    10  335.9939
    ## 11    11  326.6634
    ## 12    12  327.7692
    ## 13    13  342.2026
    ## 14    14  344.2162
    ## 15    15  332.1913
    ## 16    16  344.3812
    ## 17    17  330.1365
    ## 18    18  330.5448
    ## 19    19  330.7756
