<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Build Status](https://travis-ci.org/paul-buerkner/brms.svg?branch=master)](https://travis-ci.org/paul-buerkner/brms) [![Coverage Status](https://codecov.io/github/paul-buerkner/brms/coverage.svg?branch=master)](https://codecov.io/github/paul-buerkner/brms?branch=master) [![CRAN Version](http://www.r-pkg.org/badges/version/brms)](http://cran.r-project.org/package=brms)

brms
====

The <b>brms</b> package provides an interface to fit Bayesian generalized (non-)linear mixed models using Stan, which is a C++ package for obtaining Bayesian inference using the No-U-turn sampler (see <http://mc-stan.org/>). The formula syntax is very similar to that of the package lme4 to provide a familiar and simple interface for performing regression analyses.

<!--

-->
How to use brms
===============

``` r
library(brms)
```

As a simple example, we use poisson regression to model the seizure counts in epileptic patients to investigate whether the treatment (represented by variable Trt\_c) can reduce the seizure counts. Three random intercepts are incorporated to account for the variance between patients and visits, as well as for the residual variance.

``` r
fit <- brm(count ~ log_Age_c + log_Base4_c * Trt_c + (1|patient) + (1|visit) + (1|obs), 
           data = epilepsy, family = "poisson")
#> Compiling the C++ model
```

The results (i.e. posterior samples) can be investigated using

``` r
summary(fit, waic = TRUE) 
#>  Family: poisson (log) 
#> Formula: count ~ log_Age_c + log_Base4_c * Trt_c + (1 | patient) + (1 | visit) + (1 | obs) 
#>    Data: epilepsy (Number of observations: 236) 
#> Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1; 
#>          total post-warmup samples = 4000
#>    WAIC: 1145.78
#>  
#> Group-Level Effects: 
#> ~obs (Number of levels: 236) 
#>               Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
#> sd(Intercept)     0.37      0.04     0.29     0.46       1279    1
#> 
#> ~patient (Number of levels: 59) 
#>               Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
#> sd(Intercept)     0.51      0.07     0.38     0.66        979    1
#> 
#> ~visit (Number of levels: 4) 
#>               Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
#> sd(Intercept)     0.11      0.11        0     0.42        769    1
#> 
#> Population-Level Effects: 
#>                   Estimate Est.Error l-95% CI u-95% CI Eff.Sample Rhat
#> Intercept             1.56      0.10     1.35     1.78        851    1
#> log_Age_c             0.46      0.36    -0.25     1.18        953    1
#> log_Base4_c           1.07      0.11     0.85     1.29        956    1
#> Trt_c                -0.33      0.16    -0.66    -0.01        848    1
#> log_Base4_c:Trt_c     0.36      0.21    -0.06     0.78       1080    1
#> 
#> Samples were drawn using sampling(NUTS). For each parameter, Eff.Sample 
#> is a crude measure of effective sample size, and Rhat is the potential 
#> scale reduction factor on split chains (at convergence, Rhat = 1).
```

On the top of the output, some general information on the model is given, such as family, formula, number of iterations and chains, as well as the WAIC, which is an information criterion for Bayesian models. Next, random effects are displayed seperately for each grouping factor in terms of standard deviations and (in case of more than one random effect per grouping factor; not displayed here) correlations between random effects. On the bottom of the output, fixed effects are displayed. If incorporated, autocorrelation effects and family specific parameters (e.g., the residual standard deviation 'sigma' in normal models) are also given.

In general, every parameter is summarized using the mean ('Estimate') and the standard deviation ('Est.Error') of the posterior distribution as well as two-sided 95% Credible intervals ('l-95% CI' and 'u-95% CI') based on quantiles. The last two values ('Eff.Sample' and 'Rhat') provide information on how well the algorithm could estimate the posterior distribution of this parameter. If 'Rhat' is considerably greater than 1, the algorithm has not yet converged and it is necessary to run more iterations and / or set stronger priors.

To visually investigate the chains as well as the posterior distributions, you can use

``` r
plot(fit) 
```

An even more detailed investigation can be achieved by applying the shinystan package:

``` r
launch_shiny(fit) 
```

There are several methods to compute and visualize model predictions. Suppose that we want to predict responses (i.e. seizure counts) of a person in the treatment group (`Trt_c = 0.5`) and in the control group (`Trt_c = -0.5`) with average age and average number of previous seizures at the fourth visit. Than we can use

``` r
newdata <- data.frame(Trt_c = c(0.5, -0.5), log_Age_c = 0, 
                      log_Base4_c = 0, visit = 4)
predict(fit, newdata = newdata, allow_new_levels = TRUE, probs = c(0.05, 0.95))
#>   Estimate Est.Error 5%ile 95%ile
#> 1  4.71100  3.861503     0     12
#> 2  6.54625  5.355327     1     16
```

We need to set `allow_new_levels = TRUE` because we want to predict responses of a person that was not present in the data used to fit the model. While the `predict` method returns predictions of the responses, the `fitted` method returns predictions of the regression line.

``` r
fitted(fit, newdata = newdata, allow_new_levels = TRUE, probs = c(0.05, 0.95))
#>   Estimate Est.Error    5%ile   95%ile
#> 1 4.736238  3.273596 1.347698 10.97859
#> 2 6.603166  4.560335 1.868479 15.71147
```

Both methods return the same etimate (up to random error), while the latter has smaller variance, because the uncertainty in the regression line is smaller than the uncertainty in each response. If we want to predict values of the original data, we can just leave the `newdata` argument empty.

A related feature is the computation and visualization of marginal effects, which can help in better understanding the influence of the predictors on the response.

``` r
plot(marginal_effects(fit, probs = c(0.05, 0.95)))
```

For a complete list of methods to apply on <b>brms</b> models see

``` r
methods(class = "brmsfit") 
#>  [1] as.mcmc           coef              expose_functions  family           
#>  [5] fitted            fixef             formula           hypothesis       
#>  [9] launch_shiny      logLik            LOO               marginal_effects 
#> [13] model.frame       ngrps             nobs              pairs            
#> [17] parnames          plot              posterior_samples predict          
#> [21] print             prior_samples     ranef             residuals        
#> [25] stancode          standata          stanplot          summary          
#> [29] update            VarCorr           vcov              WAIC             
#> see '?methods' for accessing help and source code
```

Details on formula syntax, families and link functions, as well as prior distributions can be found on the help page of the brm function:

``` r
help(brm) 
```

More instructions on how to use <b>brms</b> are given in the package's vignette.

``` r
vignette("brms") 
```

FAQ
===

How do I install brms?
----------------------

To install the latest release version from CRAN use

``` r
install.packages("brms")
```

The current developmental version can be downloaded from github via

``` r
library(devtools)
install_github("paul-buerkner/brms")
```

Because <b>brms</b> is based on Stan, a C++ compiler is required. The program Rtools (available on <https://cran.r-project.org/bin/windows/Rtools/>) comes with a C++ compiler for Windows. On Mac, you should install Xcode. For further instructions on how to get the compilers running, see the prerequisites section on <https://github.com/stan-dev/rstan/wiki/RStan-Getting-Started>.

Can I avoid compiling models?
-----------------------------

When you fit your model for the first time with <b>brms</b>, there is currently no way to avoid compilation. However, if you have already fitted your model and want to run it again, for instance with more samples, you can do this without recompilation by using the `update` method (type `help(update.brmsfit)` in R for more details).

What is the difference between brms and rstanarm?
-------------------------------------------------

<b>rstanarm</b> is an R package similar to <b>brms</b> that also allows to fit regression models using <b>Stan</b> for the backend estimation. Contrary to <b>brms</b>, <b>rstanarm</b> comes with precompiled code to save the compilation time (and the need for a C++ compiler) when fitting a model. However, as <b>brms</b> generates its <b>Stan</b> code on the fly, it offers more flexibility in model specification than <b>rstanarm</b>. Also, multilevel models are currently fitted a bit more efficiently in <b>brms</b>. For a detailed comparison of <b>brms</b> with other common R packages implementing multilevel models, type `vignette("brms")` in R.

What is the best way to ask a question or propose a new feature?
----------------------------------------------------------------

You can either open an issue on [github](https://github.com/paul-buerkner/brms) or write me an email (<paul.buerkner@gmail.com>).
