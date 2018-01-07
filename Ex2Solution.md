Exercise Set 2 Solution
================
Tjeerd Dijkstra
2018-01-07

Introduction
============

Examplary solution of excersises set 2 for Tuebingen neuroschool Machine Learning 1 Class. Analysis of data from the "EEG brainwave for confusion" Kaggle competition at <https://www.kaggle.com/wanghaohan/eeg-brain-wave-for-confusion> This solution shows a couple of features that are helpful: 1. it uses an R notebook, webinar at <https://www.rstudio.com/resources/webinars/introducing-notebooks-with-r-markdown/> 2. it uses git and github for version control and backup. See <http://happygitwithr.com>

``` r
library(readr); library(dplyr, warn.conflicts = FALSE); library(tidyr); library(tibble); library(ggplot2)
```

1 Load and clean data
=====================

E1.1&E1.2
---------

I prefer to use read\_csv() from the readr package as it gives more control than the default R read.csv(). I also like to set the colum types myself as this faster and safer. Lastly, as a minor detail it is a good idea to name the code chunks after the main object that is created in the chunk, here data.frame (tibble) Y.

``` r
Y <- read_csv("EEGData.csv", col_types = "ddddddddddddddd",
              col_names = c("SubjectID", "VideoID", "Attention", "Mediation",
                            "Raw", "Delta", "Theta", "Alpha1", "Alpha2", "Beta1", "Beta2",
                            "Gamma1", "Gamma2", "ExpectedConfusion", "ReportedConfusion"))
```

E1.4
----

For some reason the ID variables are coded as floating point. Change the ID variables to integer. I use package dplyr with the pipe notation. Using dplyr for data wrangling avoids almost all loops.

``` r
Y <- Y %>% mutate(SubjectID = as.integer(SubjectID), VideoID = as.integer(VideoID))
```

E1.3
----

calculate mean of predictors per subject and video.

``` r
Ymean <- Y %>% group_by(SubjectID, VideoID) %>% summarise_all(mean)
```

optional
--------

This adds reported confusion as meaningfully encoded factor with levels "confused" and "not confused". In general you should encode your factors with descriptive levels. Almost all packages in R can work with these (in particular caret and ggplot2) but threre are still "old school" exceptions (for example glmnet for L1-penalized generalized linear models). As a last note, you should also explicitly prescribe the order of the levels. Withe two levels this is rarely relevant but when you have more there is usually a base or reference level that should come first.

``` r
Ymean <- Ymean %>% mutate(RepConfFactor = factor(if_else(ReportedConfusion == 1, "confused", "not confused"),
                                         c("confused", "not confused")))
```

2 Outlier analysis
==================

Using ggpairs() to look at all the data.

``` r
library(GGally, warn.conflicts = FALSE); ggp.out <- ggpairs(select(ungroup(Ymean), Attention:Gamma2)); print(ggp.out, progress = FALSE)
```

![](Ex2Solution_files/figure-markdown_github/ggpairs-1.png)

Using ggplot to look at Attention and Mediation data only

``` r
print(ggplot(Ymean, aes(x = VideoID, y = Attention)) + geom_point() +
        facet_wrap( ~ SubjectID, nrow = 2) + labs(title = "Attention"))
```

![](Ex2Solution_files/figure-markdown_github/OutlierPlotAttentionMediationSeparate-1.png)

``` r
print(ggplot(Ymean, aes(x = VideoID, y = Mediation)) + geom_point() +
        facet_wrap( ~ SubjectID, nrow = 2) + labs(title = "Mediation"))
```

![](Ex2Solution_files/figure-markdown_github/OutlierPlotAttentionMediationSeparate-2.png)

E2
--

Visual way to show that Attention and Mediation have the same outliers.

``` r
Y.plot <- gather(Ymean, AttMed, AttMedValue, c(Attention, Mediation))
print(ggplot(Y.plot, aes(x = VideoID, y = AttMedValue, group = AttMed, color = AttMed)) +
        geom_point() + facet_wrap( ~ SubjectID, nrow = 2) +
        labs(title = "Outlier Analysis of Attention and Mediation"))
```

![](Ex2Solution_files/figure-markdown_github/OutlierPlotAttentionMediationCombined-1.png)
