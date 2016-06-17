# Data 621 Homework 1: Moneyball
Critical Thinking Group 2  



# Overview

In this homework assignment, you will explore, analyze and model a data set containing approximately 2200 records. Each record represents a professional baseball team from the years 1871 to 2006 inclusive. Each record has the performance of the team for the given year, with all of the statistics adjusted to match the performance of a 162 game season. 

Your objective is to build a multiple linear regression model on the training data to predict the number of wins for the team. You can only use the variables given to you (or variables that you derive from the variables
provided). Below is a short description of the variables of interest in the data set:

# Part 1. Data Exploration

## Data Summary 
The data base originally sent out has 17 attributes with 2277 lines.

Describe the size and the variables in the moneyball training data set. 
Mean / Standard Deviation / Median

## Data Plots
Bar Chart or Box Plot of the data

## Correlation Plot
Is the data correlated to the target variable (or to other variables?)

## Missing and Invalid Data

We began by creating a new attribute for singles, taking the hits value and subtracting out the doubles, triples and home runs. Then we eliminated the batting hits column. We believe that separating out singles with the other unique hit values will minimize collinearity.

We began by excluding 4 data attributes for the models:

1. Hit by Pitch: In the case of hit by pitch there were very few values present (2085 missing). Based on SME knowledge from actual coaches we discovered that hit by pitches rarely impact wins. Therefore, we chose to exclude it.

2. Caught stealing: In the case of caught stealing there were many missing values as well (772). 
This attribute was found to be highly collinear with the stolen base attribute (because teams that steal a lot of bases will have more caught stealing). As a result, we chose to exclude this value and kept the positive value of stolen bases which had fewer missing components.

3. Pitching Home Runs Allowed: These data were also found to be highly collinear with Batting Homeruns (because the years of "juicing" tended to have a lot of homeruns hit and therefore pitched, while the "dead ball years" had very few hit and allowed). We chose to exclude this attribute and kept the home runs batting value

4. Index: These numbers were simply sorting keys and offer no real statistical value to the model and were therefore excluded. 

We then worked on filling in the remaining missing data. To do this we used a linear regression approach recommended by Faraway (p.???) and Fox (p.???). We decided against the mean and median as the regression approach will fill in with better variance. We filled the following fields:

1. Batting strikeouts: The adjusted R squared value for our regression was 0.7223 and the data appeared normal. We created a function that allowed us to only replace the missing values.Here we replaced 102 values.

2. Pitching strikeouts: We achieved a really good approach here as the adjusted R squared value was 0.9952. Here again we replaced the same 102 values

3. Stolen Bases: The model here was not quite as strong with an adjusted R squared value of 0.3427. Here we replaced 131 values.

4. Double Plays: The model here had an adjusted R squared value of 0.3904. Here we replaced 286 missing values. We completed this phase for the master data source by eliminating some clear outliers. The record for the most pitching strikeouts is 1450 by the 2014 Cleveland Indians. Therefore we know that everything above that point is an aberration, so we ignored all lines with a strikeout total above 1450. Similarly, the most errors by team was 639 by Philadelphia in 1883. Prorated to 162 games we ignored all rows with errors above 1046. The most ever hits by a team was 1730. To allow for any margin of error when translated to pitching we ignored all pitching hits allowed above 3000. This approach removed 104 rows and the data had a much more normal distribution for these values. For our individual models we did look at combining certain fields or creating some unique attributes from the data fields provided. We also looked at power transformations as well. These model-based transformations will be covered in the individual model section.




# Part 2. Data Preparation

## Prompt
Describe how you have transformed the data by changing the original variables or creating new variables. If you did transform the data or create new variables, discuss why you did this. Here are some possible transformations:

Fix missing values (maybe with a Mean or Median value)
Create flags to suggest if a variable was missing
Transform data by putting it into buckets
Mathematical transforms such as log or square root (or use Box-Cox)
Combine variables (such as ratios or adding or multiplying) to create new variables

## Notes
are we introducing outliers through imputation?
4 variables have significant NA variables and need to be addressed 
[chart of NA percentage]
concerns about linearity
linear model introduced the fewest problems compared to other options
linear model fits the data to the existing distribution of the variable
is it valid to replace the outliers or should we ignore them? 
good leverage points - fictitious data or outside norm but still fit on line
bad leverage points - pulling data
if beyond 2 SDs using cook's distance test, then remove
how many instances are we replacing - 3000
delete about 100 lines out of 2200
3 variables with very skewed data


## Addressing Outliers via Historical Data

If you run the two strikeouts variables against each other you really see these leveraged "bad" values clustered near zero and between 1500 & 20000 pitched strikeouts (clearly made-up numbers).  Many of these outliers are invalid in more that one column. 

The record for pitched strikeouts is 1450 by the 2014 Cleveland Indians.

Proving that players are striking out with less frequency in 1921, the Cincinnati Reds set a major league mark for #the fewest team strikeouts with 308, while Phillie pitchers set an all-time low by striking out 333 opponents through #the year.

min & max team strikeouts all-time
http://www.baseball-almanac.com/recbooks/rb_strike2.shtml
329 1526

http://www.thisgreatgame.com/1921-baseball-history.html
http://www.foxsports.com/florida/story/tampa-bay-rays-set-franchise-strikeout-record-092114

## Addressing Outliers via Cook's Distance

A defensible approach, simplistic but valid is just deleting "bad" leverage points or invalid data and refitting the model w/o them. (Sheather MAR pg. 57). If you look at the Residuals vs Leverage plot (Sheather MAR pg. 70) is a good example, these points are outside of the +/- 2 SD --the so called Cook's distance and can be removed.  Its also in the flowchart on pg. 103.  The bonds example in the chapter 3 exercises of MAR have the Cook's Distance test code for the "Bond" example where he throws out 2 of 35 observations due to "bad" leverage.

