Hazard-Based Duration Models
================

#### Example: Work-to-home departure delay

A survey of 204 Seattle-area commuters was conducted to examine the
duration of time that commuters delay their work-to-home trips in an
effort to avoid peak period traffic congestion. Of the 204 commuters
surveyed, 96 indicated that they sometimes delayed their work-to-home
trip to avoid traffic congestion. These commuters provided their average
time delay. Thus, each commuter has a completed delay duration so that
neither left nor right censoring is present in the data.

**Your tasks:**

1.  Plot the Kaplan-Meier estimate of the duration of time that
    commuters delay their work-to-home trips;
2.  Determine the significant factors that affect the duration of
    commutersâ€™ delay using a Cox model;
3.  Examine the work-to-home departure delay using exponential, Weibull,
    and log-logistic proportional-hazards models.

## Get to know your data

##### Import Libraries

``` r
library(readxl)
library(skimr)
library(survival)
library(coxme)
library(survminer)
library(ggplot2)
```

##### Import dataset, transform in dataframe, and take a first look.

``` r
data.delay <- read_excel("Data/ExerciseHBDM.xlsx")
data.delay <- data.frame(data.delay)
head(data.delay) #variable names are missing
```

    ##     id X1 X2 X3 X4 X5 X6 X7 X8 X9 X10 X11 X12 X13 X14 X15   X16  X17  X18  X19
    ## 1 1000  0  0  0  1  2  1  1  1  2   0   5   1  12   1 1.4 37070 1024 1295 5610
    ## 2 1001 30  1  1  1  2  1  2  0  2   0   5   0  14   1 1.8 24410 6430 3206 1997
    ## 3 1002  0  0  0  1  5  1  4  1  1   0   3   0   5   1 1.3 24410 6430 3206 1997
    ## 4 1003  0  0  0  1  5  1  4  1  1   0   2   0   5   1 1.3 24410 6430 3206 1997
    ## 5 1004  0  0  0  2  2  1  2  1  1   0   3   0   3   0 1.2  5079 1024  899 4092
    ## 6 1005 38  1  3  1  2  1  7  1  2   0   5   0  14   1 1.8 24410 6430 3206 1997

##### Assign variable lables to each one respectively

``` r
names(data.delay) <- c("id","minutes","activity","number_of_times","mode","route","congested","age",
                        "gender","number_cars","number_children","income","flexible","distance",
                        "LOSD","rate_of_travel","population","retail","service","size") 
df <- data.frame(data.delay, row.names = 1) #make id (1st variable) as row name
```

##### Take a look at the structure

``` r
str(df)
```

    ## 'data.frame':    204 obs. of  19 variables:
    ##  $ minutes        : num  0 30 0 0 0 38 0 0 45 0 ...
    ##  $ activity       : num  0 1 0 0 0 1 0 0 1 0 ...
    ##  $ number_of_times: num  0 1 0 0 0 3 0 0 0 0 ...
    ##  $ mode           : num  1 1 1 1 2 1 2 1 2 1 ...
    ##  $ route          : num  2 2 5 5 2 2 2 1 2 5 ...
    ##  $ congested      : num  1 1 1 1 1 1 1 0 1 0 ...
    ##  $ age            : num  1 2 4 4 2 7 2 7 3 6 ...
    ##  $ gender         : num  1 0 1 1 1 1 1 1 1 1 ...
    ##  $ number_cars    : num  2 2 1 1 1 2 2 2 2 3 ...
    ##  $ number_children: num  0 0 0 0 0 0 0 0 0 1 ...
    ##  $ income         : num  5 5 3 2 3 5 6 4 4 5 ...
    ##  $ flexible       : num  1 0 0 0 0 0 0 0 0 0 ...
    ##  $ distance       : num  12 14 5 5 3 14 11 13 22 6 ...
    ##  $ LOSD           : num  1 1 1 1 0 1 1 0 1 0 ...
    ##  $ rate_of_travel : chr  "1.4" "1.8" "1.3" "1.3" ...
    ##  $ population     : num  37070 24410 24410 24410 5079 ...
    ##  $ retail         : num  1024 6430 6430 6430 1024 ...
    ##  $ service        : num  1295 3206 3206 3206 899 ...
    ##  $ size           : num  5610 1997 1997 1997 4092 ...

``` r
skim(df)
```

|                                                  |      |
|:-------------------------------------------------|:-----|
| Name                                             | df   |
| Number of rows                                   | 204  |
| Number of columns                                | 19   |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |      |
| Column type frequency:                           |      |
| character                                        | 1    |
| numeric                                          | 18   |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |      |
| Group variables                                  | None |

