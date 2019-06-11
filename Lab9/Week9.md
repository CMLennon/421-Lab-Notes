---
Title: Lab 9 2SLS, Causality and IV
date: June 1 2019
output:
  html_document:
    highlight: haddock
    keep_md: yes
    theme: flatly
    toc: yes
    toc_depth: 4
    toc_float: yes
---


# Week 9 Lab: Instrumental Variables

Welcome to week 9! You're almost done with this course! Next week will be a review week for the final, so this is our last R-lab. Let's get started.

Today, we're in a similar situation to last week: you may not learn about instrumental variables in full until your next lecture. So, we're going to do a very soft introduction to IV, with some coding examples. There WILL be a homework 4 help posted as well on the note page. We're going to start with an intuitive approach to instrumentation, and then move onto some coding.

## Lesson 1: IV, a motivation

### Instrumental Variables:
  
IV is a tool that you can use when you want to estimate the causal effect of some variable on an outcome, but that variable is 'messy.' in its effect. In particular, it is useful when it is hard to distinguish whether some coefficient you've estimated is simply do to an unfortunate correlation with something else that has an impact on your outcome.

I think the best way to motivate this is with an example. Suppose you estimate a regression of income on years of schooling, ie:
  
`income_i = b0+b1*schooling_i+u_i`
  
One assumption we make for OLS to be BLUE is exogeneity, or in other words: 

`E[schooling|u] = 0`

Does this seem reasonable in this case? I certainly do not think so. It is almost certain that schooling and income are both correlated with the error u, because u contains some measure of 'natural ability', which we cannot observe. Further, someone with more income might choose to get more school because their parents were wealthy. Or, growing up wealthy puts more pressure on kids to go to college. There are so many problems.

In other words, if an individual has higher ability, or childhood income impacts choice of schooling it is likely that both their salary and schooling will be higher. One can see that if E[schooling|u] != 0, our estimate for **b1** will be inconsistent and biased.

**Recall**, we can write:
  
`b_hat = b + cov(x,u)/var(x)`

So if we think `cov(x,u)` is NOT equal to zero, then our estimates of **b** will be biased.

So what do we do? Give up? Sometimes that's the best approach. Sometimes however, you CAN do something. 

It's called magic.

But, even other times, we use an approach called **Instrumental Variables** or (IV). And that's pretty much as good as magic.

Before giving you the formal definition of an instrument, lets talk intuition for a second.

Okay, so maybe the covariance of x and u isn't zero, that is, we have some variation in an excluded variable potentially, or a 'two-way-street' referred to as 'simultaneity bias.' But maybe, we have some other variable we can use that explains our causal variable well, but does not explain our outcome.

Lets imagine you have a perfect, worldwide laboratory, and you could change some policy that would 

- impact peoples' schooling choices, but 
- not their income. At least, not directly. 
 
A good example: mandatory minimum schooling for highschool - some minimum amount of schooling required by the government.

Now, assuming we can  manipulate this new variable, we can see how differing levels of mandatory minimum schooling impacts students' education levels, and use THAT change to estimate changes in income. Perhaps by regressing mandatory minimums on education levels, and then predicted education levels on income. A **two-stage**, **least squares**. (2SLS)

**Formal Definition**: Instrumental Variable:
  
*An instrumental variable, z for the regression y= b1\*x+u is one that satsifies the following two conditions:*
  
1. Exogeneity - it is uncorrelated with the error, u
  
2. Relevance - it is correlated with x


Let's draw a venn diagram. This helps solidify the true idea of IV. These I've borrowed unceremoniously from MY graduate metrics professor Glenn Waddell. He's the first person I've seen do this type of explanation for IV.


Think of these like so:

 - each circle illustrates a variable
 - overlap gives the share of correlation
 - dotted borders denote omitted variables.

Our problem, with a series of instruments:

![Initial](Lab9/Lab9_files/venn_iv-1.png)

![endogenous](Lab9/Lab9_files/venn_iv_endog-1.png)

![not relevant](/Users/connor/Dropbox/EC421_Lab/week9/Week9_files/venn_iv_irrelevant-1.png)

![emdogenous](/Users/connor/Dropbox/EC421_Lab/week9/Week9_files/venn_iv_endog2-1.png)

Let's look at an example, one you talked about a bit in class.

A common instrument (made popular by Card (1995)) for schooling in the income regression is to use inidividual proximity to a college. Lets think about if this satisifies our assumptions. 

Any thoughts? Is this exogenous to income? Why might it be, why might not it be?

It's reasonable. but I think there is a strong argument here. 

