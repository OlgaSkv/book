## Checking Outliers

The plot and predictive intervals suggest that predictions for Case 39 are not well captured by the model. There is always the possibility that this case does not meet the assumptions of the simple linear regression model (wrong mean or variance) or could be in error. Model diagnostics such as plots of residuals versus fitted values are useful in identifying potential outliers. Now with the interpretation of Bayesian paradigm, we can go further to calculate the probability to demonstrate whether a case falls too far from the mean. 

The article by Chaloner & Brant (1988) (???reference????) suggested an approach for defining outliers and then calculating the probability that a case or multiple cases were outliers. The assumed model for our simple linear regression is $y_i=\alpha + \beta x_i+\epsilon_i$, with $\epsilon_i$ having independent, identical distributions that are normal with mean zero and constant variance $\sigma^2$, i.e., $\epsilon_i \sim \mathcal{N}(0, \sigma^2)$. Chaloner & Brant consider outliers to be points where the error or the model discrepancy $\epsilon_i$ is greater than $k$ standard deviation for some large $k$, and then proceed to calculate the posterior probability that a case is an outlier to be
$$ \mathbb{P}(|\epsilon_i| > k\sigma ~|~\text{data}) $$

Since $\epsilon_i = y_i - \alpha-\beta x_i$, this is equivalent to calculating
$$ \mathbb{P}(|y_i-\alpha-\beta x_i| > k\sigma~|~\text{data}).$$

### Posterior Distribution of $\epsilon_i$ Conditioning On $\sigma$



### Implementation Using `BAS` Package

The code for calculating the probability of outliers involves integration. We have implemented this in the function `Bayes.outlier.prob` that can be sourced from the file `bayes-outliers.R`. Applying this to the `bodyfat` data for Case 39, we get

```r
library(BAS)
data(bodyfat)
#source("bayes-outliers.R")
#library(mvtnorm)
#outliers = Bayes.outlier.prob(bodyfat.lm)

# The default is to consider k=3
#prob.39 = outliers$prob.outlier[39]
#prob.39
```

We see that this case has an extremely high probability (0.9917) of being more an outlier, that is, the error is greater than $k=3$ standard deviations, based on the fitted model and data.

With $k=3$, however, there may be a high probability a priori of at least one outlier in a large sample. We can compute this using


```r
n = nrow(bodyfat)
# probability of no outliers if outliers are error greater than 3 standard deviation
prob = (1 - (2 * pnorm(-3))) ^ n
prob
```

```
## [1] 0.5059747
```

```r
# probability of at least one outlier
prob.least1 = 1 - (1 - (2 * pnorm(-3))) ^ n
prob.least1
```

```
## [1] 0.4940253
```

With $n=252$, the probability of at least one outlier is much larger than say the marginal probability that one point is an outlier of 0.05. So we would expect that there will be at least one point where the error is more than 3 standard deviations from zero almost 50% of the time. Rather than fixing $k$, we can fix the prior probability of no outliers to be say 0.95, and solve for the value of $k$.


```r
k = qnorm(0.5 + 0.5 * 0.95 ^ (1 / n))
k
```

```
## [1] 3.714602
```

This leads to a larger value of $k$ after adjusting $k$ so that there is at least one outlier. We examine Case 39 again under this $k$


```r
#outliers.no = Bayes.outlier.prob(bodyfat.lm, k = k)
#prob.no.39 = outliers.no$prob.outlier[39]
#prob.no.39
```

The posterior probability of Case 39 being an outlier is 0.68475. While this is not strikingly large, it is much larger than the marginal prior probability of

```r
2 * pnorm(-k)
```

```
## [1] 0.0002035241
```


### Summary

There is a substantial probability that Case 39 is an outlier. If you do view it as an outlier, what are your options? One option is to investigate the case and determine if the data are input incorrectly, and fix it. Another option is when you cannot confirm there is a data entry error, you may delete the observation from the analysis and refit the model without the case. If you do take this option, be sure to describe what you did so that your research is reproducible. You may want to apply diagnostics and calculate the probability of a case being an outlier using this reduced data. As a word of caution, if you discover that there are a large number of points that appear to be outliers, take a second look at your model assumptions, since the problem may be with the model rather than the data! A third option we will talk about later, is to combine inference under the model that retains this case as part of the population, and the model that treats it as coming from another population. This approach incorporates our uncertainty about whether the case is an outlier given the data.

The source code is based on using a reference prior for the linear model and extends to multiple regression.