Data summary

**Variable type: character**

| skim\_variable   | n\_missing | complete\_rate | min | max | empty | n\_unique | whitespace |
|:-----------------|-----------:|---------------:|----:|----:|------:|----------:|-----------:|
| rate\_of\_travel |          0 |              1 |   3 |   3 |     0 |        15 |          0 |

**Variable type: numeric**

| skim\_variable    | n\_missing | complete\_rate |     mean |       sd |   p0 |   p25 |     p50 |      p75 |  p100 | hist  |
|:------------------|-----------:|---------------:|---------:|---------:|-----:|------:|--------:|---------:|------:|:------|
| minutes           |          0 |              1 |    24.14 |    36.27 |    0 |     0 |     0.0 |    30.00 |   240 | â–‡â–?â–?â–?â–? |
| activity          |          0 |              1 |     0.78 |     0.97 |    0 |     0 |     0.0 |     1.00 |     3 | â–‡â–?â–?â–‚â–? |
| number\_of\_times |          0 |              1 |     0.86 |     1.31 |    0 |     0 |     0.0 |     2.00 |     5 | â–‡â–‚â–?â–?â–? |
| mode              |          0 |              1 |     2.08 |     1.49 |    1 |     1 |     1.0 |     4.00 |     5 | â–‡â–‚â–?â–‚â–‚ |
| route             |          0 |              1 |     3.49 |     1.46 |    1 |     2 |     3.5 |     5.00 |     5 | â–‚â–†â–‚â–‚â–‡ |
| congested         |          0 |              1 |     0.63 |     0.48 |    0 |     0 |     1.0 |     1.00 |     1 | â–…â–?â–?â–?â–‡ |
| age               |          0 |              1 |     2.83 |     1.69 |    1 |     2 |     2.0 |     4.00 |     7 | â–‡â–?â–‚â–?â–? |
| gender            |          0 |              1 |     0.64 |     0.48 |    0 |     0 |     1.0 |     1.00 |     1 | â–…â–?â–?â–?â–‡ |
| number\_cars      |          0 |              1 |     1.80 |     1.04 |    0 |     1 |     2.0 |     2.00 |     7 | â–‡â–‡â–‚â–?â–? |
| number\_children  |          0 |              1 |     0.70 |     1.05 |    0 |     0 |     0.0 |     1.00 |     5 | â–‡â–‚â–?â–?â–? |
| income            |          0 |              1 |     2.78 |     1.56 |    1 |     1 |     3.0 |     4.00 |     6 | â–‡â–?â–‚â–‚â–? |
| flexible          |          0 |              1 |     0.62 |     0.49 |    0 |     0 |     1.0 |     1.00 |     1 | â–…â–?â–?â–?â–‡ |
| distance          |          0 |              1 |     7.15 |     4.81 |    1 |     4 |     6.0 |    10.00 |    25 | â–‡â–†â–?â–?â–? |
| LOSD              |          0 |              1 |     0.65 |     0.48 |    0 |     0 |     1.0 |     1.00 |     1 | â–…â–?â–?â–?â–‡ |
| population        |          0 |              1 | 25367.32 | 10232.19 | 1303 | 23026 | 24410.0 | 34895.00 | 37070 | â–?â–?â–?â–‡â–‡ |
| retail            |          0 |              1 |  4604.66 |  4334.42 |  616 |  1866 |  3906.0 |  3966.00 | 16523 | â–†â–‡â–?â–?â–‚ |
| service           |          0 |              1 |  9733.06 | 10552.66 |  595 |  1606 | 10582.0 | 10582.00 | 38607 | â–‡â–‡â–?â–?â–‚ |
| size              |          0 |              1 |  3087.51 |  1598.03 |  475 |  2472 |  2753.0 |  4447.75 |  5653 | â–?â–‡â–‡â–?â–† |

##### Plot yout data