Only issue you might think that people that live in cities usually have higher wages than rural areas- this is definitely true. 

An easy fix would be to control for metropolitan area/farmland.

How about relevance? How can we make sure this variable isn't relevance? How would you tell if this variable was relevant to education in a normal context?

This one we can test. How close you live to a school almost certainly affects how likely you are to attend school. 

How would we test this? We could test by running a *regression* estimating the effect of distance on years of schooling:
  
`schooling_i = a0+b1*distance_i+e_i`
  
How would you know if the distance impacts schooling? 

Do a t-test where H_0: b1=0. Check the significance. You know, a p-value!


## Lesson 2: Implementation

How would we go about estimating an IV regression?
  
### 2 Stage Least Squares (2SLS)
  
#### Overview

This technique is called **two stage**... **least squares** because we estimate **two** ordinary **least squared** **regressions**. However, I really like to think of this as being a 4 stage procedure. Here is the general set-up; ill give you the "two" official stages in the general setting and then give an example where I illustrate why I believe you actually need a few more 'unofficial steps.' Suppose you are interested in estimating:
  
`y= b0+b1*x+u`
  
but you are concerned about the endogeneity of x. We could use some kind of proxy, say, z, using the following regression:

`y_i = n0 + n1*z_i + m_i`

but then we're worried that perhaps someone would argue we aren't telling a good story. We really only care about X.

So we can use z, if it's a valid instrument. In our above example, z would be distance from college and x would be schooling. To implement 2SLS we do the following:
  
 **Step 1**: Regress x on z and save the predicted values of x. That is, estimate the regression:
  
`x_i=  a0+a1*z_i+e_i`

From this regression we can calculate **x-hat**. Note that this regression is telling us what part/how much of x is explained by the exogenous variable z. Any left-over variation (the bad, endogenous stuff, plus any bits not related to z) are thrown into e_i
  
**Step 2**: Now, take the predicted values from stage 1 and estimate the following regression:
  
`y_i =c0 + c1*x_hat +q_i`

Where *q_i* is a new error term.

So if we did our job well, and our instrument is really exogenous and relevant, the estimate of c1 will be consistent. Note that q_i is not correlated with x_hat since this will only be the 'part' of the variation of x that is due to z, which we have claimed is exogenous to q_i.

Let's think about our new coefficient, c1.

c1 is now equal to, really, the coefficient of y_i on z divided by the coefficient of
y_i on X. That is,

`c1 = n1/b1`

Which gives us a plim such that

`plim(c1) = B1 + cov(z,u)/cov(z,X)`

This gives us a nice summary of our issues at hand. The numerator (cov(z,u)) ought to be SMALL and the denominator ought to be LARGE (cov(z,X)). What does this mean?

Well, we don't want z to covary with our excluded terms (u), and we want z to do at least a semi-decent job of 'mimicking' x (cov(z,X)>0).

### The 4 steps of 2SLS

Back to the schooling regression. Here are the *real* steps to 2SLS. We need to convince everyone that the first two stages are even worth our time!!

So. Let's go through these four steps.

 **1.** Find an instrument

**2.** Argue that it is exogenous.

 **3.** Stage 1. Demonstrate that it is relevant (how?).

 **4.** Stage 2 regression.

## Lesson 3: Estimating 2SLS in R

As per usual, we'll need to load some packages.

`install.packages("pacman",dependencies = T,repos = "http://cran.us.r-project.org")` 

if needed


```r
library(pacman)
p_load(tidyverse, magrittr, ggplot2, lmtest, MASS,AER, stargazer, knitr, arm)
```

We need to get some data to manipulate. 

We're going to use data from the education-income example above to examine the effects here.

our dataset is contained in the `AER` package, but it takes a strange call to access it. Let's use it.


```r
#in order to load this data onto our system, we need to use the 'data()' command.
data("CollegeDistance")

#assign a new name: you don't need to do this, but I did.
wage_data<-CollegeDistance

#let's look at available variables...
names(wage_data)
```

```
##  [1] "gender"    "ethnicity" "score"     "fcollege"  "mcollege" 
##  [6] "home"      "urban"     "unemp"     "wage"      "distance" 
## [11] "tuition"   "education" "income"    "region"
```

what variables do we have? Let's take a look at some summary statistics


```r
summary(wage_data)
```

