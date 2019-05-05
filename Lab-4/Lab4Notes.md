---
Title: Lab 4 Heteroskedasticity Tests and Correction
date: 25 April 2019
output:
  html_document:
    highlight: haddock
    keep_md: yes
    theme: flatly
    toc: yes
    toc_depth: 4
    toc_float: yes
---

# Welcome to week 4!!!

Today, we will be working through some heteroskedasticity examples. 

In order to do that though, we need two things: some data and our handy dandy pacman package. We'll be using the `Caschool` data from `Ecdat` to look at some fun data.

## Lesson 0: loading data and packages

First, let's load pacman. If you need to, run `install.packages("pacman")` first. If you run into errors, run
`library(Ecdat,ggplot2)` to get around pacman install concerns. For everybody else,


```r
library(pacman)
#install.packages("Ecdat") if necessary
p_load(Ecdat, tidyverse, ggplot2, sandwich, lmtest)
#Let's explore our dataset, Caschool
```

The `Ecdat` contains the `Caschool` dataset which features all sorts of fun heterokedasticity we can look at.

Let's do some exploration on our own. How do we inspect something? We can ask R using the `?` key


```r
?Caschool
```

We're going to start with some exploration on our own.


- Question1: Run this command on your own. How many observations are there? What is the timespan the data covers?

Let's take a peak at the first few rows of the dataset. How do we do this?



```r
#Let's pass in our caschool data to a head command
head(Caschool)
```

```
##   distcod  county                        district grspan enrltot teachers
## 1   75119 Alameda              Sunol Glen Unified  KK-08     195    10.90
## 2   61499   Butte            Manzanita Elementary  KK-08     240    11.15
## 3   61549   Butte     Thermalito Union Elementary  KK-08    1550    82.90
## 4   61457   Butte Golden Feather Union Elementary  KK-08     243    14.00
## 5   61523   Butte        Palermo Union Elementary  KK-08    1335    71.50
## 6   62042  Fresno         Burrel Union Elementary  KK-08     137     6.40
##   calwpct mealpct computer testscr   compstu  expnstu      str    avginc
## 1  0.5102  2.0408       67  690.80 0.3435898 6384.911 17.88991 22.690001
## 2 15.4167 47.9167      101  661.20 0.4208333 5099.381 21.52466  9.824000
## 3 55.0323 76.3226      169  643.60 0.1090323 5501.955 18.69723  8.978000
## 4 36.4754 77.0492       85  647.70 0.3497942 7101.831 17.35714  8.978000
## 5 33.1086 78.4270      171  640.85 0.1280899 5235.988 18.67133  9.080333
## 6 12.3188 86.9565       25  605.55 0.1824818 5580.147 21.40625 10.415000
##       elpct readscr mathscr
## 1  0.000000   691.6   690.0
## 2  4.583333   660.5   661.9
## 3 30.000002   636.3   650.9
## 4  0.000000   651.9   643.5
## 5 13.857677   641.8   639.9
## 6 12.408759   605.7   605.4
```


- Question 2: What is the range of school size?
- Question 3: What is the range of district average income ?
- Question 4: What is the range of student to teacher ratio?
- Question 5: What is the range of enrollment?
- Question 6: What variables do we have to use to measure student outcomes?

I think using some function of our test scores would work well.

Let's try to figure out what the range of the total average score (reading + math) is? How should we do this?

We'll use some tools from our friendly neighborhood tidyverse to solve this. Remember our mutate command? We'll be using that.


```r
#We need the tidyverse, so if you haven't already, run the following:
#p_load(tidyverse)

#Generate total scores by adding reading and math scores together.
Caschool <- mutate(Caschool, scrtot = readscr + mathscr)
```


Now, we need to do some analysis. Back to regressions.

## Lesson 1: Running Regressions! Again!

Let's run the linear regression of total score on income, student-to-teacher ratio, enrollment and expenditures per student. First, let's explore a little.


```r
ols1 <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu)
summary(ols1)
```