``` r
df <- df[order(df$minutes),] #sort by time
plot(df$minutes, type="h") #high-density vertical lines
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-5-1.png)<!-- -->

## Survival function

### 1.Plot the Kaplan-Meier estimate curve

##### Create the life table survival object for df

``` r
data.delay2 <-subset(df, minutes>0)
df.survfit = survfit(Surv(minutes) ~ 1, data= data.delay2)
summary(df.survfit)
```

    ## Call: survfit(formula = Surv(minutes) ~ 1, data = data.delay2)
    ## 
    ##  time n.risk n.event survival std.err lower 95% CI upper 95% CI
    ##     4     96       1   0.9896  0.0104      0.96948       1.0000
    ##    10     95       4   0.9479  0.0227      0.90450       0.9934
    ##    15     91       7   0.8750  0.0338      0.81128       0.9437
    ##    20     84       1   0.8646  0.0349      0.79878       0.9358
    ##    24     83       1   0.8542  0.0360      0.78640       0.9278
    ##    25     82       1   0.8438  0.0371      0.77416       0.9196
    ##    30     81      31   0.5208  0.0510      0.42990       0.6310
    ##    38     50       1   0.5104  0.0510      0.41961       0.6209
    ##    40     49       3   0.4792  0.0510      0.38897       0.5903
    ##    45     46      10   0.3750  0.0494      0.28965       0.4855
    ##    53     36       1   0.3646  0.0491      0.27997       0.4748
    ##    60     35      16   0.1979  0.0407      0.13231       0.2961
    ##    80     19       1   0.1875  0.0398      0.12364       0.2843
    ##    90     18       7   0.1146  0.0325      0.06571       0.1998
    ##   100     11       1   0.1042  0.0312      0.05794       0.1873
    ##   105     10       1   0.0937  0.0297      0.05033       0.1746
    ##   120      9       6   0.0312  0.0178      0.01026       0.0952
    ##   130      3       1   0.0208  0.0146      0.00529       0.0821
    ##   150      2       1   0.0104  0.0104      0.00148       0.0732
    ##   240      1       1   0.0000     NaN           NA           NA

> **Note:** The functions `survfit()` and `Surv()` create a life table
> survival object.

##### Plot the Kaplan-Meier curve

``` r
# option 1
plot(
  df.survfit,
  xlab = "Time (minutes)",
  ylab = "Survival
probability",
conf.int = TRUE
)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-7-1.png)<!-- -->