```
##     gender        ethnicity        score       fcollege   mcollege  
##  male  :2139   other   :3050   Min.   :28.95   no :3753   no :4088  
##  female:2600   afam    : 786   1st Qu.:43.92   yes: 986   yes: 651  
##                hispanic: 903   Median :51.19                        
##                                Mean   :50.89                        
##                                3rd Qu.:57.77                        
##                                Max.   :72.81                        
##   home      urban          unemp             wage           distance     
##  no : 852   no :3635   Min.   : 1.400   Min.   : 6.590   Min.   : 0.000  
##  yes:3887   yes:1104   1st Qu.: 5.900   1st Qu.: 8.850   1st Qu.: 0.400  
##                        Median : 7.100   Median : 9.680   Median : 1.000  
##                        Mean   : 7.597   Mean   : 9.501   Mean   : 1.803  
##                        3rd Qu.: 8.900   3rd Qu.:10.150   3rd Qu.: 2.500  
##                        Max.   :24.900   Max.   :12.960   Max.   :20.000  
##     tuition         education      income       region    
##  Min.   :0.2575   Min.   :12.00   low :3374   other:3796  
##  1st Qu.:0.4850   1st Qu.:12.00   high:1365   west : 943  
##  Median :0.8245   Median :13.00                           
##  Mean   :0.8146   Mean   :13.81                           
##  3rd Qu.:1.1270   3rd Qu.:16.00                           
##  Max.   :1.4042   Max.   :18.00
```

### OLS: Naive

Now, lets estimate the returns to education on income the 'naive' way, via OLS:

Why is it *naive* to do this?
  
  

```r
ols_mod <- lm(wage ~ education+urban + gender + ethnicity + unemp +income , data=wage_data)
summary(ols_mod)
```

```
## 
## Call:
## lm(formula = wage ~ education + urban + gender + ethnicity + 
##     unemp + income, data = wage_data)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3790 -0.8513  0.1701  0.8286  3.8570 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)        8.685542   0.157062  55.300  < 2e-16 ***
## education         -0.003891   0.010568  -0.368   0.7128    
## urbanyes           0.078633   0.044691   1.760   0.0786 .  
## genderfemale      -0.076373   0.037060  -2.061   0.0394 *  
## ethnicityafam     -0.533688   0.052335 -10.197  < 2e-16 ***
## ethnicityhispanic -0.516272   0.049014 -10.533  < 2e-16 ***
## unemp              0.135182   0.006716  20.127  < 2e-16 ***
## incomehigh         0.181047   0.042397   4.270 1.99e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.266 on 4731 degrees of freedom
## Multiple R-squared:  0.1132,	Adjusted R-squared:  0.1119 
## F-statistic: 86.29 on 7 and 4731 DF,  p-value: < 2.2e-16
```

Note that in the case of OLS, there is no significant effect of education on earnings. I guess y'all should leave school now!

Let's use our "2SLS" approach outlined above.

### 2SLS: R tutorial

Now lets treat education as endogenous and instrument for it using distance to college. Recall, we nee two conditions to hold for `distance` to be **valid** instrument. What are they?

 1. **Exogeneity**: Does it directly impact my earnings? Probably not. I am talking about the direct effect of distance on education. Anything we can control for (such as living in a city vs the country) isnt really an issue. Ask yourself this: when you go apply for a job, would it be strange for the employer to ask you how far away

2. **Relevance**: Does it impact my own level of schooling? This one we can test. And the answer is probably. 

Okay, now lets implement 2SLS

#### Stage 1:

**Note: in an earlier stage of these notes, I mistakenly forgot to include the control variables in the first stage, which is an important step. If you do this, this should produce the same results in both regressions.**


```r
#regress education on distance from a place of higher ed
stage_one <- lm(education~distance + urban + gender + ethnicity + unemp + income, data=wage_data)
```

#### check relevance

```r
summary(stage_one)
```

```
## 
## Call:
## lm(formula = education ~ distance + urban + gender + ethnicity + 
##     unemp + income, data = wage_data)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -2.679 -1.564 -0.465  1.479  4.691 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       13.678428   0.085939 159.165  < 2e-16 ***
## distance          -0.072826   0.012045  -6.046 1.60e-09 ***
## urbanyes          -0.035527   0.063884  -0.556   0.5782    
## genderfemale       0.015520   0.050790   0.306   0.7599    
## ethnicityafam     -0.403432   0.071541  -5.639 1.81e-08 ***
## ethnicityhispanic -0.144992   0.067187  -2.158   0.0310 *  
## unemp              0.016602   0.009586   1.732   0.0833 .  
## incomehigh         0.794369   0.057067  13.920  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.735 on 4731 degrees of freedom
## Multiple R-squared:  0.06142,	Adjusted R-squared:  0.06003 
## F-statistic: 44.23 on 7 and 4731 DF,  p-value: < 2.2e-16
```

