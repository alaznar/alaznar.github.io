Survival Analysis in Breast Cancer patients
================

# Introduction

Breast Cancer is one of the most common cancer type worldwide. Depending
on the type of cancer, age, or other biomarkers, the prognosis of a
patient might vary.

In this project, we will explore th GBSG2 dataset, from the pec package.
Thios dataset contains information of over 600 female patients who
suffered Breast Cancer. The objetive is to develop a Survioval Analysis
model that can explain which features are relevant and to which extent.
In clinical practice, it is quite common to report Hazard Ratios, that
can be estimated from Cox models.

The Cox proportional hazards model is a regression model that allows for
comparing the effect of various variables on survival. It is such a
widely used model because it accommodates multiple covariates, something
especially important in clinical trials, where multidimensional datasets
are typically involved. An HR value above 1 indicates that, if the value
of the i-th covariate increases, the risk of the event occurring also
increases, and therefore the survival curve will be reduced. Thus, the
HR makes it possible to determine whether a covariate increases the risk
of the event, reduces it, or, if its value is close to 1, indicates that
it has no significant effect on that risk. For categorical value, this
risk is compared to a reference (typically, one of the values of that
categorical variable).

``` r
library(pec)
```

    ## Cargando paquete requerido: prodlim

``` r
library(DataExplorer)
library(summarytools)
library(ggplot2)
library(dplyr)
```

    ## 
    ## Adjuntando el paquete: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(purrr)
