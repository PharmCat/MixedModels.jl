# Linear mixed-effects models in [Julia](http://julialang.org)

## Installation

This package requires Steve Johnson's
[NLopt](https://github.com/stevengj/NLopt.jl.git) package for
Julia. Before installing the `NLopt` package be sure to read the
installation instructions as it requires you to have installed the
`nlopt` library of C functions.

Once the `NLopt` package is installed,

```julia
Pkg.add("MixedModels")
```

will install this package.

## Fitting linear mixed-effects models

The `lmer` function is similar to the function of the same name in the
[lme4](http://cran.R-project.org/package=lme4) package for
[R](http://www.R-project.org).  The first two arguments for in the `R`
version are `formula` and `data`.  The principle method for the
`Julia` version takes these arguments.

### A model fit to the `Dyestuff` data from the `lme4` package

The simplest example of a mixed-effects model that we use in the
[lme4 package for R](https://github.com/lme4/lme4) is a model fit to
the `Dyestuff` data.

```R
> str(Dyestuff)
'data.frame':	30 obs. of  2 variables:
 $ Batch: Factor w/ 6 levels "A","B","C","D",..: 1 1 1 1 1 2 2 2 2 2 ...
 $ Yield: num  1545 1440 1440 1520 1580 ...
> (fm1 <- lmer(Yield ~ 1|Batch, Dyestuff, REML=FALSE))
Linear mixed model fit by maximum likelihood ['lmerMod']
Formula: Yield ~ 1 | Batch 
   Data: Dyestuff 

      AIC       BIC    logLik  deviance 
 333.3271  337.5307 -163.6635  327.3271 

Random effects:
 Groups   Name        Variance Std.Dev.
 Batch    (Intercept) 1388     37.26   
 Residual             2451     49.51   
Number of obs: 30, groups: Batch, 6

Fixed effects:
            Estimate Std. Error t value
(Intercept)  1527.50      17.69   86.33
```

These `Dyestuff` data are available in the `RDatasets` package for `julia`
```julia
julia> using MixedModels, RDatasets

julia> ds = dataset("lme4","Dyestuff");

julia> head(ds)
6x2 DataFrame
|-------|-------|-------|
| Row # | Batch | Yield |
| 1     | A     | 1545  |
| 2     | A     | 1440  |
| 3     | A     | 1440  |
| 4     | A     | 1520  |
| 5     | A     | 1580  |
| 6     | B     | 1540  |
```

`lmm` defaults to maximum likelihood estimation whereas `lmer` in `R`
defaults to REML estimation.

```julia
julia> fm1 = lmm(Yield ~ 1|Batch, ds)
Linear mixed model fit by maximum likelihood
 logLik: -163.663530, deviance: 327.327060

 Variance components:
       37.260474  1388.342960
       49.510070  2451.247052
 Number of obs: 30; levels of grouping factors: 6

  Fixed-effects parameters:
        Estimate Std.Error z value
[1,]      1527.5   17.6946 86.3258
```

(The formatting of the output will be improved.)

In general the model should be fit through an explicit call to the `fit`
function, which may take a second argument indicating a verbose fit.

```julia
julia> @time m = fit(lmm(:(Yield ~ 1 + (1|Batch)), ds),true);
f_1: 327.76702, [1.0]
f_2: 331.03619, [1.75]
f_3: 330.64583, [0.25]
f_4: 327.69511, [0.97619]
f_5: 327.56631, [0.928569]
f_6: 327.3826, [0.833327]
f_7: 327.35315, [0.807188]
f_8: 327.34663, [0.799688]
f_9: 327.341, [0.792188]
f_10: 327.33253, [0.777188]
f_11: 327.32733, [0.747188]
f_12: 327.32862, [0.739688]
f_13: 327.32706, [0.752777]
f_14: 327.32707, [0.753527]
f_15: 327.32706, [0.752584]
FTOL_REACHED
elapsed time: 0.053209407 seconds (1015136 bytes allocated)
```

The numeric representation of the model has type
```julia
julia> typeof(m)
LMMScalar1 (constructor with 2 methods)
```

It happens that `show`ing an object of this type causes the model to
be fit so it is okay to omit the call to `fit` in the REPL (the
interactive Read-Eval-Print-Loop).

Those familiar with the `lme4` package for `R` will see the usual
suspects.
```julia
julia> fixef(m)  # estimates of the fixed-effects parameters
1-element Array{Float64,1}:
 1527.5

julia> show(coef(m))  # another name for fixef
[1527.4999999999998]

julia> ranef(m)
1x6 Array{Float64,2}:
 -16.6283  0.369517  26.9747  -21.8015  53.5799  -42.4944

julia> ranef(m,true) # on the U scale
1x6 Array{Float64,2}:
 -22.0949  0.490998  35.8428  -28.9689  71.1947  -56.4647

julia> deviance(m)
327.3270598812219
```

## A more substantial example

Fitting a model to the `Dyestuff` data is trivial.  The `InstEval`
data in the `lme4` package is more of a challenge in that there are
nearly 75,000 evaluations by 2972 students on a total of 1128
instructors.

```julia
julia> inst = dataset("lme4","InstEval");

julia> head(inst)
6x7 DataFrame
|-------|---|------|---------|---------|---------|------|---|
| Row # | S | D    | Studage | Lectage | Service | Dept | Y |
| 1     | 1 | 1002 | 2       | 2       | 0       | 2    | 5 |
| 2     | 1 | 1050 | 2       | 1       | 1       | 6    | 2 |
| 3     | 1 | 1582 | 2       | 2       | 0       | 2    | 5 |
| 4     | 1 | 2050 | 2       | 2       | 1       | 3    | 3 |
| 5     | 2 | 115  | 2       | 1       | 0       | 5    | 2 |
| 6     | 2 | 756  | 2       | 1       | 0       | 5    | 4 |

julia> @time fm2 = fit(lmm(Y ~ Dept*Service + (1|S) + (1|D), inst), true)
f_1: 241920.83782, [1.0,1.0]
f_2: 244850.35313, [1.75,1.0]
f_3: 242983.26659, [1.0,1.75]
f_4: 238454.23551, [0.25,1.0]
f_5: 241716.05374, [1.0,0.25]
f_6: 240026.05964, [0.0,0.445919]
f_7: 241378.58265, [0.0,1.27951]
f_8: 238463.85417, [0.346954,0.95699]
f_9: 238337.43511, [0.25115,0.933415]
f_10: 238450.00008, [0.195179,0.871898]
f_11: 238282.78311, [0.268777,0.913682]
f_12: 238250.87656, [0.280725,0.897062]
f_13: 238207.46301, [0.303696,0.863175]
f_14: 238195.80295, [0.319029,0.842641]
f_15: 238196.40553, [0.324123,0.837137]
f_16: 238179.53711, [0.317249,0.835355]
f_17: 238160.85286, [0.312094,0.829907]
f_18: 238126.53332, [0.303016,0.817967]
f_19: 238066.44966, [0.289624,0.791122]
f_20: 237960.37093, [0.275292,0.732859]
f_21: 237771.65691, [0.259406,0.613915]
f_22: 237651.6783, [0.242704,0.374497]
f_23: 240177.28897, [0.0941482,0.185999]
f_24: 237591.38115, [0.283051,0.410359]
f_25: 237898.58452, [0.330503,0.300139]
f_26: 237625.00815, [0.312366,0.409428]
f_27: 237599.45773, [0.286273,0.470272]
f_28: 237599.86007, [0.287796,0.399337]
f_29: 237586.35767, [0.281267,0.433017]
f_30: 237586.23507, [0.280855,0.432391]
f_31: 237586.04587, [0.280107,0.432456]
f_32: 237585.75792, [0.278609,0.432398]
f_33: 237585.55709, [0.275788,0.431376]
f_34: 237585.58866, [0.275575,0.430052]
f_35: 237585.59988, [0.274638,0.432066]
f_36: 237585.55357, [0.275935,0.432124]
f_37: 237585.56011, [0.275855,0.432869]
f_38: 237585.55342, [0.275921,0.432005]
f_39: 237585.5536, [0.275995,0.431996]
f_40: 237585.55345, [0.275908,0.431931]
f_41: 237585.55355, [0.275846,0.432008]
f_42: 237585.55342, [0.275915,0.431994]
f_43: 237585.55342, [0.275916,0.432003]
f_44: 237585.55342, [0.275916,0.431994]
f_45: 237585.55342, [0.275914,0.431994]
f_46: 237585.55342, [0.275915,0.431993]
f_47: 237585.55342, [0.275915,0.431994]
XTOL_REACHED
elapsed time: 7.679552299 seconds (262493008 bytes allocated)
Linear mixed model fit by maximum likelihood
 logLik: -118792.776708, deviance: 237585.553415

 Variance components:
                Variance    Std.Dev.
 S              0.105418    0.324681
 D              0.258416    0.508347
 Residual       1.384728    1.176745
 Number of obs: 73421; levels of grouping factors: 2972, 1128

  Fixed-effects parameters:
        Estimate Std.Error   z value
 [1]     3.22961  0.064053   50.4209
 [2]    0.129536  0.101294   1.27882
 [3]   -0.176751 0.0881352  -2.00545
 [4]   0.0517102 0.0817524  0.632522
 [5]   0.0347319  0.085621  0.405647
 [6]     0.14594 0.0997984   1.46235
 [7]    0.151689 0.0816897   1.85689
 [8]    0.104206  0.118751  0.877517
 [9]   0.0440401 0.0962985  0.457329
[10]   0.0517546 0.0986029  0.524879
[11]   0.0466719  0.101942  0.457828
[12]   0.0563461 0.0977925   0.57618
[13]   0.0596536  0.100233   0.59515
[14]  0.00556281  0.110867 0.0501756
[15]    0.252025 0.0686507   3.67112
[16]   -0.180757  0.123179  -1.46744
[17]   0.0186492  0.110017  0.169512
[18]   -0.282269 0.0792937  -3.55979
[19]   -0.494464 0.0790278  -6.25683
[20]   -0.392054  0.110313  -3.55403
[21]   -0.278547 0.0823727  -3.38154
[22]   -0.189526  0.111449  -1.70056
[23]   -0.499868 0.0885423  -5.64553
[24]   -0.497162 0.0917162  -5.42065
[25]    -0.24042 0.0982071   -2.4481
[26]   -0.223013 0.0890548  -2.50422
[27]   -0.516997 0.0809077  -6.38997
[28]   -0.384773  0.091843  -4.18946
```

Models with vector-valued random effects can be fit
```julia
julia> sleep = dataset("lme4","sleepstudy")
180x3 DataFrame
|-------|----------|------|---------|
| Row # | Reaction | Days | Subject |
| 1     | 249.56   | 0    | 308     |
| 2     | 258.705  | 1    | 308     |
| 3     | 250.801  | 2    | 308     |
| 4     | 321.44   | 3    | 308     |
| 5     | 356.852  | 4    | 308     |
| 6     | 414.69   | 5    | 308     |
| 7     | 382.204  | 6    | 308     |
| 8     | 290.149  | 7    | 308     |
| 9     | 430.585  | 8    | 308     |
⋮
| 171   | 269.412  | 0    | 372     |
| 172   | 273.474  | 1    | 372     |
| 173   | 297.597  | 2    | 372     |
| 174   | 310.632  | 3    | 372     |
| 175   | 287.173  | 4    | 372     |
| 176   | 329.608  | 5    | 372     |
| 177   | 334.482  | 6    | 372     |
| 178   | 343.22   | 7    | 372     |
| 179   | 369.142  | 8    | 372     |
| 180   | 364.124  | 9    | 372     |

julia> lmm(Reaction ~ Days + (Days|Subject), sleep)
Linear mixed model fit by maximum likelihood
 logLik: -875.969672, deviance: 1751.939345

 Variance components:
                Variance    Std.Dev.
 Subject      565.722158   23.784915
 Residual      32.463510    5.697676
 Number of obs: 180; levels of grouping factors: 18

  Fixed-effects parameters:
     Estimate Std.Error z value
[1]   251.405   6.63225 37.9065
[2]   10.4673   1.50219 6.96801
```

At present the estimated covariance of vector-valued random effects is
not printed in the model summary.