Great. Looks like we're relevant! (why?)

#### get fitted values and add them to our data-frame

We can use a command `fitted.values()` to return fitted values from a regression object. Let's attach that object to our original `wage_data` dataframe.


```r
wage_data$x_hat <- fitted.values(stage_one)
```

#### Stage 2

Now we need to run the regression, using our new `x_hat` variable


```r
stage_two <- lm(wage~urban + gender + ethnicity + unemp+x_hat+income, data=wage_data)
```

Lets compare the results. 


```r
summary(ols_mod, robust = TRUE)
```

```
## 
## Call:
## lm(formula = wage ~ education + urban + gender + ethnicity + 
##     unemp + income, data = wage_data)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.3790 -0.8513  0.1701  0.8286  3.8570 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)        8.685542   0.157062  55.300  < 2e-16 ***
## education         -0.003891   0.010568  -0.368   0.7128    
## urbanyes           0.078633   0.044691   1.760   0.0786 .  
## genderfemale      -0.076373   0.037060  -2.061   0.0394 *  
## ethnicityafam     -0.533688   0.052335 -10.197  < 2e-16 ***
## ethnicityhispanic -0.516272   0.049014 -10.533  < 2e-16 ***
## unemp              0.135182   0.006716  20.127  < 2e-16 ***
## incomehigh         0.181047   0.042397   4.270 1.99e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.266 on 4731 degrees of freedom
## Multiple R-squared:  0.1132,	Adjusted R-squared:  0.1119 
## F-statistic: 86.29 on 7 and 4731 DF,  p-value: < 2.2e-16
```

```r
summary(stage_two, robust = TRUE)
```

```
## 
## Call:
## lm(formula = wage ~ urban + gender + ethnicity + unemp + x_hat + 
##     income, data = wage_data)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.1816 -0.8702  0.1452  0.8464  3.8382 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       -1.37169    1.64027  -0.836 0.403053    
## urbanyes           0.02368    0.04540   0.522 0.602032    
## genderfemale      -0.09017    0.03698  -2.438 0.014804 *  
## ethnicityafam     -0.24668    0.06992  -3.528 0.000423 ***
## ethnicityhispanic -0.39711    0.05252  -7.562 4.75e-14 ***
## unemp              0.13487    0.00669  20.160  < 2e-16 ***
## x_hat              0.73370    0.12021   6.103 1.12e-09 ***
## incomehigh        -0.42616    0.10724  -3.974 7.18e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.261 on 4731 degrees of freedom
## Multiple R-squared:  0.1201,	Adjusted R-squared:  0.1188 
## F-statistic: 92.27 on 7 and 4731 DF,  p-value: < 2.2e-16
```

After estimating the equation via 2SLS, we have significance on x_hat! (the causal effect of education on earnings). Phew. I can tell you to stay in school.

There is also a built in way to do this in R, called ivreg. Let's do this process using ivreg.

This is a fun tool. You separate, in a sense, your two stages with a `|` operator. On the left side, put your original equation. On the right side put what variables you will use to instrument. Of course, R won't know what you're instrumenting for, so you need to tell it with a `.-` before it. In our case, `.-education`.



```r
#this command comes from the AER package. There is also IV_robust you can use as well.
reg_iv <- AER::ivreg(wage~urban + gender + ethnicity + unemp + income + education|.-education + distance, data=wage_data)
summary(reg_iv)
```

```
## 
## Call:
## AER::ivreg(formula = wage ~ urban + gender + ethnicity + unemp + 
##     income + education | . - education + distance, data = wage_data)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -5.57678 -1.21632 -0.02711  1.40379  5.07247 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       -1.371690   2.346001  -0.585  0.55878    
## urbanyes           0.023678   0.064936   0.365  0.71540    
## genderfemale      -0.090166   0.052895  -1.705  0.08833 .  
## ethnicityafam     -0.246683   0.100003  -2.467  0.01367 *  
## ethnicityhispanic -0.397113   0.075111  -5.287 1.30e-07 ***
## unemp              0.134875   0.009569  14.095  < 2e-16 ***
## incomehigh        -0.426157   0.153387  -2.778  0.00549 ** 
## education          0.733704   0.171931   4.267 2.02e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.803 on 4731 degrees of freedom
## Multiple R-Squared: -0.7999,	Adjusted R-squared: -0.8026 
## Wald test:  45.1 on 7 and 4731 DF,  p-value: < 2.2e-16
```

And that's it for today, we'll see you next week for a review and keep an eye out for a homework help lab posted soon.
