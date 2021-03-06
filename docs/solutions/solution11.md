Lab 11: Multitype summary functions and models
================

This session is concerned with summary statistics and Gibbs models for multitype point patterns.
The lecturer's R script is [available here](https://raw.githubusercontent.com/spatstat/SSAI2017/master/Scripts/script11.R) (right click and save).

``` r
library(spatstat)
```

### Exercise 1

The `amacrine` dataset contains the locations of cells of two types (“on” and “off” detectors) in a layer of the retina.

1.  Compute and plot the bivariate *L* function for the amacrine data.

    ``` r
    Lam <- alltypes(amacrine, "L")
    plot(Lam)
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-3-1.png)

2.  plot estimates of the bivariate pair correlation functions by

    ``` r
    plot(alltypes(amacrine, pcfcross))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-4-1.png)

3.  What is the overall interpretation of these summary functions?

    Inhibition for types of same type. Independence between types.

### Exercise 2

Continuing with the `amacrine` data,

1.  Use `alltypes` to plot the bivariate *G*-functions *G*<sub>*i**j*</sub> for each pair of types *i*, *j* in the amacrine data.

    ``` r
    plot(alltypes(amacrine, Gcross))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-5-1.png)

2.  Use `alltypes` to plot the functions *G*<sub>*i*•</sub> (`Gdot` in `spatstat`) for each type *i* in the amacrine data.

    ``` r
    plot(alltypes(amacrine, Gdot))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-6-1.png)

3.  What is the overall interpretation of the *G*-functions?

    Same as before.

### Exercise 3

The dataset `bramblecanes` gives the locations and ages of bramble cane plants in a study region. Age is a categorical variable, with three levels. We will conduct a randomisation test of the Random Labelling Property.

1.  Read the help for the command `rlabel`.

2.  We will use the bivariate *K*-function *K*<sub>2, 0</sub> as our summary statistic. Compute this for the data using `Kcross(bramblecanes, "2", "0")` and plot it.

    ``` r
    plot(Kcross(bramblecanes, "2", "0"))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-7-1.png)

3.  Read the help for `Kcross`. Find the names of the second and third arguments to the function.

    The names are `i` and `j`. Alternatively the arguments `from` and `to` can be used for the same purpose.

4.  Generate the simulation envelopes as follows

    ``` r
    shuffle <- expression(rlabel(bramblecanes))
    E <- envelope(bramblecanes, Kcross, nsim=19, simulate=shuffle, i="2", j="0")
    ```

        ## Generating 19 simulations by evaluating expression  ...
        ## 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18,  19.
        ## 
        ## Done.

    ``` r
    plot(E)
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-8-1.png)

    Note that the named arguments `i` and `j` are not recognised by the `envelope` command (as we can check from the help file for `envelope`), so they are passed to the command `Kcross` as we intended.

5.  Generate the corresponding simulation envelopes of the bivariate *L*-function, either by replacing `Kcross` by `Lcross` in the code above, or by

    ``` r
    plot(E, sqrt(./pi) ~ r)
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-9-1.png)

### Exercise 4

We want to fit a Gibbs process model to the `betacells` data.

1.  Access the `betacells` data and plot the pattern.

    ``` r
    plot(betacells, main = "Beta cells")
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-10-1.png)

2.  Save the data as a point pattern `X` and save only the mark `type`

    ``` r
    X <- betacells
    marks(X) <- marks(betacells)$type
    ```

    Also we will save the two type names:

    ``` r
    typ <- levels(marks(X))
    ```

3.  Plot the bivariate *K* functions.

    1.  Does it appear that cells of the same type interact? If so, guess at a suitable interaction distance.
    2.  Does it appear that cells of different types interact? If so, guess at a suitable interaction distance.

    ``` r
    plot(alltypes(X, Kcross))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-13-1.png)

    Yes, points of same type appear to be interacting at e.g. 60 microns.

    No, points of opposite types do not appear to interact (or if they do it is only at quite short distances).

4.  Fit a multitype Strauss model using the selected interactions. For example if your answer to question *i* was “yes, at 20 microns” and your answer to question *ii* was “yes, at 30 microns”,

    ``` r
    rad <- matrix(c(20,30,30,20), 2, 2)
    ppm(X ~ marks, MultiStrauss(typ,rad))
    ```

    while if your answer to question *i* was “no” and your answer to question *ii* was “yes, at 60 microns”,

    ``` r
    rad <- matrix(c(NA,60,60,NA), 2, 2)
    ppm(X ~ marks, MultiStrauss(typ,rad))
    ```

    ``` r
    rad <- matrix(c(60,NA,NA,60), 2, 2)
    fit <- ppm(X ~ marks, MultiStrauss(typ,rad))
    ```

    Interpret the fitted model. Plot the array of fitted pairwise interactions using `plot(fitin(fit))` where `fit` is the fitted model. What is the fitted strength of the interaction?

    ``` r
    plot(fitin(fit))
    ```

    ![](solution11_files/figure-markdown_github/unnamed-chunk-17-1.png)

    The fitted interaction is very strong for same type cells (and absent for opposite types).

5.  For comparison purposes, fit the following models, interpret them, and compare the results:

    ``` r
    fitU <- ppm(X ~ marks, Strauss(60))
    rad <- matrix(60, 2, 2)
    fitE <- ppm(X ~ marks, MultiStrauss(rad))
    ```

### Exercise 5

Here we will use profile pseudolikelihood to estimate the interaction distances for the multitype Strauss model in Question 4. We’ll assume that points of different types do not interact, and that points of the same type interact at a distance *R* which is the same for each type.

1.  Create a vector of values of *R* to search over:

    ``` r
    rval <- data.frame(R=seq(50,100,by=5))
    ```

    This will become the argument `s` of `profilepl`.

2.  We need the argument `f` of `profilepl`, and this should be a function that takes the value *R* and produces a multitype Strauss interaction. So define

    ``` r
    MS <- function(R){ MultiStrauss(diag(c(R,R))) }
    ```

    Try typing `MS(50)` to check that this is what you expect.

    ``` r
    MS(50)
    ```

        ## Pairwise interaction family
        ## Interaction:Multitype Strauss process
        ## 2 types of points
        ## Possible types:   not yet determined
        ## Interaction radii:
        ##      [,1] [,2]
        ## [1,]   50   NA
        ## [2,]   NA   50

3.  Then we can use maximum profile pseudolikelihood:

    ``` r
    profilepl(rval, MS, X ~ marks)
    ```

        ## (computing rbord)

        ## comparing 11 models...

        ## 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,  11.

        ## fitting optimal model...

        ## done.

        ## profile log pseudolikelihood
        ## for model:  ppm(X ~ marks,  interaction = MS)
        ## fitted with rbord = 100
        ## interaction: Multitype Strauss process
        ## irregular parameter: R in [50, 100]
        ## optimum value of irregular parameter:  R = 80