```
## 
## Call:
## lm(formula = scrtot ~ avginc + str + enrltot + expnstu, data = Caschool)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -80.657 -17.753   2.213  17.866  62.657 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  1.313e+03  2.783e+01  47.171  < 2e-16 ***
## avginc       3.864e+00  1.851e-01  20.876  < 2e-16 ***
## str         -1.395e+00  8.927e-01  -1.563   0.1188    
## enrltot     -1.611e-03  3.411e-04  -4.722  3.2e-06 ***
## expnstu     -6.052e-03  2.613e-03  -2.316   0.0211 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 25.86 on 415 degrees of freedom
## Multiple R-squared:  0.5439,	Adjusted R-squared:  0.5395 
## F-statistic: 123.7 on 4 and 415 DF,  p-value: < 2.2e-16
```

On your own, interpret coefficients. In particular, think about the coefficient on expnstu: it's significant at the 5% level. Why might we see a **negative** sign on expenditures per student?

Ok, now we need to find a way to examine heteroskedasticity. The first step in doing that is to recover errors, which we have to figure out. Let's do that.

## Lesson 2: Retrieving Residuals from an LM model
Create the residuals of the regression you just ran. Hint: you can use the mutate command, combined with
the `resid(your_lm_model)` command. This will return a vector of residuals! This is super handy, especially for this section. Let's create a new column in our dataframe with our errors.


```r
Caschool <- mutate(Caschool, resid1 = resid(ols1))
```

However, we know this is the heteroskedasticity lab, so we need to check for heteroskedasticity. How would we do this?

Plot residuals against each of the four explanatory variables!


```r
#Let's use our qplot command to graph our X variables against our residuals to see if they seem reasonable. Think a bit about
#what 'reasonable' means.

#Average income
qplot(Caschool$avginc, Caschool$resid1)
```

![](Lab4Notes_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
#student teacher ratio
qplot(Caschool$str, Caschool$resid1)
```

![](Lab4Notes_files/figure-html/unnamed-chunk-6-2.png)<!-- -->

```r
#enrollment total
qplot(Caschool$enrltot, Caschool$resid1)
```

![](Lab4Notes_files/figure-html/unnamed-chunk-6-3.png)<!-- -->

```r
#expenditures per student
qplot(Caschool$expnstu, Caschool$resid1)
```

![](Lab4Notes_files/figure-html/unnamed-chunk-6-4.png)<!-- -->



Does it appear as though our disturbances are heteroskedastic? explain.

Why do we care about heteroskedasticity? What issues does that cause with our assumptions about OLS?


Let's Run a similar regression but this time let's use some common transformation techiniques. Why do we want to do this?

The reason why we want to first try model specifications is because model mispecification itself can cause heteroskedascity. Why do log-type heteroskedasticities help us sometimes?

Let's start with a log-linear specification. How would we go about this?

```r
#Let's run a regression with  the form log(y) = x1 + x2 + ...
lm_i <- lm(data = Caschool, log(scrtot) ~ avginc + str + enrltot + expnstu)
```


Now, we can use our quick-plot tool to examine our new residuals



```r
qplot(Caschool$avginc, resid(lm_i))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
qplot(Caschool$str, resid(lm_i))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-8-2.png)<!-- -->

```r
qplot(Caschool$enrltot, resid(lm_i))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-8-3.png)<!-- -->

```r
qplot(Caschool$expnstu, resid(lm_i))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-8-4.png)<!-- -->

 Next, let's do a log-log specification

```r
#our form is now log(Y) = log(X1) + log(X2)...
lm_ii <- lm(data = Caschool, log(scrtot) ~ log(avginc) + log(str) + log(enrltot) + log(expnstu))
```
 
qplot again