``` r
#option 2
ggsurvplot(
  df.survfit,
  xlab = "Time (minutes)",
  xlim =
    range(0:250) ,
  conf.int = TRUE,
  pallete = "red",
  ggtheme =
    theme_minimal()
)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-8-1.png)<!-- -->

> **Note:** It is the most widely applied nonparametric method in
> survival analysis.The Kaplanâ€“Meier method provides useful estimates of
> survival probabilities and a graphical presentation of the survival
> distribution.  
> The Kaplanâ€“Meier method assumes that censoring is independent of
> survival times. If this is false, the Kaplanâ€“Meier method is
> inappropriate.

### 2. Cox proportional-hazards Model

The Cox proportional-hazards model is semiparametric method that
produces estimated hazard ratios (sometimes called rate ratios or risk
ratios).  
Letâ€™s perform a Cox Proportional Hazard Model Estimate estimate the
model for Duration of Commuter Work-To-Home Delay to Avoid Congestion

``` r
result.cox <- coxph(Surv(minutes) ~ gender + rate_of_travel + distance + population, data= data.delay2)
summary(result.cox)
```

    ## Call:
    ## coxph(formula = Surv(minutes) ~ gender + rate_of_travel + distance + 
    ##     population, data = data.delay2)
    ## 
    ##   n= 96, number of events= 96 
    ## 
    ##                         coef  exp(coef)   se(coef)      z Pr(>|z|)  
    ## gender             1.617e-01  1.176e+00  2.608e-01  0.620   0.5352  
    ## rate_of_travel1.3 -3.625e-01  6.960e-01  1.280e+00 -0.283   0.7771  
    ## rate_of_travel1.4  1.233e+00  3.430e+00  1.251e+00  0.985   0.3245  
    ## rate_of_travel1.5 -5.127e-01  5.989e-01  1.186e+00 -0.432   0.6656  
    ## rate_of_travel1.6  3.660e-02  1.037e+00  1.147e+00  0.032   0.9746  
    ## rate_of_travel1.7  1.018e+00  2.767e+00  1.341e+00  0.759   0.4478  
    ## rate_of_travel1.8 -8.239e-01  4.387e-01  1.093e+00 -0.754   0.4510  
    ## rate_of_travel1.9 -9.901e-01  3.716e-01  1.142e+00 -0.867   0.3860  
    ## rate_of_travel2.0 -9.167e-01  3.998e-01  1.115e+00 -0.822   0.4112  
    ## rate_of_travel2.1 -1.468e+00  2.303e-01  1.086e+00 -1.353   0.1762  
    ## rate_of_travel2.2 -1.138e+00  3.204e-01  1.119e+00 -1.017   0.3089  
    ## rate_of_travel2.3 -1.853e+00  1.568e-01  1.109e+00 -1.670   0.0949 .
    ## rate_of_travel2.4 -1.515e+00  2.197e-01  1.117e+00 -1.357   0.1748  
    ## rate_of_travel2.5 -1.510e+00  2.208e-01  1.114e+00 -1.355   0.1753  
    ## distance          -7.486e-02  9.279e-01  3.403e-02 -2.200   0.0278 *
    ## population        -2.157e-05  1.000e+00  1.319e-05 -1.635   0.1020  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                   exp(coef) exp(-coef) lower .95 upper .95
    ## gender               1.1755     0.8507   0.70512    1.9597
    ## rate_of_travel1.3    0.6960     1.4369   0.05662    8.5547
    ## rate_of_travel1.4    3.4301     0.2915   0.29538   39.8330
    ## rate_of_travel1.5    0.5989     1.6698   0.05858    6.1226
    ## rate_of_travel1.6    1.0373     0.9641   0.10950    9.8255
    ## rate_of_travel1.7    2.7666     0.3615   0.19994   38.2812
    ## rate_of_travel1.8    0.4387     2.2793   0.05150    3.7373
    ## rate_of_travel1.9    0.3716     2.6914   0.03962    3.4847
    ## rate_of_travel2.0    0.3998     2.5010   0.04492    3.5593
    ## rate_of_travel2.1    0.2303     4.3425   0.02743    1.9335
    ## rate_of_travel2.2    0.3204     3.1208   0.03578    2.8698
    ## rate_of_travel2.3    0.1568     6.3758   0.01784    1.3792
    ## rate_of_travel2.4    0.2197     4.5514   0.02462    1.9612
    ## rate_of_travel2.5    0.2208     4.5287   0.02486    1.9616
    ## distance             0.9279     1.0777   0.86801    0.9919
    ## population           1.0000     1.0000   0.99995    1.0000
    ## 
    ## Concordance= 0.677  (se = 0.036 )
    ## Likelihood ratio test= 30.65  on 16 df,   p=0.01
    ## Wald test            = 32.27  on 16 df,   p=0.009
    ## Score (logrank) test = 36  on 16 df,   p=0.003

> **Note:** Regression coefficients are on a log scale.

##### Testing proportional Hazards assumption

Test the proportional hazards assumption on the basis of partial
residuals. Type of residual known as Schoenfeld residuals.  
It includes an interaction between the covariate and a function of time
(or distance). Log time is often used but it could be any function. For
each covariate, the function `cox.zph()` correlates the corresponding
set of scaled Schoenfeld residuals with time, to test for independence
between residuals and time. Additionally, it performs a global test for
the model as a whole.  
A plot that shows a non-random pattern against time is evidence of
violation of the PH assumption.

``` r
test.ph <- cox.zph(result.cox)
par(mfrow=c(2,2))
plot(test.ph)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-10-1.png)<!-- -->

``` r
ggcoxzph(test.ph)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-10-2.png)<!-- -->

> **Note:** If significant then the assumption is violated.  
> In principle, the Schoenfeld residuals are independent of time.

##### Plot the baseline survival function

``` r
ggsurvplot(
  survfit(result.cox),
  data = data.delay2,
  palette = "#2E9FDF",
  ggtheme = theme_minimal()
)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-11-1.png)<!-- -->

##### Plot the cummulative hazard function

``` r
ggsurvplot(
  survfit(result.cox),
  data = data.delay2,
  conf.int = TRUE,
  palette = c("#FF9E29", "#86AA00"),
  risk.table = TRUE,
  risk.table.col = "strata",
  fun = "event"
)
```

![](RmdFiles/9-HazardBasedModels/unnamed-chunk-12-1.png)<!-- -->

##### Calculate the McFadden Pseudo R-square

``` r
result.cox$loglik
```

    ## [1] -345.3794 -330.0545

``` r
LLi = result.cox$loglik[1] # Initial log-likelihood
LLf = result.cox$loglik[2] # Final log-likelihood
1- (LLf/LLi) # pseudo R-suare
```

    ## [1] 0.0443713

### 3. Exponential, Weibull, and log-logistic proportional-hazards models

Parametric Model Estimates of the Duration of Commuter Work-ToHome Delay
to Avoid Congestion

-   **Exponential**

