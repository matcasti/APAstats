
# writR: Inferential statistics and reporting in APA style

For automated and basic inferential testing.

### Installation

For installation of developmental version run in your R console:

``` r
install.packages("remotes")
remotes::install_github("matcasti/writR")
```

### Citation

To cite package 'writR' in publications run the following code in your `R` console:


```r
citation('writR')
```

```
## 
## To cite package 'writR' in publications use:
## 
##   Matías Castillo Aguilar (2021). writR: Inferential statistics and
##   reporting in APA style. R package version 0.0.1.
##   https://github.com/matcasti/writR
## 
## A BibTeX entry for LaTeX users is
## 
##   @Manual{,
##     title = {writR: Inferential statistics and reporting in APA style},
##     author = {Matías {Castillo Aguilar}},
##     year = {2021},
##     note = {R package version 0.0.1},
##     url = {https://github.com/matcasti/writR},
##   }
```

## Summary of available tests

#### For paired samples designs

| Nº of groups |  Type  | Test  | Function in `R` |
|:-:|---|---|---|
| 2 | `type = 'p'`: parametric. | Student's t-test. | `stats::t.test` |
| 2 | `type = 'r'`: robust. | Yuen's test for trimmed means. | `WRS2::yuend` |
| 2 | `type = 'np'`: non-parametric. | Wilcoxon signed-rank test. | `stats::wilcox.test` |
| \> 2 | `type = 'p'`: parametric. | One-way repeated measures ANOVA (rmANOVA). | `afex::aov_ez` |
| \> 2 | `type = 'p'`: parametric. | rmANOVA with Greenhouse-Geisser correction. | `afex::aov_ez` |
| \> 2 | `type = 'p'`: parametric. | rmANOVA with Huynh-Feldt correction. | `afex::aov_ez` |
| \> 2 | `type = 'r'`: robust. | Heteroscedastic rmANOVA for trimmed means. | `WRS2::rmanova` |
| \> 2 | `type = 'np'`: non-parametric. | Friedman rank sum test. | `stats::friedman.test` |

#### For independent samples design

| Nº of groups | Type | Test | Function in `R` |
|:-:|---|---|---|
| 2 | `type = 'p'`: parametric. | Student's t-test. | `stats::t.test` |
| 2 | `type = 'p'`: parametric. | Welch's t-test. | `stats::t.test` |
| 2 | `type = 'r'`: robust. | Yuen's test for trimmed means. | `WRS2::yuen` |
| 2 | `type = 'np'`: non-parametric. | Mann-Whitney *U* test. | `stats::wilcox.test` |
| \> 2 | `type = 'p'`: parametric. | Fisher's One-way ANOVA. | `stats::oneway.test` |
| \> 2 | `type = 'p'`: parametric. | Welch's One-way ANOVA. | `stats::oneway.test` |
| \> 2 | `type = 'p'`: parametric. | Kruskal-Wallis one-way ANOVA. | `stats::kruskal.test` |
| \> 2 | `type = 'r'`: robust. | Heteroscedastic one-way ANOVA for trimmed means. | `WRS2::t1way` |

#### Corresponding Post-Hoc tests for Nº groups \> 2