library(tidyr)
library(patchwork)
library(corrplot)
```

    ## corrplot 0.95 loaded

``` r
library(survival)
library(survminer)
```

    ## Cargando paquete requerido: ggpubr

    ## 
    ## Adjuntando el paquete: 'survminer'

    ## The following object is masked from 'package:survival':
    ## 
    ##     myeloma

``` r
library(riskRegression)
```

    ## riskRegression version 2023.12.21

    ## 
    ## Adjuntando el paquete: 'riskRegression'

    ## The following objects are masked from 'package:pec':
    ## 
    ##     ipcw, selectCox

``` r
library(rms)
```

    ## Cargando paquete requerido: Hmisc

    ## 
    ## Adjuntando el paquete: 'Hmisc'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     src, summarize

    ## The following objects are masked from 'package:summarytools':
    ## 
    ##     label, label<-

    ## The following objects are masked from 'package:base':
    ## 
    ##     format.pval, units

``` r
library(broom)
library(randomForestSRC)
```

    ## 
    ##  randomForestSRC 3.3.3 
    ##  
    ##  Type rfsrc.news() to see new features, changes, and bug fixes. 
    ## 

    ## 
    ## Adjuntando el paquete: 'randomForestSRC'

    ## The following object is masked from 'package:Hmisc':
    ## 
    ##     impute

    ## The following object is masked from 'package:purrr':
    ## 
    ##     partial

``` r
library(splines)
library(stringr)
library(forcats)
```

# EDA

You can include R code in the document as follows:

``` r
data(GBSG2, package = "pec")
max(GBSG2$time) / 365 # years
```

    ## [1] 7.284932

``` r
summary(GBSG2)
```

    ##  horTh          age        menostat       tsize        tgrade   
    ##  no :440   Min.   :21.00   Post:396   Min.   :  3.00   I  : 81  
    ##  yes:246   1st Qu.:46.00   Pre :290   1st Qu.: 20.00   II :444  
    ##            Median :53.00              Median : 25.00   III:161  
    ##            Mean   :53.05              Mean   : 29.33            
    ##            3rd Qu.:61.00              3rd Qu.: 35.00            
    ##            Max.   :80.00              Max.   :120.00            
    ##      pnodes         progrec           estrec             time       
    ##  Min.   : 1.00   Min.   :   0.0   Min.   :   0.00   Min.   :   8.0  
    ##  1st Qu.: 1.00   1st Qu.:   7.0   1st Qu.:   8.00   1st Qu.: 567.8  
    ##  Median : 3.00   Median :  32.5   Median :  36.00   Median :1084.0  
    ##  Mean   : 5.01   Mean   : 110.0   Mean   :  96.25   Mean   :1124.5  
    ##  3rd Qu.: 7.00   3rd Qu.: 131.8   3rd Qu.: 114.00   3rd Qu.:1684.8  
    ##  Max.   :51.00   Max.   :2380.0   Max.   :1144.00   Max.   :2659.0  
    ##       cens       
    ##  Min.   :0.0000  
    ##  1st Qu.:0.0000  
    ##  Median :0.0000  
    ##  Mean   :0.4359  
    ##  3rd Qu.:1.0000  
    ##  Max.   :1.0000

``` r
# NAs?
plot_missing(GBSG2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
# See structure
str(GBSG2)
```

    ## 'data.frame':    686 obs. of  10 variables:
    ##  $ horTh   : Factor w/ 2 levels "no","yes": 1 2 2 2 1 1 2 1 1 1 ...
    ##  $ age     : int  70 56 58 59 73 32 59 65 80 66 ...
    ##  $ menostat: Factor w/ 2 levels "Post","Pre": 1 1 1 1 1 2 1 1 1 1 ...
    ##  $ tsize   : int  21 12 35 17 35 57 8 16 39 18 ...
    ##  $ tgrade  : Factor w/ 3 levels "I","II","III": 2 2 2 2 2 3 2 2 2 2 ...
    ##  $ pnodes  : int  3 7 9 4 1 24 2 1 30 7 ...
    ##  $ progrec : int  48 61 52 60 26 0 181 192 0 0 ...
    ##  $ estrec  : int  66 77 271 29 65 13 0 25 59 3 ...
    ##  $ time    : int  1814 2018 712 1807 772 448 2172 2161 471 2014 ...
    ##  $ cens    : int  1 1 1 1 1 1 0 0 1 0 ...

``` r
head(GBSG2)
```

    ##   horTh age menostat tsize tgrade pnodes progrec estrec time cens
    ## 1    no  70     Post    21     II      3      48     66 1814    1
    ## 2   yes  56     Post    12     II      7      61     77 2018    1
    ## 3   yes  58     Post    35     II      9      52    271  712    1
    ## 4   yes  59     Post    17     II      4      60     29 1807    1
    ## 5    no  73     Post    35     II      1      26     65  772    1
    ## 6    no  32      Pre    57    III     24       0     13  448    1

``` r
print(dfSummary(GBSG2), method = "render")
```

<div class="container st-container">
<h3>Data Frame Summary</h3>
<h4>GBSG2</h4>
<strong>Dimensions</strong>: 686 x 10
  <br/><strong>Duplicates</strong>: 0
<br/>
<table class="table table-striped table-bordered st-table st-table-striped st-table-bordered st-multiline ">
  <thead>
    <tr>
      <th align="center" class="st-protect-top-border"><strong>No</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Variable</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Stats / Values</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Freqs (% of Valid)</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Graph</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Valid</strong></th>
      <th align="center" class="st-protect-top-border"><strong>Missing</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">1</td>
      <td align="left">horTh
[factor]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">1. no</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">2. yes</td></tr></table></td>
      <td align="left" style="padding:0;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">440</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">64.1%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">246</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">35.9%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr></table></td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAG0AAAA3BAMAAAD0w9n0AAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAQ0lEQVRIx2NgGP5AiUSgANWnbEwaGNU3qm9U36g+YvWRWy4JkggEBkgfHq/g1Yc7PI1G9Y3qG9U3qo8ifeSWS8MZAADbIWjKOJfxCwAAAABJRU5ErkJggg=="></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">2</td>
      <td align="left">age
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 53.1 (10.1)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">21 &le; 53 &le; 80</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 15 (0.2)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">54 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAArklEQVRo3u3ZQQ6DIBCFYa4gNyjeoNz/bm0ZE8cQtDCzKPZ/CxeIX8YY0QwhkJEsZ0kly3W+wdb8zhMMbGYsllfi4YR9BjIYGNgopr5Rdizv9f0kJjfrhKkjGBgY2ESYLIVOWGEyGBgYGBjYNFiqGhIGbK3qA/PD4v7L4oCp+WBg/4RF1QexY9V8MLB7YXojz4zJSbB+TB7DAVMrWScmwxvW6IaMYY2rLJVZE8hIXgmiV0oPGO5xAAAAAElFTkSuQmCC"></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">3</td>
      <td align="left">menostat
[factor]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">1. Post</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">2. Pre</td></tr></table></td>
      <td align="left" style="padding:0;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">396</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">57.7%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">290</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">42.3%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr></table></td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAGMAAAA3BAMAAADqCulHAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAP0lEQVRIx2NgGF5AiXigANWibEw0GNUyqmVUyzDSQkZxIUg8EKCrFkIewKKFQIgZjWoZ1TKqZXhqIaO4GC4AAEhoWS8PHW8sAAAAAElFTkSuQmCC"></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">4</td>
      <td align="left">tsize
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 29.3 (14.3)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">3 &le; 25 &le; 120</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 15 (0.5)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">58 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAjklEQVRo3u3ZQQqAIBBAUa9gN0hvkPe/WzQuLIoabVpo/y9cBD0KRZKco5Z8aQqSb+gCS1sLGBgYGBgYGBgYGBjYQFg+NRlhURgwMLDfYLKBzEaYjGBgYGBgYGAjYKHuI+Eei3XP1zem/qulwfJlsC+xPGFG2P6u7jDF0tVj6ZmsxVJ55fP+dJjNtzlqaQW745HM4W5k1gAAAABJRU5ErkJggg=="></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">5</td>
      <td align="left">tgrade
[factor]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">1. I</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">2. II</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">3. III</td></tr></table></td>
      <td align="left" style="padding:0;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">81</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">11.8%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">444</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">64.7%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr><tr style="background-color:transparent"><td style="padding:0 5px 0 7px;margin:0;border:0" align="right">161</td><td style="padding:0 2px 0 0;border:0;" align="left">(</td><td style="padding:0;border:0" align="right">23.5%</td><td style="padding:0 4px 0 2px;border:0" align="left">)</td></tr></table></td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAG0AAABOBAMAAADY7oX9AAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAW0lEQVRYw+3UsQ3AIAwFUUaADZCzAdl/NwqgIAjJsWiM7mq/ytIP4f5klHTF7p63VXA4HG7jrPuiPHfr5Gf58wdtOBzOr7PuhJcdPLefOrf8oeBwONzsrPtycxXmkaICcWWbTQAAAABJRU5ErkJggg=="></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">6</td>
      <td align="left">pnodes
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 5 (5.5)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">1 &le; 3 &le; 51</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 6 (1.1)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">30 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAbElEQVRo3u3ZsQ2AIBBAUVZwBHED2X83otBYWHhQGH2/oHy5iksgJUXKrWWsjm3laIfBYDAYDAaDwWAwGAwGg8FgMNiHsMF35Cs2OB8MBoO9DGtX5DoHO8/yEyzwS3mPBbYf7PmS71ieUlKkChrPV7ilCdnWAAAAAElFTkSuQmCC"></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">7</td>
      <td align="left">progrec
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 110 (202.3)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">0 &le; 32.5 &le; 2380</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 124.8 (1.8)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">242 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAZUlEQVRo3u3ZsQ2AMAwAwazACDEbkP13QwQkGihiIqr7wuXJhTuXokzRq8u3LmxtRzAYDAaDwWAwGAwGg8FgMBgMBoPBYDDYDOx8ZU3C+txgsBcsf20PWEvv9w8Wd3UQiykVZdoB3Tg6UBadYFQAAAAASUVORK5CYII="></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">8</td>
      <td align="left">estrec
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 96.3 (153.1)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">0 &le; 36 &le; 1144</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 106 (1.6)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">244 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAdElEQVRo3u3ZvQmAMBCA0azgCMYNzP67iTGF4E9xBkF5X5HyccWR5lJSpFwbh2c1bCprMBgMBoPBYDAYDAaDwWAwGAwGg8FgP8G2S0onrL4zDHaBxbftBCvh+T6IRQ6el1jkG3kRy7seY8f1u4UblruUFGkB6ctHD7vwGIUAAAAASUVORK5CYII="></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">9</td>
      <td align="left">time
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean (sd) : 1124.5 (642.8)</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">min &le; med &le; max:</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">8 &le; 1084 &le; 2659</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">IQR (CV) : 1117 (0.6)</td></tr></table></td>
      <td align="left" style="vertical-align:middle">574 distinct values</td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAJgAAABuBAMAAAApJ8cWAAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAuklEQVRo3u3XUQ6EIAxFUbagOxjdgex/b6M1osOgkdIY0fs++DDhpCEtqnNEkyakm/JpVPnDej8GDKw6TDkGaUxZHhgYWHVY/tVxgOWX9xSslXO0wqZHQ+WYHIgVFjlvx3ab7RQWds/YbnmnsH7ZfScs9J4FFhYwsEswaV8rzP8uYPlY6rpVY6nywNRY6jNDjR06YGBgYM/HovddGRaVBwYGVoitA2qAreWBgdWEyRhYYbJsfyqK44gmX8CUwdpALxrcAAAAAElFTkSuQmCC"></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
    <tr>
      <td align="center">10</td>
      <td align="left">cens
[integer]</td>
      <td align="left" style="padding:8;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Min  : 0</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Mean : 0.4</td></tr><tr style="background-color:transparent"><td style="padding:0;margin:0;border:0" align="left">Max  : 1</td></tr></table></td>
      <td align="left" style="padding:0;vertical-align:middle"><table style="border-collapse:collapse;border:none;margin:0"><tr style="background-color:transparent"><td style="padding:0 2px 0 7px;margin:0;border:0" align="right">0</td><td style="padding:0 2px;border:0;" align="left">:</td><td style="padding:0 4px 0 6px;margin:0;border:0" align="right">387</td><td style="padding:0;border:0" align="left">(</td><td style="padding:0 2px;margin:0;border:0" align="right">56.4%</td><td style="padding:0 4px 0 0;border:0" align="left">)</td></tr><tr style="background-color:transparent"><td style="padding:0 2px 0 7px;margin:0;border:0" align="right">1</td><td style="padding:0 2px;border:0;" align="left">:</td><td style="padding:0 4px 0 6px;margin:0;border:0" align="right">299</td><td style="padding:0;border:0" align="left">(</td><td style="padding:0 2px;margin:0;border:0" align="right">43.6%</td><td style="padding:0 4px 0 0;border:0" align="left">)</td></tr></table></td>
      <td align="left" style="vertical-align:middle;padding:0;background-color:transparent;"><img style="border:none;background-color:transparent;padding:0;max-width:max-content;" src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAGEAAAA3BAMAAADu/zl6AAAAIGNIUk0AAHomAACAhAAA+gAAAIDoAAB1MAAA6mAAADqYAAAXcJy6UTwAAAAPUExURf////3+/aampvDw8P///7GHRIEAAAACdFJOUwAAdpPNOAAAAAFiS0dEAIgFHUgAAAAHdElNRQfpCBQOIR8ZnV7/AAAAP0lEQVRIx2NgGD5AiWigANWhbEwsGNUxqmNUx1DVQXrJIEg0EKCjDoLOx9BBKKyMRnWM6hjVMeR1kF4yDAcAAPMIVhDz6gFOAAAAAElFTkSuQmCC"></td>
      <td align="center">686
(100.0%)</td>
      <td align="center">0
(0.0%)</td>
    </tr>
  </tbody>
</table>
<p>Generated by <a href='https://github.com/dcomtois/summarytools'>summarytools</a> 1.1.3 (<a href='https://www.r-project.org/'>R</a> version 4.4.3)<br/>2025-08-20</p>
</div>

``` r
# Violin plots / count plot
num_vars <- GBSG2 %>% select(where(is.numeric)) %>% names()
cat_vars <- GBSG2 %>% select(where(is.factor)) %>% names()
num_vars <- setdiff(num_vars, c("time","cens")) # Exclude status and T2E from graphs

plots_num <- map(num_vars, function(var) {
  ggplot(GBSG2, aes(x = "", y = .data[[var]])) +
    geom_violin(fill = "skyblue", alpha = 0.6) +
    geom_boxplot(width = 0.1, outlier.colour = "red", alpha = 0.5) +
    labs(title = paste("Violin plot:", var), y = var, x = "") +
    theme_minimal()
})

plots_cat <- map(cat_vars, function(var) {
  ggplot(GBSG2, aes(x = .data[[var]])) +
    geom_bar(fill = "coral", alpha = 0.7) +
    labs(title = paste("Frequency:", var), x = var, y = "Count") +
    theme_minimal()
})

all_plots <- c(plots_num, plots_cat)
wrap_plots(all_plots, ncol = 2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

We can see that the variables in this dataset cobntain the following
features, with no missing values: - horTh: hormonal therapy, a factor
with levels no yes - age: age in years - menostat: menopausal status, a
factor with levels Pre Post - tsize: tumor size (in mm) - tgrade: an
ordered factor with levels I \< II \< III - pnodes: number of positive
nodes - progrec: progesterone receptor (in fmol). - estrec: estrogen
receptor (in fmol). - time: recurrence free survival time (in days). -
cens: censoring indicator (0- censored, 1- event).

We see that progrec and estrec variables are quite skewed, it might be
interesting to convert them to a log scale. Some patients have values of
0, so we have to take that into account when transforming the variable

``` r
# Transform variables
GBSG2$log_estrec  <- log1p(GBSG2$estrec)   # log(1+ER)
GBSG2$log_progrec <- log1p(GBSG2$progrec)  # log(1+PR)
```

Let’s now check if there is any high correlation in the dataset:

``` r
# Correlation map (Pearson)
num_data <- GBSG2 %>% 
  select(where(is.numeric)) %>% 
  select(-time, -cens, -progrec, -estrec)

corr_matrix <- cor(num_data, use = "complete.obs")

# Plot
corrplot(corr_matrix, method = "color", type = "upper",
         addCoef.col = "black",
         tl.col = "black", tl.srt = 45)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

There might be some correlation with PR and ER receptors, but I believe
is nothing too worrying (\<0.7). I expect that ER-/PR- patients have 0
values for both, displaying a high correlaction.

# Kaplan-Meier curves

Kaplan-Meier (KM) estimator is an univariate, yet useful, technique, to
understand survival. Let’s check the overall survival of the patients in
the study, as well as these patients stratified by some variables, such
as menopausal status, tumour stage or if they have received hormonal
treatment.

The statistical test that allows the comparaison between different
curves is the Log-rank test. The null hypothesis of this tests is that
the survival curves as estimated with the KM method are equal. In the
following graphs, if the p-value is significant, then we can assume that
the survival curves are different.

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

As we can see from these plots, it seems like the menopausal status of
the patient does not impact overall survival. On the other hand, we can
see that patients that underwent hormone treatmente haver a better
prognostic, although this might not be true for PR-/ER- patients. Also,
there is a significant difference of overall survival for patients with
a stage 1 tumour, compared to patients with stage 2 or 3. Overall
survival for patients with a stage 2 tumoursseem to be better at early
stages, however, as time goes by, the overall survival is similar to
stage 3 patients.

# Cox model

As commented earlier, Cox models are able to accommodate several
variables, being a multivariate model. This allows to take into
consideration multiple features at once, either as confounding variables
or relevant outcomes for clinical interpretation. This model makes a
fundamental assumption, namely that the risk of experiencing the event
between two groups is proportional over time, so whenever we build the
model, it is crucial to check this assumption so we make sure our model
is roobust enough and oyur results are interpretable. This is the
Proportional Hazards (PH) assumption.

Let’s start by building our reference model!

``` r
cox0 <- coxph(
  surv_obj ~ age + menostat + tsize + tgrade + pnodes +
    log_progrec + log_estrec + horTh, data = GBSG2
)
```

We can assess the proportional hazards (PH) assumption by examining the
Schoenfeld residuals plotted against transformed time. Very small
p-values suggest the presence of time-dependent coefficients that need
to be addressed. Importantly, the proportionality assumption does not
test for linearity—since the Cox PH model is semi-parametric, it does
not impose a specific form on the hazard function. Instead, the
assumption concerns whether an individual’s hazard ratio remains
relatively constant over time, which is precisely what the `cox.zph()`
test evaluates.

``` r
zph <- cox.zph(cox0, transform = "km")
zph
```

    ##              chisq df       p
    ## age         11.058  1 0.00088
    ## menostat     5.502  1 0.01899
    ## tsize        0.224  1 0.63577
    ## tgrade      10.355  2 0.00564
    ## pnodes       0.700  1 0.40271
    ## log_progrec  6.371  1 0.01160
    ## log_estrec  12.745  1 0.00036
    ## horTh        0.178  1 0.67293
    ## GLOBAL      25.832  9 0.00218

``` r
plot(zph, var = 1) # age
abline(h = 0, col =2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
plot(zph, var = 2) # menostat
abline(h = 0, col =2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

``` r
plot(zph, var = 4) # tgrade
abline(h = 0, col =2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-7-3.png)<!-- -->

``` r
plot(zph, var = 6) # progrec
abline(h = 0, col =2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-7-4.png)<!-- -->

``` r
plot(zph, var = 7) # estrec
abline(h = 0, col =2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-7-5.png)<!-- -->

As we can see in the test, there are a few significant variables,
indicating that these variables are not following the PH assumption. We
can plot the Schoenfeld residuals (check
<https://stats.stackexchange.com/questions/547078/schoenfeld-residuals-plain-english-explanation-please>)
against time. I’ve plotted those variables where we suspect there is
violation of thee assumption. If these variables were constant over time
(i.e. following the PH assumption), we would see a straight line near 0.
However, we see that the smoothed line in these cases is not really a
straigt line near 0.

This assumption is violated in most datasets I’ve worked with. Therer
are several ways of overcoming this issue: stratifying data, creating a
Cox extended model with Time Dependent variables or by creating
step-functions and splitting time spans, so we couldn have different HRs
for different time-splits.

Let’s check how stratification would work. For this, we need a factor
variable (what we will be doing in this example) or by stratifying a
continous variable. This latter example might be interesting in some
scenarios (i.e. high levels, medium levels and highh levels) and provide
great interpretability to the model. In this case, we will stratify by
tumour stage and menstrual status. Before creating this strata, it is
important to check if we have enough data in all groups

``` r
table(GBSG2$menostat, GBSG2$tgrade)
```

    ##       
    ##          I  II III
    ##   Post  48 261  87
    ##   Pre   33 183  74

``` r
with(GBSG2, table(menostat, tgrade, cens))
```

    ## , , cens = 0
    ## 
    ##         tgrade
    ## menostat   I  II III
    ##     Post  36 137  43
    ##     Pre   27 105  39
    ## 
    ## , , cens = 1
    ## 
    ##         tgrade
    ## menostat   I  II III
    ##     Post  12 124  44
    ##     Pre    6  78  35

We see that we have event information on every strata, so we are good to
go!

``` r
cox1 <- coxph(
  surv_obj ~ age + tsize + pnodes + log_progrec + log_estrec + horTh +
    strata(menostat, tgrade),
  data = GBSG2,
  x = TRUE, y = TRUE
)

summary(cox1)
```

    ## Call:
    ## coxph(formula = surv_obj ~ age + tsize + pnodes + log_progrec + 
    ##     log_estrec + horTh + strata(menostat, tgrade), data = GBSG2, 
    ##     x = TRUE, y = TRUE)
    ## 
    ##   n= 686, number of events= 299 
    ## 
    ##                  coef exp(coef)  se(coef)      z Pr(>|z|)    
    ## age         -0.008738  0.991300  0.009209 -0.949  0.34271    
    ## tsize        0.006524  1.006545  0.004007  1.628  0.10347    
    ## pnodes       0.050624  1.051927  0.007845  6.453 1.09e-10 ***
    ## log_progrec -0.192398  0.824979  0.041113 -4.680 2.87e-06 ***
    ## log_estrec   0.025453  1.025780  0.042509  0.599  0.54933    
    ## horThyes    -0.365690  0.693718  0.130020 -2.813  0.00491 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##             exp(coef) exp(-coef) lower .95 upper .95
    ## age            0.9913     1.0088    0.9736    1.0094
    ## tsize          1.0065     0.9935    0.9987    1.0145
    ## pnodes         1.0519     0.9506    1.0359    1.0682
    ## log_progrec    0.8250     1.2122    0.7611    0.8942
    ## log_estrec     1.0258     0.9749    0.9438    1.1149
    ## horThyes       0.6937     1.4415    0.5377    0.8951
    ## 
    ## Concordance= 0.679  (se = 0.017 )
    ## Likelihood ratio test= 87.07  on 6 df,   p=<2e-16
    ## Wald test            = 101.4  on 6 df,   p=<2e-16
    ## Score (logrank) test = 106  on 6 df,   p=<2e-16

``` r
zph_strat <- cox.zph(cox1, transform = "km")
zph_strat
```

    ##               chisq df     p
    ## age         4.16194  1 0.041
    ## tsize       0.00294  1 0.957
    ## pnodes      0.71959  1 0.396
    ## log_progrec 2.84419  1 0.092
    ## log_estrec  5.47562  1 0.019
    ## horTh       0.13748  1 0.711
    ## GLOBAL      9.12561  6 0.167

``` r
plot(zph_strat, var = 1)
abline(h = 0, col = 2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
plot(zph_strat, var = 4)
abline(h = 0, col = 2)
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-9-2.png)<!-- -->

It seems that we have fixed a few PH violations, but we still have a
couple of variables pending. In this case, we will create a
s_tep-function for these variables. We will split the timespanm of the
study in 3: from th start until 2.5 years, 2.5-5 years and 5+ years. For
`age` and `log_progrec` variables, we will have 3 difefrent HRs
depending on the timespan.

``` r
# Define data cuts
cuts <- c(0, 365*2.5, 365*5, Inf)

gb <- survSplit(GBSG2, cut = cuts[-c(1, length(cuts))],
                end = "time", start = "tstart", event = "cens",
                episode = "split")


cox2 <- coxph(
  Surv(tstart, time, cens) ~ tsize + pnodes + horTh + log_estrec +
    strata(menostat, tgrade) +
    age:strata(split) + log_progrec:strata(split),
  data = gb, x = TRUE, y = TRUE
)

cox.zph(cox2) 
```

    ##                             chisq df     p
    ## tsize                      0.0298  1 0.863
    ## pnodes                     0.6664  1 0.414
    ## horTh                      0.0570  1 0.811
    ## log_estrec                 3.6195  1 0.057
    ## age:strata(split)         10.8016  3 0.013
    ## strata(split):log_progrec  1.5637  3 0.668
    ## GLOBAL                    15.1529 10 0.127

Now, we already have a model that seems to be aligned with the PH
assumption. We can check if we have any influential observation:

``` r
# --- Influential observations ---------------------------------------------
ggcoxdiagnostics(cox2, type = "dfbeta", linear.predictions = FALSE)
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Although we can observe a few observations, it seems like they do not
have a big impact on the model.

Now that we have already built the model, let’s take a look at the
results!

``` r
summary(cox2) 
```

    ## Call:
    ## coxph(formula = Surv(tstart, time, cens) ~ tsize + pnodes + horTh + 
    ##     log_estrec + strata(menostat, tgrade) + age:strata(split) + 
    ##     log_progrec:strata(split), data = gb, x = TRUE, y = TRUE)
    ## 
    ##   n= 1192, number of events= 299 
    ## 
    ##                                       coef exp(coef)  se(coef)      z Pr(>|z|)
    ## tsize                             0.006679  1.006702  0.004011  1.665  0.09587
    ## pnodes                            0.050678  1.051984  0.007910  6.407 1.48e-10
    ## horThyes                         -0.354093  0.701810  0.130015 -2.723  0.00646
    ## log_estrec                        0.023985  1.024275  0.042519  0.564  0.57268
    ## age:strata(split)split=1         -0.010354  0.989699  0.011108 -0.932  0.35126
    ## age:strata(split)split=2         -0.014772  0.985337  0.017100 -0.864  0.38766
    ## age:strata(split)split=3          0.084765  1.088462  0.061307  1.383  0.16677
    ## strata(split)split=1:log_progrec -0.219381  0.803015  0.046449 -4.723 2.32e-06
    ## strata(split)split=2:log_progrec -0.118276  0.888451  0.067157 -1.761  0.07821
    ## strata(split)split=3:log_progrec -0.339125  0.712393  0.210128 -1.614  0.10655
    ##                                     
    ## tsize                            .  
    ## pnodes                           ***
    ## horThyes                         ** 
    ## log_estrec                          
    ## age:strata(split)split=1            
    ## age:strata(split)split=2            
    ## age:strata(split)split=3            
    ## strata(split)split=1:log_progrec ***
    ## strata(split)split=2:log_progrec .  
    ## strata(split)split=3:log_progrec    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                                  exp(coef) exp(-coef) lower .95 upper .95
    ## tsize                               1.0067     0.9933    0.9988    1.0146
    ## pnodes                              1.0520     0.9506    1.0358    1.0684
    ## horThyes                            0.7018     1.4249    0.5439    0.9055
    ## log_estrec                          1.0243     0.9763    0.9424    1.1133
    ## age:strata(split)split=1            0.9897     1.0104    0.9684    1.0115
    ## age:strata(split)split=2            0.9853     1.0149    0.9529    1.0189
    ## age:strata(split)split=3            1.0885     0.9187    0.9652    1.2274
    ## strata(split)split=1:log_progrec    0.8030     1.2453    0.7331    0.8796
    ## strata(split)split=2:log_progrec    0.8885     1.1256    0.7789    1.0134
    ## strata(split)split=3:log_progrec    0.7124     1.4037    0.4719    1.0754
    ## 
    ## Concordance= 0.68  (se = 0.017 )
    ## Likelihood ratio test= 91.54  on 10 df,   p=3e-15
    ## Wald test            = 104.7  on 10 df,   p=<2e-16
    ## Score (logrank) test = 109.8  on 10 df,   p=<2e-16

Interestingly, it seems like we have a few significant variables, that
are indeed influencing the survival time of the patients, such as the
number of nodes, whether or not they have received hormonal thearapy. It
seems like, in the first 2.5 years of the study,the levels of PR
receptor seem to be relevant for patient survival.

Let’s try to plot these values, so it will be easier to understand what
then model is inferring:

``` r
ggforest(cox2, data = model.frame(cox2))
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

As commented earlier, we see that receiving hormone therapy decreses the
risk of death, with a HR of 0.7, compared to not receiving this thearpy.
This completely makes sense. If you receive hormone therapy, it is more
likely you survive, although thisd has to be dependent on PR/ER levels.
It might be interesting to create a different study for ER-/PR- patients
and the rest!

Also, for the number of nodes, we can see a HR of 1.05. This means that,
as the number of positive nodes increases, so does the hazard of
faiolure (death). It might be interesting of assessing whether or not
this is clinically significant!

Moreover, since we stratified `menostat` and `tgrade`, the model is
unable to report these HRs. In case we are interested in checking these
HRs, it might be interesting to try a time-dependent variable in the
model, and assess the evoilution onf the HR over time. We also see on
these plots the HRs for age and log_progrec. This function does not
allow plottinh time dependent HRs, so we have to do it manually.

``` r
# Does not display the change in HRs over time!
tt <- tidy(cox2, conf.int = TRUE, exponentiate = TRUE)
# age per split "age:strata(split)split=1"
age_df <- tt %>%
  filter(str_detect(term, "^age:strata\\(split\\)split=")) %>%
  mutate(
    split = as.integer(str_replace(term, "^age:strata\\(split\\)split=", "")),
    var   = "age"
  ) %>%
  select(var, split, estimate, conf.low, conf.high, p.value)

# log_progrec per split
pr_df <- tt %>%
  filter(str_detect(term, "log_progrec"),
         str_detect(term, "strata\\(split\\)split=")) %>%
  mutate(
    split = as.integer(str_extract(term, "(?<=split=)\\d+")),
    var   = "log_progrec"
  ) %>%
  select(var, split, estimate, conf.low, conf.high, p.value)

# Tags for intervals
bounds <- gb %>%
  group_by(split) %>%
  summarise(
    t_start = min(tstart, na.rm = TRUE),
    t_end   = max(time,   na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(split) %>%
  mutate(
    int_lab = paste0("[", t_start, ", ", t_end, "] días")
  )

age_df  <- age_df  %>% left_join(bounds, by = "split")
pr_df   <- pr_df   %>% left_join(bounds, by = "split")

age_df$int_lab <- fct_inorder(age_df$int_lab)
pr_df$int_lab  <- fct_inorder(pr_df$int_lab)

# Plot

g_age_step <- ggplot(age_df, aes(x = t_start, xend = t_end,
                                 y = estimate, yend = estimate)) +
  geom_hline(yintercept = 1, linetype = 2) +
  geom_segment(linewidth = 1) +
  geom_rect(aes(xmin = t_start, xmax = t_end,
                ymin = conf.low, ymax = conf.high),
            alpha = 0.15) +
  labs(x = "Time (days)", y = "HR",
       title = "Age: HR(t) - step function") +
  theme_minimal()

g_pr_step <- ggplot(pr_df, aes(x = t_start, xend = t_end,
                               y = estimate, yend = estimate)) +
  geom_hline(yintercept = 1, linetype = 2) +
  geom_segment(linewidth = 1) +
  geom_rect(aes(xmin = t_start, xmax = t_end,
                ymin = conf.low, ymax = conf.high),
            alpha = 0.15) +
  labs(x = "Time (days)", y = "HR",
       title = "log_progrec: HR(t) - step function") +
  theme_minimal()

g_age_step
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

``` r
g_pr_step
```

![](Survival-Analysis_files/figure-gfm/unnamed-chunk-14-2.png)<!-- -->

We can see that, on early days of the study, the HR for log_progrec is
0.8, meaning that high levels of the receptor correwlate with a better
prognosis. However, this does not seem to be true after 2.5 years. In
terms of age, it does not seem to be significant at all. For future
models, it might be interesting to model age as a Predictable
time-dependent covariates (check
<https://cran.r-project.org/web/packages/survival/vignettes/timedep.pdf>)