Here's an extract . . .

Figure 3.13 on page 68

```r
#cd1 <- cooks.distance(m1)
#plot(CouponRate,cd1,xlab="Coupon Rate (%)", ylab="Cook's Distance")
#abline(h=4/(35-2),lty=2)
#identify(CouponRate,cd1,Case)
```

Once removed, these influential points may improve the variability problems with some of our data, or at least make it easier run a transform.
A pattern of non-constant variability calls into question whether a variable belongs in the linear model.


## Combining Variables and Creating New Variables

converting the hits (singles, 2B, 3B, HR) to TOTAL_BASES

TOTAL BASES only includes actual hits and not SB's or HBP's or BB's.  

The following variable was added:


```r
# mb_red8$TOTAL_BASES <- mb_red8$TEAM_BATTING_1B + (2 * mb_red$TEAM_BATTING_2B) + 
# (3 * mb_red$TEAM_BATTING_3B) + (4 * mb_red$TEAM_BATTING_HR)
```


which yields a nice, normal distribution thereby eliminating the skew problems that can be found in some of the underlying variables. That statistic alone accounts for nearly 18% of the variability in TARGET_WINS.


## Evaluating Constant Variability for Imputation

The linear models used for imputing the NA's all appear to fail to meet the requirement of constant variability in the residuals.  This can be seen in the "Residuals vs. Fitted" plots for each one - each one has a much narrower range of residual values for in the lower ranges of the fitted variable than they do in the upper ranges. 


## Evaluating Linearity for Imputation

Also, at least three of the "Residuals vs. Fitted" plots show clear patterns in the residuals which is a telltale sign of non-linearity in the underlying data.

Both of these issues indicate that the linear models used for imputing the NA's aren't valid. 

Do the overall predictors pass the constant variability test.  Remember we put in only a 100 or data points in 4 fields - so maybe 400 out of 20,000 or so.   For the sake of the models that really matter - target win prediction the issues you raise become very relevant. 

## Evaluating Normality for Multi Imputation

Multi-imputation doesn't work without normally distributed data.

## Addressing NA Values with Imputation

Deleting missing cases is the simplest strategy for dealing with missing data.  It avoids the complexity and possible biases introduced by more sophisticated methods. The drawback is throwing away infomration that might allow more precise inference. If relatively few cases contain missing values deleting still leaves a
large dataset or to communicate a simple data analysis method, the deltion strategy is satisfactory.

Standard errors are larger after deleting cases because of fewer records to fit the model. arger standard errors results in less precise estimates.  (Faraway, LMR 2015, p.200)

Single imputation  . .  causes bias, while deletion causes a loss of information. Multiple imputation is a way to reduce the bias caused by single imputation.  The problem with single imputation is the value tends to be less variable than the value we would have seen because it does not include the error variation normally seen in observed data.  The idea behind multiple imputation is to re-include that error variation.
(Faraway, LMR 2015, p.202)

Multiple imputation can be done using the Amelia package.  Per Faraway, the assumption is the data is multivariate normal, so heavily skewed variables should be log-transformed first.

we want to fill in NA's where possible since we are concerned about the possible effects of discarding so many data records.

Using multiple imputation allows us to fill in some of the NA's without introducing undue bias into the model the way single imputation would. 

The R^2 values are MUCH better WITHOUT the missing variables filled in, in most instances a difference of at least 0.10, i.e., if a model yielded an R^2 of 0.40 without the NA's filled in that same model yields an R^2 of less than 0.30 with them filled in.

Could it be we're introducing bias via the NA fill in and/or setting the outliers to the median for those 3 variables? For example the PITCHING_H variable has a total of 86 elements (3.8% of the total) with a value > 3000, and we're setting them all to a value of 1518, etc..

Also, after running diagnostics on several different models, SB's and TEAM_BATTING_BB's, and FIELDING_E consistently showed as having non-constant variability relative to the residuals. All of those variables have fairly significant skew in their own distributions so we might want to consider transforming them or seeing if we can normalize them by getting rid of some of their outliers per Scott's suggestion.


Stolen Bases - consider Amelia script for multiple imputing


# Part 3. Build Models

## Model 1. SMK Model 1

**Writeup**

Ran a regression of home runs against wins using Box-cox.
Transformed home runs to become symmetric and improve standard errors.
Used this transformed primiteive variable in the multi regression model.


<a href="https://github.com/spsstudent15/2016-02-621-W1/blob/master/HW_1_SMKmodel.pdf">SMK Model 1 PDF</a>

<a href="https://raw.githubusercontent.com/spsstudent15/2016-02-621-W1/master/HW_1_SMKmodel.Rmd">SMK Model 1 RMD</a>

## Model 2. 

**Writeup**

<a href="https://github.com/spsstudent15/2016-02-621-W1/blob/master/">Model 2 PDF</a>

<a href="https://raw.githubusercontent.com/spsstudent15/2016-02-621-W1/master/">Model 2 RMD</a>

## Model 3. 

**Writeup**

<a href="https://github.com/spsstudent15/2016-02-621-W1/blob/master/">Model 3 PDF</a>

<a href="https://raw.githubusercontent.com/spsstudent15/2016-02-621-W1/master/">Model 3 RMD</a>


# Part 4. Select Models

To test our models, we needed an evaluation data set with NA's imputed. We checked the imputation output to ensure it conformed with the actual distribution of each of the impacted variables in the EVAL set. 

The EVAL data set with the NA's filled can be found here:

<a href="https://raw.githubusercontent.com/spsstudent15/2016-02-621-W1/master/621-HW1-Clean-EvalData-.csv">Clean Eval Data</a>