```r
qplot(log(Caschool$avginc), resid(lm_ii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
qplot(log(Caschool$str), resid(lm_ii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-10-2.png)<!-- -->

```r
qplot(log(Caschool$enrltot), resid(lm_ii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-10-3.png)<!-- -->

```r
qplot(log(Caschool$expnstu), resid(lm_ii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-10-4.png)<!-- -->


```r
lm_checker <- lm(data = Caschool, (resid(lm_ii)^2) ~ log(avginc) + log(str) + log(enrltot))
summary(lm_checker)$r.squared
```

```
## [1] 0.0350631
```


That's a bit hard to interpret. We might be okay here. But let's keep looking. (Note: how would you interpret these coefficients?)

*How about lin-log?*

```r
#y = log(x1)+ log(x2)...
lm_iii <- lm(data = Caschool, scrtot ~ log(avginc) + log(str) + log(enrltot))
```


```r
qplot(log(Caschool$avginc), resid(lm_iii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
qplot(log(Caschool$str), resid(lm_iii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-13-2.png)<!-- -->

```r
qplot(log(Caschool$enrltot), resid(lm_iii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-13-3.png)<!-- -->

```r
qplot(log(Caschool$enrltot), resid(lm_iii))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-13-4.png)<!-- -->


 Maybe interaction terms will help us...

```r
lm_iv <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu + avginc:str + avginc:enrltot + avginc:expnstu + str:enrltot + str:expnstu + enrltot:expnstu)
```
 
Ok, let's look at our errors


```r
qplot(Caschool$avginc, resid(lm_iv))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

```r
qplot(Caschool$str, resid(lm_iv))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-15-2.png)<!-- -->

```r
qplot(Caschool$enrltot, resid(lm_iv))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-15-3.png)<!-- -->

```r
qplot(Caschool$expnstu, resid(lm_iv))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-15-4.png)<!-- -->



v) Okay. Maybe we really need quadratic terms. We'll keep our interactions.


```r
lm_v <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu + avginc:str + avginc:enrltot + avginc:expnstu + str:enrltot + str:expnstu + enrltot:expnstu + avginc ^ 2 + str ^ 2 + enrltot ^ 2 + expnstu^2)
```



```r
qplot(Caschool$avginc, resid(lm_v))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

```r
qplot(Caschool$str, resid(lm_v))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-17-2.png)<!-- -->

