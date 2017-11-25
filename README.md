## RegressionTables.jl

This package provides publication-quality regression tables for use with [FixedEffectModels.jl](https://github.com/matthieugomez/FixedEffectModels.jl).

In its objective it is similar to  (and heavily inspired by) the Stata command [`esttab`](http://repec.sowi.unibe.ch/stata/estout/esttab.html) and the R package [`stargazer`](https://cran.r-project.org/web/packages/stargazer/).

To install the package, type in the Julia command prompt

```julia
Pkg.clone("http://github.com/jmboehm/RegressionTables.jl.git")
```

## A brief demonstration

```julia
using RegressionTables, FixedEffectModels, RDatasets

df = dataset("datasets", "iris")
df[:SpeciesDummy] = pool(df[:Species])

rr1 = reg(df, @model(SepalLength ~ SepalWidth   , fe = SpeciesDummy))
rr2 = reg(df, @model(SepalLength ~ SepalWidth + PetalLength   , fe = SpeciesDummy))
rr3 = reg(df, @model(SepalLength ~ SepalWidth + PetalLength + PetalWidth  , fe = SpeciesDummy))
rr4 = reg(df, @model(SepalWidth ~ SepalLength + PetalLength + PetalWidth  , fe = SpeciesDummy))

RegressionTables.regtable(rr1,rr2,rr3,rr4; renderSettings = RegressionTables.asciiOutput())
```
yields
```
---------------------------------------------------------
                        SepalLength            SepalWidth
              ------------------------------   ----------
                   (1)        (2)        (3)          (4)
---------------------------------------------------------
SepalWidth    0.804***   0.432***   0.496***             
               (0.106)    (0.081)    (0.086)             
PetalLength              0.776***   0.829***      -0.188*
                          (0.064)    (0.069)      (0.083)
PetalWidth                           -0.315*     0.626***
                                     (0.151)      (0.123)
SepalLength                                      0.378***
                                                  (0.066)
---------------------------------------------------------
N                  150        150        150          150
R2               0.726      0.863      0.867        0.635
---------------------------------------------------------
```
LaTeX output can be produced by using
```julia
RegressionTables.regtable(rr1,rr2,rr3,rr4; renderSettings = RegressionTables.latexOutput())
```
which yields
```
\begin{tabular}{lrrrr}
\toprule
            & \multicolumn{3}{c}{SepalLength} & \multicolumn{1}{c}{SepalWidth} \\
\cmidrule(lr){2-4} \cmidrule(lr){5-5}
            &      (1) &      (2) &       (3) &                            (4) \\
\midrule
SepalWidth  & 0.804*** & 0.432*** &  0.496*** &                                \\
            &  (0.106) &  (0.081) &   (0.086) &                                \\
PetalLength &          & 0.776*** &  0.829*** &                        -0.188* \\
            &          &  (0.064) &   (0.069) &                        (0.083) \\
PetalWidth  &          &          &   -0.315* &                       0.626*** \\
            &          &          &   (0.151) &                        (0.123) \\
SepalLength &          &          &           &                       0.378*** \\
            &          &          &           &                        (0.066) \\
\midrule
N           &      150 &      150 &       150 &                            150 \\
R2          &    0.726 &    0.863 &     0.867 &                          0.635 \\
\bottomrule
\end{tabular}
```

## Options

### Function Arguments
* `rr::AbstractRegressionResult...` are the `AbstractRegressionResult`s from `FixedEffectModels.jl` that should be printed. Only required argument.
* `regressors` is a `Vector` of regressor names (`String`s) that should be shown, in that order. Defaults to an empty vector, in which case all regressors will be shown.
* `fixedeffects` is a `Vector` of FE names (`String`s) that should be shown, in that order. Defaults to an empty vector, in which case all FE's will be shown.
* `labels` is a `Dict` that contains displayed labels for variables (strings) and other text in the table. If no label for a variable is found, it default to variable names. See documentation for special values.
* `estimformat` is a `String` that describes the format of the estimate. Defaults to "%0.3f".
* `estim_decoration` is a `Function` that takes the formatted string and the p-value, and applies decorations (such as the beloved stars). Defaults to (* p<0.05, ** p<0.01, *** p<0.001).
* `statisticformat` is a `String` that describes the format of the number below the estimate (se/t). Defaults to "%0.4f".
* `below_statistic` is a `Symbol` that describes a statistic that should be shown below each point estimate. Recognized values are `:blank`, `:se`, and `:tstat`. Defaults to `:se`.
* `below_decoration` is a `Function` that takes the formatted statistic string, and applies a decorations. Defaults to round parentheses.
* `regression_statistics` is a `Vector` of `Symbol`s that describe statistics to be shown at the bottom of the table. Recognized symbols are `:nobs`, `:r2`, `:r2_a`, `:r2_within`, `:f`, `:p`, `:f_kp`, `:p_kp`, and `:dof`. Defaults to `[:nobs, :r2]`.
* `number_regressions` is a `Bool` that governs whether regressions should be numbered. Defaults to `true`.
* `number_regressions_decoration` is a `Function` that governs the decorations to the regression numbers. Defaults to `s -> "($s)"`.
* `print_fe_section` is a `Bool` that governs whether a section on fixed effects should be shown. Defaults to `true`.
* `print_estimator_section`  is a `Bool` that governs whether to print a section on which estimator (OLS/IV) is used. Defaults to `true`.
* `renderSettings::RenderSettings` is a `RenderSettings` composite type that governs how the table should be rendered. Standard supported types are ASCII (via `asciiOutput(outfile::String)`) and LaTeX (via `latexOutput(outfile::String)`). If no argument to these two functions are given, the output is sent to STDOUT. Defaults to ASCII with STDOUT.

### Label Codes

The following is the exhaustive list of strings that govern the output of labels. Use e.g.
```julia
label = Dict("__LABEL_STATISTIC_N__" => "Number of observations")
```
to change the label for the row showing the number of observations in each regression.

* `__LABEL_ESTIMATOR__` (default: "Estimator")
* `__LABEL_ESTIMATOR_OLS__` (default: "OLS")
* `__LABEL_ESTIMATOR_IV__` (default: "IV")

* `__LABEL_FE_YES__` (default: "Yes")
* `__LABEL_FE_NO__` (default: "")

* `__LABEL_STATISTIC_N__` (default: "N" in `asciiOutput()`)
* `__LABEL_STATISTIC_R2__` (default: "R2" in `asciiOutput()`)
* `__LABEL_STATISTIC_R2_A__` (default: "Adjusted R2" in `asciiOutput()`)
* `__LABEL_STATISTIC_R2_WITHIN__` (default: "Within-R2" in `asciiOutput()`)
* `__LABEL_STATISTIC_F__` (default: "F" in `asciiOutput()`)
* `__LABEL_STATISTIC_P__` (default: "F-test p value" in `asciiOutput()`)
* `__LABEL_STATISTIC_F_KP__` (default: "First-stage F statistic" in `asciiOutput()`)
* `__LABEL_STATISTIC_P_KP__` (default: "First-stage p value" in `asciiOutput()`)
* `__LABEL_STATISTIC_DOF__` (default: "Degrees of Freedom" in `asciiOutput()`)
