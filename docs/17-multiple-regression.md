# Multiple regression

There is currently much debate (and hype) surrounding smartphones and their effects on well-being, especially with regard to children and teenagers.  We'll be looking at data from this recent study of English adolescents:

> Przybylski, A. & Weinstein, N. (2017). A Large-Scale Test of the Goldilocks Hypothesis. *Psychological Science*, *28*, 204--215.

This was a large-scale study that found support for the "Goldilocks" hypothesis among adolescents: that there is a "just right" amount of screen time, such that any amount more or less than this amount is associated with lower well-being.  This was a huge survey study: the data contain responses from over 120,000 participants! 

Fortunately, the authors made the data from this study openly available, which allows us to dig deeper into their results. In this exercise, we will look at whether the relationship between screen time and well-being is modulated by participants' (self-reported) gender.

The dependent measure used in the study was the [Warwick-Edinburgh Mental Well-Being Scale (WEMWBS)](https://warwick.ac.uk/fac/med/research/platform/wemwbs/). This is a 14-item scale with 5 response categories, summed together to form a single score ranging from 14-70.

At [Przybylski & Weinstein's page for this study on the Open Science Framework](https://osf.io/82ybd/), you can find the [participant survey](https://osf.io/82ybd/) which asks a large number of additional questions (see page 14 for the WEMWBS questions and pages 4-5 for the questions about screen time). Within the same page you can also find the [raw data](https://osf.io/82ybd/); however, for the purpose of this exercise, you will be using local pre-processed copies of the data which we will provide.

Przybylski and Weinstein looked at multiple measures of screen time, but we will be focusing on smartphone use.  They found that decrements in well-being started to appear when respondents reported more than one hour of weekly smartphone use.  Our question: Does the negative association between hours of use and well-being (beyond the one-hour point) differ for boys and girls?

Note that in this analysis, we have:

- a continuous$^*$ DV, well-being;

- a continuous$^*$ predictor, screen time;

- a categorical predictor, gender.

$^*$these variables are only quasi-continuous, inasmuch as only discrete values are possible. However, there are a sufficient number of discrete categories that we can treat them as effectively continuous.

We want to estimate two slopes relating screen time to well-being, one for girls and one for boys, and then statistically compare these slopes. So this problem seems simultaneously like a situation where you would run a regression (to estimate the slopes) but also one where you would need a t-test (to compare two groups).

But the expressive power of regression allows us to do this all within a single model. As the [Bishop blog showed](http://deevybee.blogspot.com/2017/11/anova-t-tests-and-regression-different.html), *an independent groups t-test is just a special case of ordinary regression with a single categorical predictor; ANOVA is just a special case of regression where all predictors are categorical.*  So although we can express any ANOVA design using regression, the converse is not true: we cannot express every regression design in ANOVA. Regression allows us to have any combination of continuous and categorical predictors in the model. The only inconvenience with running ANOVA models as regression models is that you have to take care in how you numerically code the categorical predictors.

## Activity 1: Set-up {#mulregression-a1}

* Open R Studio and set the working directory to your chapter folder. Ensure the environment is clear.    
* Open a new R Markdown document and save it in your working directory. Call the file "Multiple Regression".    
* Download <a href="wellbeing.csv" download>wellbeing.csv</a>, <a href="participant_info.csv" download>participant_info.csv</a> and <a href="screen_time.csv" download>screen_time.csv</a> and save them in your Chapter folder. Make sure that you do not change the file names at all.    
* If you're on the server, avoid a number of issues by restarting the session - click `Session` - `Restart R` 
* Delete the default R Markdown welcome text and insert a new code chunk that loads `pwr`, `car`, `broom`, and `tidyverse` using the `library()` function.
* Load the CSV datasets into variables called `pinfo`, `wellbeing` and `screen` using `read_csv()`.



## Activity 2: Look at the data {#mulregression-a2}

Take a look at the resulting tibbles `pinfo`, `wellbeing`, and `screen`.  The `wellbeing` tibble has information from the WEMWBS questionnaire; `screen` has information about screen time use on weekends (variables ending with `we`) and weekdays (variables ending with `wk`) for four types of activities: using a computer (variables starting with `Comph`; Q10 on the survey), playing video games (variables starting with `Comp`; Q9 on the survey), using a smartphone (variables starting with `Smart`; Q11 on the survey) and watching TV (variables starting with `Watch`; Q8 on the survey).  If you want more information about these variables, look at the items 8-11 on pages 4-5 of the the [PDF version of the survey on the OSF website](https://osf.io/82ybd/).

* The variable corresponding to *gender* is located in the table named <select class='solveme' data-answer='["pinfo"]'> <option></option> <option>pinfo</option> <option>wellbeing</option> <option>screen</option></select> and this variable is called <input class='solveme nospaces' size='6' data-answer='["male"]'/>.

* The WEMWBS data is in <select class='solveme' data-answer='["wide"]'> <option></option> <option>long</option> <option>wide</option></select> format, and contains observations from <input class='solveme nospaces' size='10' data-answer='["102580","102,580"]'/> participants on <input class='solveme nospaces' size='2' data-answer='["15"]'/> items.

* Individual participants in this dataset are identified by the variable named <input class='solveme nospaces' size='9' data-answer='["Serial"]'/> [be sure to type the name *exactly*, including capitalization].  This variable will allow us to link information across the three tables.

* Run `summary()` on the three data-sets. Are there any missing data points? <select class='solveme' data-answer='["No"]'> <option></option> <option>Yes</option> <option>No</option></select>


## Activity 3: Compute the well-being score for each respondent {#mulregression-a3}

The WEMWBS well-being score is simply the *sum* of all the items. 

* Write the code to create a new table called `wemwbs`, with two variables: `Serial` (the participant ID), and `tot_wellbeing`, the total WEMWBS score.


<div class='solution'><button>Hint</button>

- "pivot" the table from wide to long

</div>



<div class='solution'><button>Another Hint</button>

- `group_by()`; `summarise(tot_wellbeing = ...)`

</div>




**Sanity check:** Verify for yourself that the scores all fall in the 14-70 range.  Przybylski and Weinstein reported a mean of 47.52 with a standard deviation of 9.55. Can you reproduce these values?


<div class='solution'><button>Hint</button>

- `summarise()`, `min()`, `max()`

</div>

<br>

* Now visualise the distribution of `tot_wellbeing` in a histogram using ggplot2.  


<div class='solution'><button>Hint</button>

- `geom_histogram()`

</div>



<div class='solution'><button>Solution</button>


```r
ggplot(wemwbs, aes(tot_wellbeing)) + geom_histogram() 
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<div class="figure" style="text-align: center">
<img src="17-multiple-regression_files/figure-html/wemwbs_histogram-1.png" alt="**CAPTION THIS FIGURE!!**" width="100%" />
<p class="caption">(\#fig:wemwbs_histogram)**CAPTION THIS FIGURE!!**</p>
</div>


</div>


The distribution of well-being scores is <select class='solveme' data-answer='["negatively skewed"]'> <option></option> <option>symmetric</option> <option>negatively skewed</option> <option>positively skewed</option></select>.

## Activity 4: Visualise the relationship {#mulregression-a4}

Let's take a quick look at the relationship between screen time (for the four different technologies) and measures of well-being.  Here is code to do this. 

* Run the below code and try and explain in words what each line of code is doing (remember, pronounce `%>%` as "and then"). You may find it easier to look at each of the tables that are produced.


```r
screen_long <- screen %>%
  pivot_longer(names_to = "var", values_to = "hours", -Serial) %>%
  separate(var, c("variable", "day"), "_")

screen2 <- screen_long %>%
  mutate(variable = dplyr::recode(variable,
               "Watch" = "Watching TV",
               "Comp" = "Playing Video Games",
               "Comph" = "Using Computers",
               "Smart" = "Using Smartphone"),
     day = dplyr::recode(day,
              "wk" = "Weekday",
              "we" = "Weekend"))

dat_means <- inner_join(wemwbs, screen2, "Serial") %>%
  group_by(variable, day, hours) %>%
  summarise(mean_wellbeing = mean(tot_wellbeing))

ggplot(dat_means, aes(hours, mean_wellbeing, linetype = day)) +
  geom_line() +
  geom_point() +
  facet_wrap(~variable, nrow = 2)
```

<div class="figure" style="text-align: center">
<img src="17-multiple-regression_files/figure-html/combined-1.png" alt="Relationship between wellbeing and screentime usage by technology and weekday" width="100%" />
<p class="caption">(\#fig:combined)Relationship between wellbeing and screentime usage by technology and weekday</p>
</div>

The graph makes it evident that smartphone use of more than 1 hour per day is associated with increasingly negative well-being.  Note that we have combined the tables using an `inner_join()`, such that we only include data for which we have observations across the `wemwbs` and `screen2` tables.

In the next step, we are going to focus in on the smartphone/well-being relationship.

## Activity 5: Smartphone and well-being for boys and girls {#mulregression-a5}

For this analysis, we are going to collapse weekday and weekend use for smartphones.

* Create a new table, `smarttot`, that has the that has mean number of hours per day of smartphone use for each participant, averaged over weekends/weekdays. 
* You will need to filter the dataset to only include smartphone use and not other technologies. 
* You will also need to group the results by the participant ID (i.e., `serial`). 
* The final data-set should have two variables: `Serial` (the participant) and `tothours`.
* You will need to use the data-set `screen2` to do this.


<div class='solution'><button>Hint</button>

- `filter()` then `group_by()` then `summarise()`

</div>


* Next, create a new tibble called `smart_wb` that only includes (filters) participants from `smarttot` who used a smartphone for more than one hour per day each week, and then combine (join) this table with the information in `wemwbs` and `pinfo`.**


<div class='solution'><button>Hint</button>

- `filter()` then `inner_join()` then another `inner_join()`

</div>




## Activity 6: Mean-centering variables {#mulregression-a6}

As discussed in the lecture, When you have continuous variables in a regression, it is often sensible to transform them by *mean centering*.  You mean center a predictor `X` simply by subtracting the mean (`X_centered = X - mean(X)`). This has two useful consequences:

- the model intercept reflects the prediction for $Y$ at the mean value of the predictor variable, rather than at the zero value of the unscaled variable;

- if there are interactions in the model, any lower-order effects can be given the same interpretation as they receive in ANOVA (main effects, rather than simple effects).

For categorical predictors with two levels, these become coded as -.5 and .5 (because the mean of these two values is 0).

* Use `mutate` to add two new variables to `smart_wb`: `tothours_c`, calculated as a mean-centered version of the `tothours` predictor; and `male_c`, recoded as -.5 for female and .5 for male.
* To create `male_c` you will need to use `if_else(male == 1, .5, -.5)` You can read this code as "if the variable `male` equals 1, recode it as .5, if not, recode it as -.5".
* Finally, recode `male` and `male_c` as factors, so that R knows not to treat them as a real numbers.




## Activity 7: Visualise the relationship {#mulregression-a7}

* Reverse-engineer the below plot. Calculate mean well-being scores for each combination of `male` and `tothours`, and then create a scatterplot plot that includes separate regression lines for each gender.
* You may find it useful to refer to the Visualisation chapter.


<div class='solution'><button>Hint</button>

- `group_by()` both variables then `summarise()`
- `colour = variable_you_want_different_colours_for`

</div>


<div class="figure" style="text-align: center">
<img src="17-multiple-regression_files/figure-html/plots-1.png" alt="Relationship between mean wellbeing and smartphone use by gender" width="100%" />
<p class="caption">(\#fig:plots)Relationship between mean wellbeing and smartphone use by gender</p>
</div>

Write an interpretation of the above plot in plain English.


<div class='solution'><button>Possible solution</button>

Girls show lower overall well-being compared to boys.  In addition, the slope for girls appears more negative than that for boys; the one for boys appears relatively flat.  This suggests that the negative association between well-being and smartphone use is stronger for girls.

</div>


## Activity 8: Running the regression {#mulregression-a8}

Now we're going to see if there is statistical support for our above interpretation of the graph.

For the data in `smart_wb`, use the `lm()` function to calculate the multiple regression model:

$Y_i = \beta_0 + \beta_1 X_{1i}  + \beta_2 X_{2i}  + \beta_3 X_{3i} + e_i$

where

- $Y_i$ is the well-being score for participant $i$;
- $X_{1i}$ is the mean-centered smartphone use variable for participant $i$;
- $X_{2i}$ is gender (-.5 = female, .5 = male);
- $X_{3i}$ is the interaction between smartphone use and gender ($= X_{1i} \times X_{2i}$)

Then use `summary()` to view the results and store this in an object called `mod_summary()`.


<div class='solution'><button>Hint</button>

- R formulas look like this: `y ~ a + b + a:b` where `a:b` means interaction

</div>




* The interaction between smartphone use and gender is shown by the variable <select class='solveme' data-answer='["thours_c:male_c"]'> <option></option> <option>thours_c</option> <option>male_c</option> <option>thours_c:male_c</option></select>, and this interaction was <select class='solveme' data-answer='["significant"]'> <option></option> <option>significant</option> <option>nonsignificant</option></select> at the $\alpha = .05$ level.

* To 2 decimal places, what proportion of the variance in well-being scores does the overall model explain? <input class='solveme nospaces' size='4' data-answer='["9.38"]'/>

* The p-value for the overall model fit is `< 2.2e-16`. Is this significant? <select class='solveme' data-answer='["Yes"]'> <option></option> <option>Yes</option> <option>No</option></select>

* What is the most reasonable interpretation of these results? <select class='solveme' data-answer='["smartphone use was more negatively associated with wellbeing for girls than for boys"]'> <option></option> <option>smartphone use harms girls more than boys</option> <option>smartphone use harms boys more than girls</option> <option>there is no evidence for gender differences in the relationship between smartphone use and well-being</option> <option>smartphone use was more negatively associated with wellbeing for girls than for boys</option></select>

## Activity 9: Assumption checking {#mulregression-a9}

Now it's time to test those pesky assumptions. The assumptions for multiple regression are the same as simple regression but there is one additional assumption, that of multicollinearity, the idea that predictor variables should not be too highly correlated.

1. The outcome/DV is a interval/ratio level data 
2. The predictor variable is interval/ratio or categorical (with two levels)
3. All values of the outcome variable are independent (i.e., each score should come from a different participant)
4. The predictors have non-zero variance
5. The relationship between outcome and predictor is linear
6. The residuals should be normally distributed
7. There should be homoscedasticity (homogeneity of variance, but for the residuals)
8. Multicollinearity: predictor variables should not be too highly correlated

From the work we've done so far we know that assumptions 1 - 4 are met. Unlike when we did simple regression we can't use `crPlots()` to test for linearity when there is an interaction, but we know from looking at the grouped scatterplot that this assumption has been met.

Normally we would test for normality with a qqplot and a Shapiro-Wilk test. However, because this dataset is so large, the Shapiro-Wilk is not appropriate (if you try to run the test it will produce a warning telling you that the sample size must be between 3 and 5000). This is because as we have mentioned before, with extremely large sample sizes the Shapiro-Wilk test will find that any deviation from normality is significant. Therefore we should judge normality based upon the qqplots.

* Create a qqplot of the model residuals. Can we assume normality? <select class='solveme' data-answer='["Yes"]'> <option></option> <option>Yes</option> <option>No</option></select>

We also have this problem for testing homoscedasticity with `ncvTest()` so we need to rely on plots again. To check for homoscedasticity we can use `plot()` from Base R that will produce a bunch of helpful plots ([more information here](https://www.r-bloggers.com/how-to-detect-heteroscedasticity-and-rectify-it/)). The residuals vs leverage plot shows a flat red line so, whilst it isn't perfect, we can assume that with such a large sample size regression is still an appropriate analysis. 


```r
par(mfrow=c(2,2)) # 4 charts in 1 panel
plot(mod)
```

<div class="figure" style="text-align: center">
<img src="17-multiple-regression_files/figure-html/plots2-1.png" alt="Regression assumption plots" width="100%" />
<p class="caption">(\#fig:plots2)Regression assumption plots</p>
</div>

Finally, to check for multicollinearity we can use `vif()` to calculate the variance inflation factor. Essentially, this function estimates how much the variance of a coefficient is “inflated” because of linear dependence with other predictors, i.e., that a predictor isn't actually adding any unique variance to the model, it's just really strongly related to other predictors. [You can read more about this here](https://statisticalhorizons.com/multicollinearity). Thankfully, VIF is not affected by large samples like the other tests.

There are various rules of thumb, but most converge on a VIF of above 2 - 2.5 for any one predictor being problematic.

* Run the below code. Do any of the predictors show evidence of multicollinearity? <select class='solveme' data-answer='["There is no evidence of multicollinearity"]'> <option></option> <option>thours_c</option> <option>male_c</option> <option>thours_c:male_c</option> <option>There is no evidence of multicollinearity</option></select>


```r
vif(mod)
```

## Activity 10: Power and effect size {#mulregression-a10}

Finally, we'll calculate power and effect size as usual.

* Using the code from Power and Effect Size calculate the minimum effect size we could reliably observe given our sample size and design but for 99% power. Report this to 2 decimal places <input class='solveme nospaces' size='0.00' data-answer='[".00"]'/>







* What is the observed effect size for the study to 2 decimal places? <input class='solveme nospaces' size='0.10' data-answer='[".10"]'/>  
* Is the study sufficiently powered? <select class='solveme' data-answer='["Yes"]'> <option></option> <option>Yes</option> <option>No</option></select>

## Activity 11: Write-up {#mulregression-a11}

Now, copy and paste the below code into **white-space** and then knit the document. Note that the p-values are entered manually because of the APA `p < .001` formatting.


```r
All continuous predictors were mean-centered and deviation coding was used for categorical predictors. The results of the regression indicated that the model significantly predicted course engagement (F(`r mod_summary$fstatistic[2]`, `r mod_summary$fstatistic[3] %>% round(2)`) = `r mod_summary$fstatistic[1] %>% round(2)`, p < .001, Adjusted R2 = `r mod_summary$adj.r.squared %>% round(2)`, f^2^ = .63), accounting for `r (mod_summary$adj.r.squared %>% round(2))*100`% of the variance. Total screen time was a significant negative predictor of wellbeing scores (β = `r mod$coefficients[2] %>% round(2)`, p < .001, as was gender (β = `r mod$coefficients[3] %>% round(2)`, p < .001, with girls having lower wellbeing scores than boys. Importantly, there was a significant interaction between screentime and gender (β = `r mod$coefficients[4] %>% round(2)`, p < .001), smartphone use was more negatively associated with wellbeing for girls than for boys. 
```

> All continuous predictors were mean-centered and deviation coding was used for categorical predictors. The results of the regression indicated that the model significantly predicted course engagement (F(3, 7.1029\times 10^{4}) = 2450.89, p < .001, Adjusted R2 = 0.09, f2 = .63), accounting for 9% of the variance. Total screen time was a significant negative predictor of well-being scores (β = -0.77, p < .001, as was gender (β = 5.14, p < .001, with girls having lower well-being scores than boys. Importantly, there was a significant interaction between screen time and gender (β = 0.45, p < .001), smartphone use was more negatively associated with well-being for girls than for boys.

## Finished! {#mulregression-fin}

And you're done! Not just with this week but with the R component of RM2! Well, aside from the final portfolio worksheet. The progress that you have made is truly astonishing. Even if you struggled with R and haven't quite understood every single line of code we've shown, what you're capable of with data wrangling and visualisation alone makes you some of the most highly competitive psychology graduates in the world. 

Regardless of whether you continue with quantitative methods and using R, remember the more important critical skills that you have learned as part of this process. The next time you see a dataset or you see data being talked about in the news, think about all work that was put into getting the data into the final format. More importantly, think about all the decisions that the researcher needed to make along the way and how that might have affected the outcome. 

![](https://media.giphy.com/media/ujGfBmVppmgEg/giphy.gif) 


## Activity solutions {#mulregression-sols}

### Activity 3 {#mulregression-a3sol}

<div class="solution"><button>Activity 3</button>

```r
wemwbs <- wellbeing %>%
  pivot_longer(names_to = "var", values_to = "score", -Serial) %>%
  group_by(Serial) %>%
  summarise(tot_wellbeing = sum(score))

# sanity check values

wemwbs %>% summarise(mean = mean(tot_wellbeing),
                     sd = sd(tot_wellbeing),
                     min = min(tot_wellbeing), 
                     max = max(tot_wellbeing))
```
</div>

### Activity 5 {#mulregression-a5sol}

<div class="solution"><button>Activity 5</button>

```r
smarttot <- screen2 %>%
  filter(variable == "Using Smartphone") %>%
  group_by(Serial) %>%
  summarise(tothours = mean(hours))

smart_wb <- smarttot %>%
  filter(tothours > 1) %>%
  inner_join(wemwbs, "Serial") %>%
  inner_join(pinfo, "Serial") 
```
</div>

### Activity 6 {#mulregression-a6sol}

<div class="solution"><button>Activity 6</button>

```r
smart_wb <- smarttot %>%
  filter(tothours > 1) %>%
  inner_join(wemwbs, "Serial") %>%
  inner_join(pinfo, "Serial") %>%
  mutate(thours_c = tothours - mean(tothours),
         male_c = ifelse(male == 1, .5, -.5),
         male_c = as.factor(male_c),
         male = as.factor(male))
```
</div>

### Activity 7 {#mulregression-a7sol}

<div class="solution"><button>Activity 7</button>

```r
smart_wb_gen <- smart_wb %>%
  group_by(tothours, male) %>%
  summarise(mean_wellbeing = mean(tot_wellbeing))

ggplot(smart_wb_gen, aes(tothours, mean_wellbeing, color = male)) +
  geom_point() +
  geom_smooth(method = "lm") +
  scale_color_discrete(name = "Gender", labels = c("Female", "Male"))+
  scale_x_continuous(name = "Total hours smartphone use") +
  scale_y_continuous(name = "Mean well-being score")
```
</div>

### Activity 8 {#mulregression-a8sol}

<div class="solution"><button>Activity 8</button>

```r
mod <- lm(tot_wellbeing ~ thours_c * male_c, smart_wb)
# alternatively: 
# mod <- lm(tot_wellbeing ~ thours_c + male_c + thours_c:male_c, smart_wb)

mod_summary <- summary(mod)
```
</div>

### Activity 9 {#mulregression-a9sol}

<div class="solution"><button>Activity 9</button>

```r
qqPlot(mod$residuals)
```
</div>

### Activity 10 {#mulregression-a10sol}

<div class="solution"><button>Activity 9</button>

```r
pwr.f2.test(u = 3, v = 71029, f2 = NULL, sig.level = .05, power = .99)
f2 <- mod_summary$adj.r.squared/(1 - mod_summary$adj.r.squared)
```
</div>
