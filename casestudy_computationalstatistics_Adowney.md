


Generational changes in support for gun laws: a case study in computational statistics
===========================================================================
by Allen Downey, October 17 2018

Audience level:
Intermediate

### Description 
In the United States, support for gun control has been declining among all age groups since 1990; among young adults, support is substantially lower than among previous generations.  Using data from the General Social Survey (GSS), I perform age-period-cohort analysis to measure and predict these generational effects.

Abstract: 
In this talk, I demonstrate a computational approach to statistics that replaces mathematical analysis with random simulation.  Using Python and libraries like NumPy and StatsModels, we can define basic operations — like resampling, filling missing values, modeling, and prediction — and assemble them into a data analysis pipeline.

This approach is more flexible than math-based statistics, and fits better into an incremental software development process.  The result is a set of versatile, reusable components and novel analytic methods.


#### Effect possibilities to examine

In general, it is hard to dis-entangle these effects. 


##### period effect
How to diagnose? 
Divide people into groups and plot versus time. Should be able observe trends reflecting by all segments/groups. 


##### cohort effect
Depends on when you are born, measures the effect of your environment.
How to diagnose? 
Possibility Y vs year of birth mixes both cohort and age affects. Then check for an age affect. 


##### age effect
Factors that change in predictable ways as people age. 
How to diagnose? 
1. group by age, then plot across ages.
2. check if there is a consistent effect or pattern. 


#### Conclusions for this gun control case study
- likely a period affect and likely a cohort effect. 

- Regression model here can pick any two from period effect, age and cohort.
    - age and cohort are dependent on one another, pick one
    - throw independent variables into a logistic regression:
          - the coefs are log-odds
          
- Interpretation of logistic model
    - base case audience has a 64% of favoring gun control
    - the indepenent variables are variables that deviate from the base case features or offset from the base-case
    - model takeaways: someone surveyed today is 14-15 points less likely to support gun control 
    