``` r
survreg(
  Surv(minutes) ~ gender + rate_of_travel + distance + population,
  data = data.delay2,
  dist = "exponential" )
```

    ## Call:
    ## survreg(formula = Surv(minutes) ~ gender + rate_of_travel + distance + 
    ##     population, data = data.delay2, dist = "exponential")
    ## 
    ## Coefficients:
    ##       (Intercept)            gender rate_of_travel1.3 rate_of_travel1.4 rate_of_travel1.5 rate_of_travel1.6 
    ##      2.879512e+00     -1.108216e-01      2.286504e-01     -6.243518e-01      2.089118e-01     -7.778191e-02 
    ## rate_of_travel1.7 rate_of_travel1.8 rate_of_travel1.9 rate_of_travel2.0 rate_of_travel2.1 rate_of_travel2.2 
    ##     -5.431004e-01      3.753339e-01      4.918786e-01      4.186932e-01      6.440768e-01      5.006781e-01 
    ## rate_of_travel2.3 rate_of_travel2.4 rate_of_travel2.5          distance        population 
    ##      9.390733e-01      7.377887e-01      8.025575e-01      4.070892e-02      1.261692e-05 
    ## 
    ## Scale fixed at 1 
    ## 
    ## Loglik(model)= -467.8   Loglik(intercept only)= -474
    ##  Chisq= 12.5 on 16 degrees of freedom, p= 0.709 
    ## n= 96

-   **Weibull**

``` r
survreg(
  Surv(minutes) ~ gender + rate_of_travel + distance + population,
  data = data.delay2,
  dist = "weibull" )
```

    ## Call:
    ## survreg(formula = Surv(minutes) ~ gender + rate_of_travel + distance + 
    ##     population, data = data.delay2, dist = "weibull")
    ## 
    ## Coefficients:
    ##       (Intercept)            gender rate_of_travel1.3 rate_of_travel1.4 rate_of_travel1.5 rate_of_travel1.6 
    ##      2.895928e+00     -9.903521e-02      2.181794e-01     -6.093365e-01      2.762764e-01     -3.648657e-03 
    ## rate_of_travel1.7 rate_of_travel1.8 rate_of_travel1.9 rate_of_travel2.0 rate_of_travel2.1 rate_of_travel2.2 
    ##     -4.933829e-01      4.904227e-01      5.528334e-01      5.234497e-01      8.567097e-01      6.509833e-01 
    ## rate_of_travel2.3 rate_of_travel2.4 rate_of_travel2.5          distance        population 
    ##      1.210978e+00      8.503809e-01      8.828783e-01      3.939630e-02      1.222171e-05 
    ## 
    ## Scale= 0.5153148 
    ## 
    ## Loglik(model)= -442.3   Loglik(intercept only)= -461.8
    ##  Chisq= 38.9 on 16 degrees of freedom, p= 0.00112 
    ## n= 96

-   **Log-logistic**

``` r
survreg(
  Surv(minutes) ~ gender + rate_of_travel + distance + population,
  data = data.delay2,
  dist = "loglogistic" ) 
```

    ## Call:
    ## survreg(formula = Surv(minutes) ~ gender + rate_of_travel + distance + 
    ##     population, data = data.delay2, dist = "loglogistic")
    ## 
    ## Coefficients:
    ##       (Intercept)            gender rate_of_travel1.3 rate_of_travel1.4 rate_of_travel1.5 rate_of_travel1.6 
    ##      2.892950e+00     -1.442251e-01      2.314555e-01     -6.226149e-01      1.475325e-01     -1.335428e-01 
    ## rate_of_travel1.7 rate_of_travel1.8 rate_of_travel1.9 rate_of_travel2.0 rate_of_travel2.1 rate_of_travel2.2 
    ##     -5.913971e-01      2.521084e-01      4.608343e-01      2.887788e-01      3.701638e-01      3.211386e-01 
    ## rate_of_travel2.3 rate_of_travel2.4 rate_of_travel2.5          distance        population 
    ##      5.197213e-01      6.084442e-01      7.333592e-01      4.171315e-02      1.217428e-05 
    ## 
    ## Scale= 0.3326078 
    ## 
    ## Loglik(model)= -442.2   Loglik(intercept only)= -456.9
    ##  Chisq= 29.45 on 16 degrees of freedom, p= 0.0211 
    ## n= 96

> **Note:** The argument *dist* has several options to describe the
> parametric model used (â€śweibullâ€ť, â€śexponentialâ€ť, â€śgaussianâ€ť,
> â€ślogisticâ€ť, â€ślognormalâ€ť, or â€śloglogisticâ€ť. See more with `?survreg`
