# 421-Lab-Notes

Here is where you can see the lab notes for Econometrics, EC421 for Spring 2019. I'll periodically update these after class on Thursdays.

Some Useful Resources:

[Ed's Winter Course Github Page](https://github.com/edrubin/EC421W19)

[Getting Started With R](https://www.rstudio.com/online-learning/#r-programming) 
- A nice collection of resources for those starting out in R.

[Tidyr cheatsheet](https://github.com/rstudio/cheatsheets/blob/master/data-import.pdf) 
- A few of the most useful pages you can find.

[ggplot2 cheatsheet](https://www.rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf)
- Another few of the most useful pages you can find, ggplot this time

**Lab Notes**

**Lab 1**: [Learning R: Introduction link 1](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab1/Lab-1.md) Alternatively, [link 2](http://rpubs.com/Clennon/Lab1)
- Objects, Functions, Mathematical Operators, Dataframes, Regression, Basic Plots

**Lab 2**: [Learning R: Enter the Tidyverse](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab2/Lab_2.md) Alternatively, [link 2](http://rpubs.com/Clennon/Lab2) 
- Loading your own CSVs into R, dplyr commands (mutate, summarise, filter etc.), qplot

**Lab 3**: [Homework Review Session](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab3/Lab3Notes.md) Alternatively, [link 2](http://rpubs.com/Clennon/Lab3)
- Loading in the problem set CSV, two sample means test, interaction terms, PS-1 help

*Note: I made an error in these notes regarding Goldfeld Quandt degrees of frredom. Look in the notes for a correction*  

**Lab 4**: [Heteroskedasticity](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab-4/Lab4Notes.md), Alternatively, [link 2](http://rpubs.com/Clennon/Lab4)
- Retrieving errors from your models, Testing for heteroscedasticity in R, graphing errors, using heteroskedasticity robust standard errors.

**Lab 5**: [Homework Help: Problem Set 2](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab5/Lab5Notes.md), Alternatively, [link 2](http://rpubs.com/Clennon/Lab5)
- Reviewing tests for heteroskedasticity, retrieving errors, and using heteroskedasticity robust errors in an LM model (FELM)

**Lab 6**: [Functions and introduction to time series data](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab6/Lab6Notes.md), Alternatively, [link 2](http://rpubs.com/Clennon/Lab6)
 - Generating simulated data, functions, sapply, for loops, a tiny bit of time series (including bonus prediction method at end)

**Lab 7**: [Practice for Time Series Simulation, with Suggested Activities](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab7/Lab7Notes.md), Altenatively, [link 2](http://rpubs.com/Clennon/Lab7)
 - Simulate an ADL(1,1) model, piece by piece, and generate your own data simulations for an AR(2) and MA(2) process on your own.

**Lab 8_5**: [Homework Help for Problem set 3](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab8_HW/Week8HomeworkNotes.md), alternatively, [link 2](http://rpubs.com/Clennon/Lab8_5)
 - Testing different hypotheses of data structure against one another, graphing residuals and lags of residuals (including explanation of lag() function.)
 
 **Lab 8**: [What We actually did in Lab](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab8Notes/Lab8Notes.md), alternatively, [link 2](http://rpubs.com/Clennon/Lab8)
 - Work with FRED data to examine autocorrelation, test for different data structures, and understanding the `lag`/`waldtest` commands.
 
 **Mistake was caught in lab 9:** When estimating 2SLS in IV manually, you need to include all variables when estimating your first stage. 
 
 **Lab 9**: [Instrumental Variables (IV), Two-stage Least-Squares (2SLS) and other acronyms](https://github.com/CMLennon/421-Lab-Notes/blob/master/Lab9/Week9.md), alternatively, [link 2](http://rpubs.com/Clennon/Lab9)
 - A soft introduction to Instrumental Variable estimation (IV). Using the `AER` package, try out 2SLS on education/wage data.
 
 - Specifically, if you want a very in depth look at instrumentation outside of your course materials, I would use Glen Waddell's notes
 [here](https://pages.uoregon.edu/waddell/metrics/instrumental-variables.html). You won't need to know this material, but it's good to know if you want a job in data analysis.
 
 **Lab 9_5**: [Homework Help: Problem Set 4](http://rpubs.com/Clennon/501614)
  - Review of material for PS4. Simulate a stationary process.
 
 **Lab 10**: [Final review material, with practice prepared in advance.](http://rpubs.com/Clennon/FinalReview)
  - Good luck on your final exam!

A note: I owe a great debt of gratitude to John Morehouse, Alex Li, and of course Ed Rubin for laying the foundations of the lab work for me to build notes off of.

[Final Review Questions (No Answers Provided)](https://raw.githack.com/edrubin/EC421W19/master/Final/Review/final_review.pdf)

[Last Year's Final (No Answers Provided)](https://raw.githack.com/edrubin/EC421W19/master/Final/Exam/final.pdf)

An aside:

If you're adventurous, Grant McDermott's data science for economists is an amazing resource. It's designed for PhD students, but not those with any experience with computers. If metrics is your calling, you might enjoy looking at these.

[Grant's Notes](https://github.com/uo-ec607/lectures)