```r
qplot(Caschool$enrltot, resid(lm_v))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-17-3.png)<!-- -->

```r
qplot(Caschool$expnstu, resid(lm_v))
```

![](Lab4Notes_files/figure-html/unnamed-chunk-17-4.png)<!-- -->



Shoot. Okay, what can we do from here on out? We need tests. Tests with funny names....

## Lesson 3:Goldfeld-Quant test!

Ok. Before we start down this path, let's try to figure out what exactly we need to look for in a heteroskedasticity test. What exactly are we testing for here? We want to be **sure** that our variance isn't changing with our predictors. Well, what's one way of checking that? How can we do this? 

One way is to compare different values of our predicted variable and see how our errors are changing that. That's what the GQ test is trying to do! But we need a process. You can actually run all of this code with tools you've already seen so far. Let's do that. 

First, we need to pick a fraction of our sample to compare. Let's look at the first and last 3/8 of our sample and then test if they have the same variances.

What we need to do: compare the variances of first 3/8 and the last 3/8 ranked by the explaining variable of interest

This looks at the ratio and sees how far it is from 1. Think about it. If the ratio is 1, then across large and small values of your variable of interest, your errors are roughly the same.

Note that Goldfeld-Quant only allows you to look at one varibale at a time. Let's focus on income at the moment

If we wanted to do this by hand, we'd need to follow some steps. 6 steps in total. Let's walk through them:

**1.)** Order your observations by your variable of interest, in this case, let's do income. It looked the worst in our graphs, so it makes sense to check here.


```r
#in order to order a variable, all we need to do is sort it, using the arrange command.
Caschool <- arrange(Caschool, avginc)
head(Caschool$avginc)
```

```
## [1] 5.335 5.699 6.216 6.577 6.613 6.983
```

we can see `avginc` is increasing

**2.)** Split the data into two groups, in appropriate sizes. We've chosen to be three-eighths to be our sample size. We need to know what 3/8 of our dataset is.

```r
n_GQ <- as.integer(nrow(Caschool) * 3/ 8)
(nrow(Caschool)%/%8) * 3 #Option 2. this is one we've covered all the commands for. 
#It will give us something close, but slightly smaller than the last option. 
#Can you think of why? What does the %/% do?
```

```
## [1] 156
```

```r
n_GQ #this will give us the closest count that gives us the number that is 3/8 of our full sample!
```

```
## [1] 157
```

**3.)** Run seperate regressions of y (total score) on x (income) for each of the two groups of 3/8. We can do this with clever use of our `head` and `tail` commands.


```r
#on the last 3/8
lm_g1 <- lm(data = tail(Caschool, n_GQ), scrtot ~ avginc + str + enrltot + expnstu)
#using the tail command, we are running the reg on the first 3/8
lm_g2 <- lm(data = head(Caschool, n_GQ), scrtot ~ avginc + str + enrltot + expnstu) 
#using head, we are doing the above on the last 3/8
```

**4.)** Record our sum of square errors to test

We need to recover the SSEs for our test, so we need to build those. We'll use the `sum()` command, as well as our `resid()` command we just learned to do this.

```r
e_g1 <- resid(lm_g1) #the resid command gets us our models' residuals!
e_g2 <- resid(lm_g2)
sse_g1 <- sum(e_g1 ^ 2) #now, to get SSE, we need to square the residuals, and then sum them.
sse_g2 <- sum(e_g2 ^ 2)
```

**5.)** Calculate the G-Q test stastistic, and compute the p-value.
How do we do this? Stop for a second and look at Ed's notes. What kind of test is the GQ test?

```r
stat_GQ <- (sse_g2 / sse_g1)
stat_GQ
```

```
## [1] 1.39631
```

Great. So did you look up the test type? Good. We can use an F-test to produce a p-value for our calculated statistic.


**Note: I made a mistake here. What we want is n_GQ MINUS the number of variables. IE Degrees of freedom for each subgroup. In this case, it is 5. So subtract five from n_GQ. Thanks to Jason for catching the issue.**
```r
#Ok. So. In general, any distribution you can think of has a set of functions you can use on R. 
#Generally, they are prefixed with r, p, q, and d. 
#These correspond to: draws from that distribution (r), 
#likelihood of seeing a number that size (p), 
#the quantile (q) and the density (d). 
#Most often you'll see r and p used but the other ones are useful as well.
p_GQ <- pf(q = stat_GQ, df1 = n_GQ - 5, df2 = n_GQ - 5, lower.tail = F) #pf gives probability from an f-dist.
p_GQ
```

```
## [1] 0.01861109
```

So what can we say. I'd challenge you to figure out EXACTLY what our null hypothesis is here. 

**6.)** State the null hypothesis and draw conclusion, 

The *Null Hypothesis* of Goldfeld-Quant is..... H0:The variances of the residuals from regressions using the first 3/8 and the last 3/8 of the dataset ordered by average income are the same.

The *Alternative Hypothesis* of Goldfeld-Quant is..........HA: The variances of the residuals from regressions using the first 3/8 and the last 3/8 of the dataset ordered by average income are different

More formally: H0: \sigma_1^2 = \sigma_2^2, and HA: \sigma_1^2 != \sigma_2^2

Conclusion, can we reject H0?

Yes! At the 5% significance level. Let's look at another test.

## Lesson 4: Breusch-Pagan test:

Again, we need to follow some steps to run this test. How do we go about this? Another six steps!!

**1.)** Regress y (scrtot) on an intercept, x1 (avginc), x2 (str), ... xk (expnstu)

```r
lm_frst <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu)
```

**2.)** Recover residuals from our original regression

Let's use the `resid()` command again

```r
e_BP <- resid(lm_frst) #use the resid() command again
```

**3.)** Regress our squared errors (e^2) on an intercept, x1, x2, ... xk, or our explanatory variables.

```r
lm_BrPa <- lm(data = Caschool, e_BP ^ 2 ~ avginc + str + enrltot + expnstu)
```

**4.)** Record R^2

Now we need to do something a little strange. We're going to call the `r.squared` object from within the `lm` **summary** object. When have we done this before? Yep! Dataframes. So we use the same formatting.


```r
r2_BP <- summary(lm_BrPa)$r.squared #this is letting us get r squared!
```

**5.)** Compute the Bruesch-Pagan statisic (called Langrange Multiplier) and the P-value


```r
LM_BP <- nrow(Caschool) * r2_BP
pchisq(q = LM_BP, df = 4, lower.tail = F) #this function calculates a chi-squared distribution
```

```
## [1] 0.02253599
```

If you're wondering why we used 4 degrees of freedom (df in above function is equal to 4) I encourage you to look up Breusch Pagan for more details, however the general rule of thumb is that BP is a test with statistic of size n*R^2 and k degrees of freedom.

**6.)** State the null hypothesis and draw conclusion. What is H0?

**H0:** b1 = b2 = b3 = b4 = 0 where b1, b2, b3, b4 are the coefficients of regression model: 
(e.BP ^ 2 ~ b0 + b1 * avginc + b2 * str + b3 * enrltot)

**What do we conclude?**

We reject the null hypothesis with 5% significance level. We have one more test to walk through.

## Lesson 5: White test
The white test is very similar to the Breusch-Pagan test, with a few modifications:

When regressing the squared errors in the B-P test, in addition to the original regressors, we add interaction terms and squared terms. 

So the first two steps of the White test are identical to those in the Breusch-Pagan test

**White test steps!**
**1.)** Regress y (scrtot) on an intercept, x1 (avginc), x2 (str), ... xk (expnstu). Same as BP


```r
lm_first <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu)
```

**2.)** Record residuals from our reg. Also, same as BP


```r
e_WhiteTest <- resid(lm_first)
```

**3.)** 
Regress e^2 on an intercept, x1, x2, ... xk AND THE INTEREACTION AND SQUARED TERMS. This is where we go off the BP script

```r
lm_White <- lm(data = Caschool, e_BP ^ 2 ~ avginc + str + enrltot + expnstu + avginc:str + avginc:enrltot + avginc:expnstu + str:enrltot + str:expnstu + enrltot:expnstu + avginc ^ 2 + str ^ 2 + enrltot ^ 2 + expnstu^2)
```


**4.)** Record R^2. Back to the BP script, but with a new regression model

```r
r2_White <- summary(lm_White)$r.squared
```

**5.)** Compute the Bruesch-Pagan statisic (called Langrange Multiplier)

```r
LM_White <- nrow(Caschool) * r2_White 
pchisq(q = LM_White, df = 9, lower.tail = F) #this is a chi-squared function. Takes value, outputs probability.
```

```
## [1] 1.717395e-05
```

So what can we say? What is our H0? What is HA?

## Lesson 6: What next?

How do we fix our problem? We have some serious het problems. We could use a specification, but sometimes those don't work, so we have to turn to heteroskedasticity robust standard errors.

**We could build these on our own**, but R has some built in functions! Let's see how they work. This next section will require new packages called `sandwich` and `lmtest`



```r
#first let's remind ourselves what our coefs and SE look like
lm_orig <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu)
summary(lm_orig)
```

```
## 
## Call:
## lm(formula = scrtot ~ avginc + str + enrltot + expnstu, data = Caschool)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -80.657 -17.753   2.213  17.866  62.657 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  1.313e+03  2.783e+01  47.171  < 2e-16 ***
## avginc       3.864e+00  1.851e-01  20.876  < 2e-16 ***
## str         -1.395e+00  8.927e-01  -1.563   0.1188    
## enrltot     -1.611e-03  3.411e-04  -4.722  3.2e-06 ***
## expnstu     -6.052e-03  2.613e-03  -2.316   0.0211 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 25.86 on 415 degrees of freedom
## Multiple R-squared:  0.5439,	Adjusted R-squared:  0.5395 
## F-statistic: 123.7 on 4 and 415 DF,  p-value: < 2.2e-16
```

```r
#Now let's look at our HR standard errors. Notice a difference?
lm_lets_correct <- lm(data = Caschool, scrtot ~ avginc + str + enrltot + expnstu)
#This command let's us pass a NEW variance-covariance matrix that is robust to heteroskedasticity. The vcovHC command
#lets us build a new matrix using a heteroskedasticity correcting method. You have options HC0-5, 
#but Stata generally uses HC1. In general, they all do the job. 
coeftest(lm_lets_correct, vcov = vcovHC(lm_lets_correct, "HC1"))
```

```
## 
## t test of coefficients:
## 
##                Estimate  Std. Error t value  Pr(>|t|)    
## (Intercept)  1.3129e+03  2.9364e+01 44.7112 < 2.2e-16 ***
## avginc       3.8640e+00  2.3972e-01 16.1192 < 2.2e-16 ***
## str         -1.3954e+00  9.3154e-01 -1.4980   0.13490    
## enrltot     -1.6109e-03  2.7802e-04 -5.7942 1.359e-08 ***
## expnstu     -6.0517e-03  2.7936e-03 -2.1663   0.03086 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
In particular, look at how different the std errors are for avg income. Why did they change so much?

Awesome! We just did some heteroskedasticity. Good job! 

This lab was a lot but it'll be useful for you later. Enjoy your weekend, and I will see you guys next week.