| Design | Type | Test | Function in `R` |
|:-:|---|---|---|
| Paired    | `type = 'p'`: parametric. | Pairwise Student's t-test. | `stats::pairwise.t.test`  |
| Paired    | `type = 'np'`: non-parametric. | Conover-Iman all-pairs comparison test for a balanced incomplete block design. | `PMCMRplus::durbinAllPairsTest` |
| Paired    | `type = 'r'`: robust. | Yuen's test for trimmed means and Rom's method for controlling FWE. (see [Wilcox, 2012](http://mqala.co.za/veed/Introduction%20to%20Robust%20Estimation%20and%20Hypothesis%20Testing.pdf), p. 385). | `WRS2::rmmcp` |
| Independent | `type = 'p'`: parametric + `var.equal = TRUE`. | Pairwise Student's t-test. | `stats::pairwise.t.test` |
| Independent | `type = 'p'`: parametric + `var.equal = FALSE`. | Games-Howell all-pairs comparison test. | `PMCMRplus::gamesHowellTest` |
| Independent | `type = 'np'`: non-parametric. | Dunn's non-parametric all-pairs comparison test. | `PMCMRplus::kwAllPairsDunnTest` |
| Independent | `type = 'r'`: robust. | Pairwise trimmed mean differences (see [Mair and Wilcox](https://rdrr.io/rforge/WRS2/f/inst/doc/WRS2.pdf)). | `WRS2::lincon` |


# Automated testing

By default, `report` function, checks automatically the assumptions of the test based on the parameters of the data entered


```r
library(writR) # Load the writR package

set.seed(123)
Diets <- data.frame(
    weight = c(rnorm(n = 100/2, mean = 70, sd = 7)   # Treatment
             , rnorm(n = 100/2, mean = 66, sd = 7) ) # Control
  , diet = gl(n = 2, k = 100/2, labels = c('Treatment', 'Control') ) )
  
result <- report(
  data = Diets,
  variable = weight, # dependent variable
  by = diet, # independent variable
  type = 'auto', # default
  pairwise.comp = TRUE, # for pairwise comparisons in case of n groups > 2
  markdown = TRUE # output in Markdown format (for inline text)? otherwise plain text.
)

result
```

```
## $report
## [1] "*t* ~Student~ (98) = 2.509, *p* = 0.014, *Cohen's d* = 0.51, IC~95%~[0.1, 0.91]"
## 
## $method
## [1] "Student's t-test for independent samples"
```

## Inline results in APA style

The core function: `report` by default return a list of length two in Markdown format (as seen before) for inline results. An example using same data as before:

The analysis of the effects of the treatment shows an statistically significant difference between the groups, `result$report`, evaluated through `result$method`.

translates into this:

The analysis of the effects of the treatment shows an statistically significant difference between the groups, *t* <sub>Student</sub> (98) = 2.509, *p* = 0.014, *Cohen's d* = 0.51, IC<sub>95%</sub> [0.1, 0.91], evaluated through Student's t-test for independent samples.

## Paired samples design

For paired designs you need to set `paired = TRUE`, and then, based on the numbers of groups detected after removing missing values, the test will run depending on the parameters stablished.

#### \> 2 groups

When `type = 'auto'` the next assumptions will be checked for \> 2 paired samples:

| Assumption checked | How is tested | If met | If not |
|----------|------------------------|------------------------|------------------------|
| Normality | `stats::shapiro.test` for n \< 50 or `nortest::lillie.test` for n \>= 50 | Sphericity check. | Friedman rank sum test |
| Sphericity | `afex::test_sphericity(model)`  | One-way repeated measures ANOVA (rmANOVA) | Greenhouse-Geisser or Huynh-Feldt correction is applied |


```r
set.seed(123)
Cancer <- data.frame(
    cells = round(c(rnorm(n = 150/3, mean = 100, sd = 10)   # Basal
           , rnorm(n = 150/3, mean = 98, sd = 10)   # Time-1
           , rnorm(n = 150/3, mean = 98, sd = 10) )) # Time-2
  , period = gl(n = 3, k = 150/3, labels = c('Basal', 'Time-1', 'Time-2') ) )

report(
  data = Cancer
  , variable = cells
  , by = period
  , paired = TRUE
  , markdown = FALSE
  )
```

```
## $report
## [1] "F(2, 98) = 3.766, p = 0.027, eta^ = 0.071, IC95% [0, 0.18]"
## 
## $method
## [1] "One-way repeated measures ANOVA"
```

However, you can specify your own parameters for the selection of the test:

| Test | Parameters |
|-------------------------------|-------------------------------|
| One-way repeated measures ANOVA (rmANOVA)  | `paired = TRUE` + `type = 'p'` + `sphericity = TRUE` |
| rmANOVA with Greenhouse-Geisser correction | `paired = TRUE` + `type = 'p'` + `sphericity = 'GG'` |
| rmANOVA with Huynh-Feldt correction        | `paired = TRUE` + `type = 'p'` + `sphericity = 'HF'` |
| Heteroscedastic rmANOVA for trimmed means  | `paired = TRUE` + `type = 'r'`                       |
| Friedman rank sum test                     | `paired = TRUE` + `type = 'np'`                      |

#### 2 groups

Similar as before, if `type = 'auto'` assumptions will be checked for 2 paired samples:

| Assumption checked | How is tested | If met | If not |
|----------|------------------------|------------------------|------------------------|
| Normality          | `stats::shapiro.test` for n \< 50 or `nortest::lillie.test` for n \>= 50 | Student's t-test | Wilcoxon signed-rank test |


```r
CancerTwo <- Cancer[Cancer$period %in% c('Time-1','Time-2'),]
  
report(
  data = CancerTwo
  , variable = cells
  , by = period
  , paired = TRUE
  , markdown = FALSE
  )
```

```
## $report
## [1] "t(49) = 2.015, p = 0.049, d = 0.29, IC95% [0, 0.57]"
## 
## $method
## [1] "Student's t-test for dependent samples"
```

Same as above, you can specify your own parameters for the selection of the test:

| Test | Parameters |
|-------------------------------|-------------------------------|
| Student's t-test for paired samples                | `paired = TRUE` + `type = 'p'`  |
| Wilcoxon signed-rank test                          | `paired = TRUE` + `type = 'np'` |
| Yuen's test on trimmed means for dependent samples | `paired = TRUE` + `type = 'r'`  |

## Independent samples design

For independent samples you need to set `paired = FALSE`, and then, based on the numbers of groups detected, the test will run depending on the parameters stablished.

#### \> 2 groups

When `type = 'auto'` the next assumptions will be checked for \> 2 independent samples:

| Assumption checked | How is tested | If met | If not |
|----------|------------------------|------------------------|------------------------|
| Normality                | `stats::shapiro.test` for n \< 50 or `nortest::lillie.test` for n \>= 50 | Homogeneity of variances check. | Kruskal-Wallis ANOVA |
| Homogeneity of variances | `car::leveneTest` | Fisher's ANOVA | Welch's ANOVA |


```r
set.seed(123)
Cancer <- data.frame(
    cells = round(c(rnorm(n = 150/3, mean = 100, sd = 10)   # Basal
           , rnorm(n = 150/3, mean = 95, sd = 10)   # Time-1
           , rnorm(n = 150/3, mean = 90, sd = 10) )) # Time-2
  , period = gl(n = 3, k = 150/3, labels = c('Basal', 'Time-1', 'Time-2') ) )

report(
  data = Cancer
  , variable = cells
  , by = period
  , paired = TRUE
  , markdown = FALSE
  )
```

```
## $report
## [1] "F(2, 98) = 23.734, p < 0.001, eta^ = 0.326, IC95% [0.18, 0.45]"
## 
## $method
## [1] "One-way repeated measures ANOVA"
```

However, you can specify your own parameters for the selection of the test:

| Test | Parameters |
|-------------------------------|-------------------------------|
| Fisher's One-way ANOVA                          | `paired = FALSE` + `type = 'p'` + `var.equal = TRUE`  |
| Welch's One-way ANOVA                           | `paired = FALSE` + `type = 'p'` + `var.equal = FALSE` |
| Kruskal–Wallis one-way ANOVA                    | `paired = FALSE` + `type = 'np'`                      |
| Heteroscedastic one-way ANOVA for trimmed means | `paired = FALSE` + `type = 'r'`                       |

#### 2 groups

Just like above, if `type = 'auto'` assumptions will be checked for 2 independent samples:

| Assumption checked | How is tested | If met | If not |
|----------|------------------------|------------------------|------------------------|
| Normality                | `stats::shapiro.test` for n \< 50 or `nortest::lillie.test` for n \>= 50 | Homogeneity of variances check. | Mann-Whitney *U* test |
| Homogeneity of variances | `car::leveneTest` | Student's t-test | Welch's t-test |


```r
CancerTwo <- Cancer[Cancer$period %in% c('Basal','Time-1'),]
  
report(
  data = CancerTwo
  , variable = cells
  , by = period
  , paired = TRUE
  , markdown = FALSE
  )
```

```
## $report
## [1] "t(49) = 2.065, p = 0.044, d = 0.29, IC95% [0.01, 0.58]"
## 
## $method
## [1] "Student's t-test for dependent samples"
```

You can specify your own parameters for the selection of the test as well:

| Test | Parameters |
|-------------------------------|-------------------------------|
| Student's t-test for independent samples | `paired = FALSE` + `type = 'p'` + `var.equal = TRUE`  |
| Welch's t-test for independent samples   | `paired = FALSE` + `type = 'p'` + `var.equal = FALSE` |
| Mann–Whitney *U* test                    | `paired = FALSE` + `type = 'np'`                      |
| Yuen's test on trimmed means             | `paired = FALSE` + `type = 'r'`                       |


## Dependencies

The package writR is standing on the shoulders of giants. `writR` depends on the following packages:


```r
library(deepdep)
plot_dependencies('writR', local = TRUE, depth = 3)
```

![](README_files/figure-html/unnamed-chunk-7-1.svg)<!-- -->

## Acknowledgments

I would like to thank Indrajeet Patil for being an inspiration for this package, as an extension of `statsExpressions`. Naturally this package is in its first steps, but I hope that future collaborative work can expand the potential of this package.

## Contact

For any issues or collaborations you can send me an email at:

-   matcasti\@umag.cl
